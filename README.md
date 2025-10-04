# hlyr-reveng

Reverse engineering project with Claude Code MCP integration.

## Claude Code Setup

This repository uses Claude Code with MCP (Model Context Protocol) servers for enhanced code analysis and documentation research.

### Prerequisites

- **Claude Code**: https://docs.claude.com/en/docs/claude-code/setup
- **Node.js 18+** (for npx to run MCP servers)

### Quick Start

1. **Clone the repository:**
   ```bash
   git clone <repo-url>
   cd hlyr-reveng
   ```

2. **Open Claude Code:**
   ```bash
   claude
   ```

3. **Approve MCP servers when prompted:**
   - ✅ **kit-dev** - Code analysis and AST tools
   - ✅ **ref** - Documentation search and research

4. **Done!** Agents will use MCP tools automatically.

### Optional: Kit MCP API Key

For higher rate limits, set the Kit MCP API key:

```bash
export KIT_API_KEY="your_key_here"
```

Get your free key from: https://kit-mcp.cased.com/

### Verify Setup

Check MCP server status:

```bash
claude mcp list
```

You should see **kit-dev** and **ref** listed.

### MCP-Enhanced Agents

This repository includes 6 MCP-enhanced agents in `.claude/agents/`:

| Agent | Purpose | MCP Tools |
|-------|---------|-----------|
| **codebase-locator** | Find files and components | Kit MCP AST tools |
| **codebase-analyzer** | Analyze code structure | Kit MCP symbols/deps |
| **codebase-pattern-finder** | Find code patterns | Kit MCP pattern tools |
| **external-doc-researcher** | Research documentation | Ref MCP |
| **mcp-package-researcher** | Deep package research | Kit + Ref MCP |
| **codebase-dependency-tracer** | Trace dependencies | Kit MCP |

Agents automatically use MCP tools when available, or gracefully degrade to basic tools if not configured.

### Configuration Files

- **`.mcp.json`** - Team-shared MCP server configs (committed to git)
- **`.claude/settings.json`** - Team permissions and approvals (committed to git)
- **`.claude/settings.local.json`** - Personal overrides (auto-ignored by git)

### Troubleshooting

**MCP servers not starting:**
```bash
# Check if npx can find the packages
npx @kitprotocol/mcp-server-kit --version
npx @modelcontextprotocol/server-ref --version

# Reset MCP server approvals
claude mcp reset-project-choices
```

**Tools not available to agents:**
- Verify MCP servers are running: `claude mcp list`
- Check `.claude/settings.json` has `enabledMcpjsonServers: ["kit-dev", "ref"]`
- Restart Claude Code

### Resources

- **Claude Code Documentation**: https://docs.claude.com/en/docs/claude-code
- **Kit MCP**: https://kit-mcp.cased.com/
- **Ref MCP**: https://ref.tools/
- **MCP Integration Guide**: `thoughts/shared/research/claude-code-configuration-guide.md`
