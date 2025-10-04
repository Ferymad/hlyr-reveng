# Linear Workspace Setup Guide
**Date**: 2025-10-04
**Purpose**: Configure Linear workspace for AI-assisted solo web SaaS development
**Workflow**: 6-state simplified workflow optimized for speed and clarity

---

## Overview

This guide walks you through setting up a Linear workspace optimized for solo development with Claude Code automation. The design is based on:
- **HumanLayer's 14-state workflow** (simplified to 6 states)
- **Linear best practices** from official docs and power users
- **Web SaaS development patterns** for full-stack work

**Time to complete**: 30-45 minutes

---

## Prerequisites

- [ ] Linear account created (free or paid tier)
- [ ] GitHub account connected to Linear
- [ ] Claude Code with Linear MCP configured
- [ ] This repository cloned with `.claude/` folder

---

## Part 1: Team & Project Setup

### 1.1 Create Your Team

1. Open Linear → Click **"Create team"**
2. **Team name**: `Development` (or your preference)
3. **Team key**: Will auto-generate (e.g., `DEV` or `ENG`)
   - This appears in ticket IDs: `DEV-123`, `ENG-456`
   - Choose something short and recognizable
4. Click **"Create team"**

### 1.2 Create Default Project (Optional)

Projects group related issues for larger initiatives.

1. Go to your team → Click **"Projects"** tab
2. Click **"New project"**
3. **Project name**: `[Your SaaS Name]` or `Main Development`
4. **Description**: Brief description of your project
5. Set as **default project** in team settings

---

## Part 2: Workflow States Configuration

### 2.1 Access Workflow Settings

1. Go to **Team Settings** → **Issue statuses & automations**
2. You'll see Linear's default states (Backlog, Todo, In Progress, Done, Canceled)

### 2.2 Configure 6-State Workflow

Create these **exact states** for Claude Code automation compatibility:

| State Name | Type | Color | Description |
|------------|------|-------|-------------|
| **Triage** | `triage` | Gray | New tickets needing initial review |
| **Research** | `unstarted` | Blue | Investigation and codebase analysis |
| **Planning** | `unstarted` | Purple | Implementation plan creation |
| **In Dev** | `started` | Yellow | Active development |
| **Review** | `started` | Orange | Code review and QA |
| **Done** | `completed` | Green | Shipped and complete |
| **Canceled** | `canceled` | Red | Won't do / duplicate / out of scope |

### 2.3 State Setup Instructions

**For each state above:**

1. Click **"Add status"** or edit existing
2. Set **Name** (exactly as shown above)
3. Choose **Type** from dropdown:
   - `triage` = New issue triage
   - `unstarted` = Not started
   - `started` = Work in progress
   - `completed` = Finished
   - `canceled` = Abandoned
4. Pick **Color** (suggested colors listed, but use your preference)
5. Add **Description** (helps future you remember the purpose)

**Important**: Match the names exactly (`Research`, `Planning`, `In Dev`, `Review`) as slash commands reference these states.

### 2.4 Set Default State

1. In workflow settings, set **Default status for new issues**: `Triage`
2. This ensures all new tickets start at the beginning of your workflow

### 2.5 Workflow Progression Logic

Your workflow should follow this progression:

```
Triage → Research → Planning → In Dev → Review → Done
   ↓                                              ↑
Canceled ←────────────────────────────────────────┘
```

**State Transition Rules**:
- **Triage → Research**: After quick refinement, problem is understood
- **Research → Planning**: After YOU approve research document
- **Planning → In Dev**: After YOU approve implementation plan
- **In Dev → Review**: After PR is created (can be automatic via GitHub integration)
- **Review → Done**: After you merge PR and verify deployment
- **Any state → Canceled**: Can cancel from any state

---

## Part 3: Label Configuration

### 3.1 Workspace-Level Labels

These labels are available across all teams. Create from **Settings → Workspace → Labels**.

| Label Name | Color | Description |
|------------|-------|-------------|
| `Urgent` | Red | Time-sensitive work requiring immediate attention |
| `Blocked` | Orange | Waiting on dependencies or external factors |
| `Breaking Change` | Purple | Requires migration or major version bump |
| `Quick Win` | Green | Small effort, high impact tasks |
| `Bug` | Red | Defects and unexpected behavior |
| `Security` | Dark Red | Security-related work, vulnerabilities |
| `Technical Debt` | Brown | Refactoring, cleanup, code quality |
| `Documentation` | Gray | Docs, README files, guides |

