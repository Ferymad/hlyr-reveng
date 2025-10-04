# Claude Code Configuration Guide

**Last Updated:** 2025-10-04
**Status:** Complete
**Research Source:** Claude Code official documentation via Ref MCP + WebSearch

---

## Table of Contents

1. [Configuration File Overview](#configuration-file-overview)
2. [Configuration Precedence](#configuration-precedence)
3. [MCP Server Configuration](#mcp-server-configuration)
4. [Tool Permissions System](#tool-permissions-system)
5. [Subagent Configuration](#subagent-configuration)
6. [Best Practices for Teams](#best-practices-for-teams)
7. [Security Considerations](#security-considerations)
8. [Example Configurations](#example-configurations)

---

## Configuration File Overview

### Settings Files: settings.json vs settings.local.json

Claude Code uses a hierarchical settings system with multiple configuration files:

| File | Location | Purpose | Git Status | Scope |
|------|----------|---------|------------|-------|
| `~/.claude/settings.json` | User home | Global settings for all projects | Not in git (personal) | User-wide |
| `.claude/settings.json` | Project root | Team-shared project settings | **Committed to git** | Project-wide (shared) |
| `.claude/settings.local.json` | Project root | Personal project overrides | **Auto-ignored by git** | Project-wide (personal) |
| `managed-settings.json` | System | Enterprise policies | Managed by IT | Enterprise |

**Key Points:**
- `.claude/settings.local.json` is **automatically gitignored** by Claude Code (no manual .gitignore entry needed)
- Use `.json` extension (NOT `.js` - this was a misconception)
- Settings are merged hierarchically (see Precedence section)

---

## Configuration Precedence

Settings are applied in order from **highest to lowest priority**:

1. **Enterprise managed policies** (`managed-settings.json`) - Cannot be overridden
2. **Command line arguments** - Temporary session overrides
3. **Local project settings** (`.claude/settings.local.json`) - Personal project preferences
4. **Shared project settings** (`.claude/settings.json`) - Team-shared configuration
5. **User settings** (`~/.claude/settings.json`) - Personal global defaults

**Example:**
```
Enterprise denies Bash(curl:*) → Cannot be overridden by any other setting
No enterprise policy → settings.local.json allow rules take precedence over settings.json
```

---

## MCP Server Configuration

### MCP Installation Scopes

Claude Code supports three MCP configuration scopes:

| Scope | Storage | Accessibility | Use Case | Command |
|-------|---------|---------------|----------|---------|
| **Local** | User settings (project-specific) | Current project only (private) | Personal dev servers, experimental configs | `claude mcp add my-server /path` (default) |
| **Project** | `.mcp.json` in project root | All team members (shared via git) | Team-shared tools, required services | `claude mcp add server --scope project /path` |
| **User** | User-global settings | All projects on your machine (private) | Personal utilities across projects | `claude mcp add server --scope user /path` |

### .mcp.json File Format

The project-scoped `.mcp.json` file lives at your project root and **should be committed to git**:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "/path/to/server",
      "args": ["arg1", "arg2"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

**Supported Transport Types:**

**1. STDIO (Standard Input/Output):**
```json
{
  "mcpServers": {
    "kit-dev": {
      "command": "npx",
      "args": ["-y", "@kitprotocol/mcp-server-kit"],
      "env": {
        "KIT_API_KEY": "${KIT_API_KEY:-}"
      }
    }
  }
}
```

**2. SSE (Server-Sent Events):**
```json
{
  "mcpServers": {
    "api-server": {
      "type": "sse",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

### Environment Variable Expansion

Claude Code supports environment variable expansion in `.mcp.json`:

- `${VAR}` - Expands to the value of `VAR`
- `${VAR:-default}` - Uses `VAR` if set, otherwise `default`

**Expansion works in:**
- `command` - Server executable path
- `args` - Command-line arguments
- `env` - Environment variables passed to server
- `url` - For SSE/HTTP server types
- `headers` - For SSE/HTTP authentication

### MCP Server Approval

Control which `.mcp.json` servers are auto-approved in `.claude/settings.json`:

```json
{
  "enableAllProjectMcpServers": true,  // Auto-approve ALL servers (convenient but less secure)
  "enabledMcpjsonServers": ["kit-dev", "github"],  // Approve specific servers only
  "disabledMcpjsonServers": ["filesystem"]  // Block specific servers
}
```

**Security Note:** Claude Code prompts for approval before using project-scoped servers from `.mcp.json` unless `enableAllProjectMcpServers: true`. Reset approvals with `claude mcp reset-project-choices`.

---

## Tool Permissions System

### Permission Rule Format

Permission rules follow the pattern: `ToolName(pattern)` or just `ToolName`

**Permission Arrays:**
- **`allow`**: Auto-approve tool use (no prompts)
- **`ask`**: Prompt for confirmation (takes precedence over `allow`)
- **`deny`**: Block tool use entirely (takes precedence over `allow` and `ask`)

### Wildcard Support by Tool Type

| Tool Type | Wildcard Support | Pattern Type | Examples |
|-----------|------------------|--------------|----------|
| **Bash** | `:*` at end only | **Prefix matching** (NOT regex) | `Bash(npm run test:*)`, `Bash(git diff:*)` |
| **Read/Edit** | Full glob patterns | **Glob** (gitignore spec) | `Read(./src/**/*.ts)`, `Edit(/docs/**)` |
| **WebFetch** | Domain matching | Domain-based | `WebFetch(domain:example.com)` |
| **MCP** | **NO wildcards** | Exact tool names | `mcp__github`, `mcp__github__get_issue` |
| **Tool-only** | N/A | Entire tool | `WebFetch`, `Bash` |

### Bash Permissions (Prefix Matching)

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run build)",      // Exact command match
      "Bash(npm run test:*)",     // Prefix match (any test:* command)
      "Bash(git status:*)",       // Prefix match for git status
      "Bash(echo:*)"              // Prefix match for echo commands
    ],
    "deny": [
      "Bash(curl:*)",             // Block curl commands
      "Bash(rm:*)"                // Block rm commands
    ]
  }
}
```

**IMPORTANT Bash Limitations:**
- Uses **prefix matching**, NOT regex
- Wildcard `:*` only works at the end
- **Can be bypassed** in many ways:
  - Options before URL: `curl -X GET http://site.com` bypasses `Bash(curl http://site.com/:*)`
  - Variables: `URL=http://site.com && curl $URL` bypasses pattern matching
  - Shell operators and redirects
- **Do NOT rely on Bash permissions for security** - use for convenience only

### Read/Edit Permissions (Glob Patterns)

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",           // Current directory .env
      "Read(./.env.*)",         // All .env.* files
      "Read(./secrets/**)",     // All files in secrets/ (relative to cwd)
      "Read(/config/**)",       // Relative to settings file location
      "Read(~/Documents/**)",   // Relative to home directory
      "Read(//tmp/secrets/**)"  // Absolute filesystem path
    ]
  }
}
```

**Path Pattern Types:**

| Pattern | Meaning | Example | Matches |
|---------|---------|---------|---------|
| `//path` | **Absolute** filesystem path | `Read(//Users/alice/secrets/**)` | `/Users/alice/secrets/**` |
| `~/path` | **Home** directory | `Read(~/Documents/*.pdf)` | `/Users/alice/Documents/*.pdf` |
| `/path` | **Relative to settings file** | `Edit(/src/**/*.ts)` | `<settings-file-path>/src/**/*.ts` |
| `path` or `./path` | **Relative to cwd** | `Read(*.env)` | `<cwd>/*.env` |

### MCP Permissions (NO Wildcards)

```json
{
  "permissions": {
    "allow": [
      "mcp__github",                    // ✅ Approves ALL GitHub tools
      "mcp__github__get_issue",         // ✅ Approves specific tool
      "mcp__github__list_issues",       // ✅ Approves specific tool
      "mcp__kit-dev__extract_symbols"   // ✅ Approves specific Kit MCP tool
    ],
    "deny": [
      "mcp__puppeteer"                  // ❌ Blocks ALL Puppeteer tools
    ]
  }
}
```

**IMPORTANT MCP Limitations:**
- **NO wildcard support**: Don't use `mcp__github__*` (won't work)
- To approve all tools from a server: Use `mcp__github` (without trailing `__*`)
- To approve specific tools: List each one individually

### Permission Modes

Set `defaultMode` to control default permission behavior:

```json
{
  "permissions": {
    "defaultMode": "acceptEdits"  // Options: "default", "acceptEdits", "plan", "bypassPermissions"
  }
}
```

| Mode | Description |
|------|-------------|
| `default` | Standard behavior - prompts on first use |
| `acceptEdits` | Auto-accepts file edits for the session |
| `plan` | Plan Mode - analyze only, no modifications |
| `bypassPermissions` | Skips all prompts (requires safe environment) |

---

## Subagent Configuration

### Subagent Tool Access

Subagents spawned via the Task tool have two modes for tool access:

1. **Inherit all tools** (default) - Omit the `tools:` field
2. **Explicit tool list** - Specify individual tools

### Agent Frontmatter Format

```markdown
---
name: your-agent-name
description: When this agent should be invoked
tools: tool1, tool2, tool3  # Optional - inherits all if omitted
model: sonnet  # Optional - "sonnet", "opus", "haiku", or "inherit"
---

Agent system prompt goes here...
```

### MCP Access for Subagents

**Key Finding:** When the `tools:` field is omitted, subagents **inherit all MCP tools** available to the main thread.

**Example - Inherits All Tools (Including MCP):**
```markdown
---
name: code-reviewer
description: Expert code review specialist
# tools field omitted - inherits ALL tools including MCP
model: inherit
---

You are a senior code reviewer...
```

This agent will have access to:
- All built-in Claude Code tools (Read, Edit, Bash, etc.)
- **All MCP tools** from configured servers (Kit MCP, Ref MCP, etc.)

**Example - Explicit Tool Restriction:**
```markdown
---
name: test-runner
description: Runs tests and fixes failures
tools: Read, Grep, Bash, mcp__kit-dev__extract_symbols
model: sonnet
---

You are a test automation expert...
```

This agent will have access to:
- Only: `Read`, `Grep`, `Bash`
- Only the specific MCP tool: `mcp__kit-dev__extract_symbols`
- **NOT** other MCP tools or built-in tools like `Edit`, `Write`

### Permission Inheritance

**Subagents DO inherit permission settings** from the main thread's settings files:

1. **Tool availability** is controlled by the `tools:` field in agent frontmatter
2. **Permission rules** (allow/ask/deny) are inherited from settings.json
3. **MCP server access** is inherited if tools are not restricted

**Example Permission Flow:**

**Settings:** `.claude/settings.json`
```json
{
  "permissions": {
    "allow": ["Bash(npm run test:*)", "mcp__github"],
    "deny": ["Bash(curl:*)"]
  }
}
```

**Subagent:** `.claude/agents/test-runner.md`
```markdown
---
name: test-runner
tools: Read, Bash, mcp__github__get_issue
---
```

**Result:**
- Agent can use: `Read`, `Bash`, `mcp__github__get_issue`
- `Bash(npm run test:*)` is auto-approved (inherited from settings)
- `Bash(curl:*)` is denied (inherited from settings)
- Other GitHub tools are **not available** (not in `tools:` list)

### Managing Subagent Tools

**Recommended:** Use `/agents` command for interactive tool management:
```bash
/agents  # Opens interactive menu showing ALL available tools (including MCP)
```

The interface shows:
- All built-in tools
- All MCP tools from connected servers
- Easy checkbox selection
- Direct editing of tool lists

### Verifying MCP Access

To verify MCP tools are accessible to agents:

1. **Check MCP server status:**
   ```bash
   claude mcp list
   ```

2. **View available tools in agent creation:**
   ```bash
   /agents  # Shows all tools including MCP in UI
   ```

3. **Test in main conversation:**
   Ask Claude: "List all available tools including MCP tools"

4. **Check agent execution:**
   - When agent runs, it will show which tools it attempted to use
   - MCP tool errors indicate connection or permission issues

---

## Best Practices for Teams

### Recommended File Structure

```
project/
├── .claude/
│   ├── settings.json           # ✅ Commit to git (team-shared)
│   ├── settings.local.json     # ❌ Auto-ignored (personal)
│   ├── agents/
│   │   ├── code-reviewer.md    # ✅ Commit to git
│   │   └── test-runner.md      # ✅ Commit to git
│   └── commands/
│       └── custom-cmd.md       # ✅ Commit to git
├── .mcp.json                   # ✅ Commit to git (team MCP servers)
├── CLAUDE.md                   # ✅ Commit to git (project instructions)
└── .gitignore                  # settings.local.json auto-ignored
```

### What Goes in Each File

**`.claude/settings.json` (Team-Shared, Committed):**
- Team-wide permission rules
- Project-required MCP server approvals
- Safe command approvals
- Sensitive file denials
- Project environment variables (non-secret)

**`.claude/settings.local.json` (Personal, Not Committed):**
- Personal tool preferences
- Additional MCP tool approvals
- Developer-specific shortcuts
- Local environment variable overrides
- Personal default modes

**`.mcp.json` (Team-Shared, Committed):**
- Required MCP servers for the project
- Server configurations with env var placeholders
- Team-shared tool integrations

**`~/.claude/settings.json` (User-Global, Not Committed):**
- Personal cross-project preferences
- Global permission modes
- Personal cleanup settings

### Secret Management

**NEVER commit to git:**
- API keys
- Authentication tokens
- AWS credentials
- Database passwords
- `.env` files

**Recommended approaches:**

**1. Environment Variables in `.mcp.json`:**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

Team members set locally:
```bash
export GITHUB_TOKEN="ghp_your_token_here"
```

**2. settings.local.json for Overrides:**
```json
{
  "env": {
    "ANTHROPIC_API_KEY": "sk-ant-your-key-here",
    "GITHUB_TOKEN": "ghp_your-token-here"
  }
}
```

**3. API Key Helper Script:**

`.claude/settings.json` (committed):
```json
{
  "apiKeyHelper": "/usr/local/bin/get-claude-key.sh"
}
```

Script retrieves keys from secure storage (1Password, AWS Secrets Manager, etc.)

### Team Configuration Template

**`.claude/settings.json` (Team-Shared):**
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test)",
      "Bash(npm run test:unit:*)",
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "mcp__github__get_issue",
      "mcp__github__list_issues",
      "mcp__kit-dev__extract_symbols",
      "mcp__kit-dev__get_file_tree"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(npm publish:*)",
      "mcp__github__create_pull_request"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Bash(curl:*)",
      "Bash(rm -rf:*)",
      "WebFetch"
    ],
    "defaultMode": "default"
  },
  "enableAllProjectMcpServers": false,
  "enabledMcpjsonServers": ["github", "kit-dev"],
  "env": {
    "NODE_ENV": "development"
  }
}
```

**`.mcp.json` (Team-Shared):**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "kit-dev": {
      "command": "npx",
      "args": ["-y", "@kitprotocol/mcp-server-kit"],
      "env": {
        "KIT_API_KEY": "${KIT_API_KEY:-}"
      }
    }
  }
}
```

---

## Security Considerations

### Preventing Sensitive File Access

**In `.claude/settings.json`:**
```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Read(./.aws/**)",
      "Read(~/.ssh/**)",
      "Edit(./.env:*)",
      "Edit(./secrets/**)"
    ]
  }
}
```

### Bash Command Restrictions

**Remember:** Bash permissions can be bypassed. Use for convenience, not security.

**Safe approach:**
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test:*)",
      "Bash(git status)"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(git commit:*)"
    ],
    "deny": [
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(rm -rf:*)"
    ]
  }
}
```

### Enterprise Policy Enforcement

For organizations requiring strict controls:

**`/Library/Application Support/ClaudeCode/managed-settings.json` (macOS):**
```json
{
  "permissions": {
    "deny": [
      "Read(./.env:*)",
      "Read(./secrets/**)",
      "Bash(curl:*)",
      "Bash(wget:*)",
      "WebFetch"
    ],
    "disableBypassPermissionsMode": "disable"
  },
  "forceLoginMethod": "console",
  "forceLoginOrgUUID": "your-org-uuid-here"
}
```

Enterprise policies **cannot be overridden** by user or project settings.

---

## Example Configurations

### Example 1: Open Source Project

**`.claude/settings.json`:**
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run:*)",
      "Bash(git status)",
      "Bash(git diff:*)",
      "mcp__github"
    ],
    "deny": [
      "Read(./.env:*)"
    ]
  },
  "enableAllProjectMcpServers": true
}
```

**`.mcp.json`:**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

### Example 2: Enterprise Secure Project

**`.claude/settings.json`:**
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test:unit:*)",
      "Bash(git status)"
    ],
    "ask": [
      "Bash(npm run:*)",
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "mcp__github__create_pull_request"
    ],
    "deny": [
      "Read(./.env:*)",
      "Read(./secrets/**)",
      "Read(./config/prod-*.json)",
      "Bash(curl:*)",
      "Bash(wget:*)",
      "WebFetch"
    ],
    "defaultMode": "default"
  },
  "enableAllProjectMcpServers": false,
  "enabledMcpjsonServers": []
}
```

**No `.mcp.json`** - MCP servers configured locally per developer

### Example 3: Data Science Project

**`.claude/settings.json`:**
```json
{
  "permissions": {
    "allow": [
      "Bash(python:*)",
      "Bash(jupyter:*)",
      "mcp__kit-dev__extract_symbols",
      "mcp__kit-dev__deep_research_package"
    ],
    "deny": [
      "Read(./.env:*)",
      "Read(./data/sensitive/**)"
    ]
  },
  "enableAllProjectMcpServers": false,
  "enabledMcpjsonServers": ["kit-dev"]
}
```

**`.mcp.json`:**
```json
{
  "mcpServers": {
    "kit-dev": {
      "command": "npx",
      "args": ["-y", "@kitprotocol/mcp-server-kit"]
    }
  }
}
```

---

## Summary

**Key Takeaways:**

1. **settings.json** (committed) = Team-shared configuration
2. **settings.local.json** (auto-ignored) = Personal overrides
3. **.mcp.json** (committed) = Team MCP servers with env var placeholders
4. **Subagents inherit MCP access** when `tools:` field is omitted
5. **MCP permissions don't support wildcards** - use `mcp__server` for all tools
6. **Bash permissions can be bypassed** - use for convenience, not security
7. **Environment variables** in `.mcp.json` allow secret management without committing keys

**For Team Success:**

- Commit `.claude/settings.json` and `.mcp.json` to git
- Document required environment variables in README/setup docs
- Use `deny` rules to prevent access to sensitive files
- Use `ask` rules for dangerous operations (push, publish, etc.)
- Let developers use `settings.local.json` for personal preferences

---

## Additional Resources

- **Settings Reference**: https://docs.claude.com/en/docs/claude-code/settings
- **MCP Configuration**: https://docs.claude.com/en/docs/claude-code/mcp
- **IAM & Permissions**: https://docs.claude.com/en/docs/claude-code/iam
- **Subagents Guide**: https://docs.claude.com/en/docs/claude-code/sub-agents