**To create:**
1. Settings → Workspace → Labels
2. Click **"Create label"**
3. Enter **Name** and choose **Color**
4. Add **Description**
5. Leave **Parent** blank (these are top-level labels)
6. Repeat for all 8 workspace labels

### 3.2 Team-Level Labels (Component Areas)

These labels are scoped to your Development team. Create from **Team Settings → Labels**.

| Label Name | Color | Description |
|------------|-------|-------------|
| `Frontend` | Blue | UI, client-side, styling, UX work |
| `Backend` | Green | Server logic, business rules, services |
| `API` | Teal | API design, endpoints, contracts |
| `Database` | Purple | Schema, migrations, queries |
| `Infrastructure` | Orange | DevOps, deployment, monitoring, CI/CD |
| `Testing` | Yellow | Test coverage, QA, automation |

**To create:**
1. Team Settings → Labels
2. Click **"Create label"**
3. Enter **Name** and choose **Color**
4. Scope: **Team only**
5. Repeat for all 6 component labels

### 3.3 Label Strategy & Rules

**Multi-labeling allowed**: Issues can have multiple component labels
- Example: `Frontend` + `API` for full-stack features
- Example: `Backend` + `Database` for data model changes

**Mutual exclusivity**: Process labels like `Urgent` + `Blocked` can coexist
- Use sparingly - most issues don't need process labels
- Only add when truly needed

**Auto-labeling pattern** (configure later with automations):
- Issues mentioning "UI", "component", "styling" → Auto-add `Frontend`
- Issues mentioning "API", "endpoint", "REST" → Auto-add `API`
- Issues mentioning "migration", "schema", "SQL" → Auto-add `Database`

### 3.4 Tech Stack-Specific Labels (Customize Later)

**DECISION POINT**: When you finalize your tech stack, add these optional labels:

**Frontend Framework** (choose one):
- [ ] `React` - If using React
- [ ] `Vue` - If using Vue.js
- [ ] `Angular` - If using Angular
- [ ] `Svelte` - If using Svelte

**Backend Framework** (choose one):
- [ ] `Node.js` / `Express`
- [ ] `Python` / `Django` / `FastAPI`
- [ ] `Go` / `Gin`
- [ ] `Ruby` / `Rails`

**Other Stack Labels** (optional):
- [ ] `GraphQL` - If using GraphQL instead of/alongside REST
- [ ] `TypeScript` - If worth calling out separately
- [ ] `Mobile` - If building mobile apps
- [ ] `[Integration Name]` - Third-party services (e.g., `Stripe`, `Auth0`)

**When to add these**: After you choose your tech stack and run 10-20 tickets through the workflow. Only add if you find yourself wanting to filter by technology.

---

## Part 4: GitHub Integration

### 4.1 Install Linear GitHub Integration

1. Linear Settings → **Integrations** → **GitHub**
2. Click **"Install GitHub app"**
3. Authenticate with GitHub
4. Select repositories to connect
5. Authorize the integration

### 4.2 Configure Auto-Linking

**Branch naming pattern**:
- Linear auto-links PRs when branch includes issue ID
- Format: `feature/DEV-123-description` or `fix/ENG-456-bug-name`

**PR title pattern**:
- Include issue ID: `DEV-123: Add user authentication`
- Linear auto-links when it detects the ID

**Commit message pattern**:
- Reference issue: `git commit -m "DEV-123: Fix login bug"`

### 4.3 Configure Auto-Status Updates

1. Team Settings → **Issue statuses & automations**
2. Scroll to **Git automations** section
3. Configure triggers:

| Git Event | Linear Action |
|-----------|---------------|
| **Branch created** | Move to `In Dev` (optional, or keep manual) |
| **PR opened** | Add comment with PR link |
| **PR merged** | Move to `Review` → Manual verification → Move to `Done` |
| **PR closed without merge** | Add comment, optionally move back to `Planning` |

**Recommended for solo development**:
- ✅ Enable: Auto-comment when PR opened/merged
- ✅ Enable: Auto-link PRs to issues
- ⚠️ Consider: Auto-move to `In Dev` when branch created (can be noisy)
- ❌ Don't use: Auto-close on merge (you want to verify manually)

### 4.4 Personal Git Automation

1. Your Profile → **Preferences** → **Git**
2. Enable:
   - [ ] **Auto-assign issues to me** when I create a branch
   - [ ] **Move to In Progress** when I create a branch (optional)

---

## Part 5: Team Settings & Defaults

### 5.1 Issue Defaults

Team Settings → **Issue defaults**:

- **Default assignee**: `Unassigned` (you'll assign manually or via automation)
- **Default priority**: `Medium (3)` - Normal work priority
- **Default project**: Select your main project (if created)
- **Default estimate**: `Unset` (optional - you can use estimate points if desired)

### 5.2 Priority Levels (Default)

Linear uses 0-4 priority scale:

| Priority | Level | When to Use |
|----------|-------|-------------|
| Urgent | 1 | Critical bugs, security issues, production down |
| High | 2 | Important features with deadlines, major bugs |
| Medium | 3 | Standard development tasks (default) |
| Low | 4 | Nice-to-haves, minor improvements |
| No Priority | 0 | Backlog items not yet prioritized |

**Best practice**: Use `Medium` as default. Only set `Urgent`/`High` when truly needed.

### 5.3 Cycles (Optional but Recommended)

Cycles = sprints. Recommended for solo developers to maintain focus.

1. Team Settings → **Cycles**
2. Enable cycles
3. **Cycle duration**: `2 weeks` (recommended)
4. **Start day**: Monday (or your preference)
5. **Auto-archive completed issues**: Yes
6. **Cool-down period**: 0 days (or 1 day for cleanup)

**Usage**:
- At start of cycle: Add 3-5 issues to current cycle
- Don't overload - it's better to finish 3 issues than start 10
- Unfinished issues auto-move to next cycle

---

## Part 6: Triage Setup (Optional - Advanced)

**Triage** is a special inbox for new issues from integrations (Slack, Sentry, etc.).

### When to enable Triage:
- ✅ If connecting error monitoring (Sentry, Rollbar)
- ✅ If accepting tickets from support tools (Intercom, etc.)
- ✅ If team members or stakeholders can create issues
- ❌ NOT needed if you create all issues yourself

### How to enable:
1. Team Settings → **Triage**
2. Toggle **"Enable triage"**
3. Set **Triage responsibility**: Assign to yourself
4. Configure **Triage rules** (optional - auto-accept certain types)

---

## Part 7: Linear MCP Configuration

### 7.1 Verify Linear MCP Connection

From terminal in this repository:

```bash
# Check if Linear MCP is configured
cat .claude/settings.json | grep -A 5 "linear-server"
```

Should see Linear MCP server configuration with your API key.

### 7.2 Test Linear MCP Tools

Create a test issue to verify connection:

```bash
# In Claude Code
/linear create issue --title "Test: Linear MCP Connection" --team Development
```

Should create an issue and return the issue ID.

### 7.3 Get Team and Status IDs

You'll need these IDs to update `.claude/commands/linear.md` (if you customize it):

**Get Team ID**:
```
Use Linear MCP: mcp__linear-server__list_teams
```

**Get Status IDs**:
```
Use Linear MCP: mcp__linear-server__list_issue_statuses --team "Development"
```

**Get Label IDs**:
```
Use Linear MCP: mcp__linear-server__list_issue_labels
```

**Save these IDs** - you might need them for automation later.

---

## Part 8: Slash Command Integration

### 8.1 Verify Slash Commands

Check that these commands exist in `.claude/commands/`:

- [x] `research_codebase.md` - Automated research workflow
- [x] `create_plan.md` - Interactive planning workflow
- [x] `implement_plan.md` - Plan execution workflow
- [x] `create_handoff.md` - Session continuity
- [x] `resume_handoff.md` - Resume from handoff
- [x] `linear.md` - Linear ticket management

### 8.2 Update Linear Command (If Needed)

If you customized team names or status names differently than this guide:

1. Edit `.claude/commands/linear.md`
2. Update **Team ID** and **Status IDs** to match your Linear workspace
3. Update **Label IDs** to match your labels
4. Update default status names in workflow descriptions

### 8.3 Workflow Mapping

**Map Linear statuses to slash commands**:

| Linear Status | Slash Command | What It Does |
|---------------|---------------|--------------|
| `Triage` | Manual | Quick problem refinement |
| `Research` | `/research_codebase` | Spawns research agents, creates research doc |
| `Planning` | `/create_plan` | Interactive plan creation, creates plan doc |
| `In Dev` | `/implement_plan` | Executes plan phases, creates commits/PR |
| `Review` | Manual | Code review, QA, manual testing |
| `Done` | Manual | Mark complete after merge & deploy |

---

## Part 9: Thoughts Directory Setup

### 9.1 Decide: Separate Repo vs Local Directory

**Option A: Separate Git Repo** (HumanLayer style, recommended)
- ✅ Version controlled knowledge base
- ✅ Can be private while code repo is public
- ✅ Easy to share research/plans across projects
- ❌ Requires sync mechanism
- ❌ More setup complexity

**Option B: Local Directory** (simpler, faster to start)
- ✅ Quick to set up (already exists: `thoughts/`)
- ✅ No sync needed
- ❌ In main repo (add to .gitignore or commit knowledge)
- ❌ Mixed with code in git history

**DECISION**: __________ (Choose A or B when setting up)

### 9.2 If Choosing Option A (Separate Repo):

```bash
# Create separate thoughts repo
cd ~/dev
mkdir myproject-thoughts
cd myproject-thoughts
git init
git remote add origin git@github.com:yourusername/myproject-thoughts.git

# Create structure
mkdir -p shared/{research,plans,handoffs,tickets}

# Initial commit
git add .
git commit -m "Initial thoughts repo structure"
git push -u origin main

# Link to main project
cd ~/dev/your-main-project
rm -rf thoughts  # Remove local thoughts dir
ln -s ~/dev/myproject-thoughts thoughts  # Symlink to thoughts repo
```

**Create sync script** (`bin/thoughts-sync.sh`):
```bash
#!/bin/bash
cd thoughts
git pull --rebase
git add .
git commit -m "Sync: $(date +%Y-%m-%d_%H-%M-%S)" || true
git push
```

### 9.3 If Choosing Option B (Local Directory):

```bash
# Already created at: thoughts/shared/{plans,research,docs,handoffs}

# DECISION: Add to .gitignore or commit?
# If private: echo "thoughts/" >> .gitignore
# If commit: git add thoughts/ && git commit -m "Add thoughts structure"
```

**DECISION**: __________ (Choose .gitignore or commit when setting up)

---

## Part 10: Validation & First Ticket

### 10.1 Create Test Ticket

1. In Linear, create a new issue:
   - **Title**: `Test: Validate AI development workflow`
   - **Description**:
     ```
     Test the full research → plan → implement cycle.

     Acceptance criteria:
     - [ ] Research document created in thoughts/shared/research/
     - [ ] Implementation plan created in thoughts/shared/plans/
     - [ ] Changes implemented and PR created
     - [ ] All workflow states transition correctly
     ```
   - **Team**: Development
   - **Status**: `Triage`
   - **Labels**: Add a component label (e.g., `Backend` or `Frontend`)

2. **Triage the ticket**:
   - Review the description
   - Set **Priority**: Medium
   - Move to **Research** status

### 10.2 Run Research Phase

```bash
# In Claude Code
/research_codebase

# When prompted, tell it about the test ticket
# It will spawn research agents, explore the codebase, and create research doc
```

**Expected output**: `thoughts/shared/research/2025-10-04-test-workflow.md`

### 10.3 Review Research

1. Read the research document
2. Add comments to Linear ticket if you want adjustments
3. Move ticket to **Planning** status when satisfied

### 10.4 Run Planning Phase

```bash
/create_plan thoughts/shared/research/2025-10-04-test-workflow.md
```

**Expected output**: `thoughts/shared/plans/2025-10-04-test-workflow.md`

### 10.5 Review Plan

1. Read the implementation plan
2. Check success criteria (automated vs manual)
3. Verify phasing makes sense
4. Move ticket to **In Dev** when approved

### 10.6 Run Implementation

```bash
/implement_plan thoughts/shared/plans/2025-10-04-test-workflow.md
```

**Expected**:
- Creates branch (or uses existing)
- Implements plan phases
- Runs automated verification
- Creates commits
- Opens PR

### 10.7 Final Review

1. Ticket should auto-move to **Review** when PR created
2. Review the PR in GitHub
3. Merge PR
4. Manually move ticket to **Done**

**If everything worked**: Your workflow is validated! 🎉

---

## Part 11: Optimization & Iteration

### 11.1 After First 5-10 Tickets

**Review and adjust**:

1. **Are labels useful?**
   - Remove labels you never use
   - Add labels you keep wishing existed

2. **Are states clear?**
   - Consider renaming if confusing (but update slash commands too!)
   - Consider adding intermediate states if gaps appear

3. **Is GitHub integration smooth?**
   - Adjust auto-status rules
   - Refine branch naming conventions

4. **Are cycles helpful?**
   - Adjust cycle length if needed
   - Enable/disable based on preference

### 11.2 Advanced Automation (Month 2+)

Once workflow is stable, consider:

1. **GitHub Actions** for automated ticket progression
   - Example: Auto-move to `Review` when PR opened
   - Example: Auto-comment with CI results

2. **Linear Triage Rules** (Business plan)
   - Auto-assign labels based on content
   - Auto-set priority for certain issue types

3. **Custom slash commands** for your specific workflow
   - Add project-specific research patterns
   - Add deployment-specific verification steps

---

## Troubleshooting

### Issue: Linear MCP not connecting
- Check API key in `.claude/settings.json`
- Verify Linear account has API access enabled
- Test with `mcp__linear-server__list_teams` directly

### Issue: Slash commands not working
- Verify `.claude/commands/*.md` files exist
- Check command syntax matches exactly
- Look for errors in Claude Code logs

### Issue: GitHub integration not auto-linking
- Check branch name includes issue ID: `feature/DEV-123-name`
- Verify GitHub app is installed on repository
- Check Linear → GitHub connection in settings

### Issue: Thoughts sync failing
- If separate repo: Check git remote is configured
- Verify write permissions to thoughts directory
- Check for merge conflicts in thoughts repo

### Issue: Status transitions not following workflow
- Review your status types (triage, unstarted, started, completed)
- Check GitHub automation rules aren't conflicting
- Verify you're manually moving through states correctly

---

## Reference: Quick Setup Checklist

Use this checklist when actually performing the setup:

### Linear Configuration
- [ ] Create team named "Development" (or chosen name)
- [ ] Create 6 workflow states: Triage, Research, Planning, In Dev, Review, Done, Canceled
- [ ] Set default state to "Triage"
- [ ] Create 8 workspace labels (Urgent, Blocked, Breaking Change, Quick Win, Bug, Security, Technical Debt, Documentation)
- [ ] Create 6 team labels (Frontend, Backend, API, Database, Infrastructure, Testing)
- [ ] Install GitHub integration
- [ ] Configure auto-linking and auto-commenting
- [ ] Set issue defaults (priority: Medium, project: Main)
- [ ] Enable 2-week cycles (optional but recommended)

### Local Configuration
- [ ] Verify `.claude/commands/` folder exists
- [ ] Verify Linear MCP connection works
- [ ] Get team ID, status IDs, label IDs (save somewhere)
- [ ] Create thoughts directory structure (shared/{research,plans,handoffs,tickets})
- [ ] Decide: Separate thoughts repo OR local directory
- [ ] If separate repo: Set up repo, symlink, sync script
- [ ] If local: Add to .gitignore or commit decision

### Validation
- [ ] Create test ticket in Linear
- [ ] Run `/research_codebase` → verify research doc created
- [ ] Run `/create_plan` → verify plan doc created
- [ ] Run `/implement_plan` → verify branch/commits/PR created
- [ ] Verify GitHub auto-linking worked
- [ ] Complete full cycle: Triage → Research → Planning → In Dev → Review → Done

---

## Next Steps After Setup

1. **Create your first real ticket** from backlog
2. **Document learnings** as you go through workflow
3. **Iterate on labels** - add/remove based on actual usage
4. **Refine cycle planning** - find your sustainable velocity
5. **Consider automation** after 10-20 tickets through the workflow

---

## Support & Resources

- **Linear Documentation**: https://linear.app/docs
- **Linear Method**: https://linear.app/method
- **Claude Code Docs**: (your .claude/commands/ folder)
- **This project's handoff**: `thoughts/shared/handoffs/general/2025-10-04_15-44-24_ai-dev-cycle-design.md`

---

**Document version**: 1.0
**Last updated**: 2025-10-04
**Maintained by**: AI Development Workflow
