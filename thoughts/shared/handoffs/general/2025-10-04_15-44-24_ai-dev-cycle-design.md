---
date: 2025-10-04T14:44:24+0000
researcher: selman
git_commit: a86bf565f2e86ebdcfc3bf032ed29e723a6662ff
branch: main
repository: hlyr-reveng
topic: "AI-Assisted Solo Development Cycle Design"
tags: [strategy, workflow, linear, claude-code, kit-mcp, development-cycle]
status: complete
last_updated: 2025-10-04
last_updated_by: selman
type: implementation_strategy
---

# Handoff: AI-Assisted Solo Development Cycle Design

## Task(s)

**COMPLETED:**
1. ✅ Analyzed HumanLayer's development cycle and workflows
2. ✅ Mapped capabilities of available tools (Kit MCP, Ref MCP, Linear MCP, Claude Code)
3. ✅ Identified key processes to adapt from HumanLayer for solo development
4. ✅ Designed comprehensive AI-assisted solo development cycle

**PLANNED/DISCUSSED (Next Session):**
1. Design Linear workspace (statuses, labels, workflow configuration)
2. Adapt copied slash commands from HumanLayer for solo use
3. Run through first ticket end-to-end as practical demonstration
4. Set up thoughts directory structure
5. Create starter ticket to validate the workflow

## Critical References

- `.claude/commands/create_plan.md` - Interactive planning workflow
- `.claude/commands/implement_plan.md` - Plan execution workflow
- `.claude/commands/research_codebase.md` - Parallel research with subagents
- `.claude/commands/create_handoff.md` - Session continuity system
- `.claude/commands/resume_handoff.md` - Handoff resumption workflow

## Recent changes

No code changes made - this was pure strategy and analysis session.

## Learnings

### HumanLayer's Development Philosophy

1. **Plan-first, review-early principle**: Alignment happens at the PLAN stage, not PR stage
   - Research → Plan → Implement workflow with human review gates
   - Plans include both automated and manual success criteria
   - Plans are phased with explicit "what we're NOT doing" sections

2. **Autonomous AI execution pattern**:
   - GitHub Actions trigger every 15 minutes on schedule
   - Claude Opus runs with `--dangerously-skip-permissions` in CI
   - Linear status changes trigger automated workflows
   - Files: `.github/workflows/linear-research-tickets.yml`, `linear-create-plan.yml`, `linear-implement-plan.yml`

3. **Knowledge management architecture**:
   - Separate `thoughts` git repository (not in code repo)
   - Structure: `thoughts/shared/{research,plans,handoffs,tickets}/`
   - Hard links in `thoughts/searchable/` for grep without following symlinks
   - Synced via `humanlayer thoughts sync` command

4. **Subagent orchestration**:
   - Specialized agents: codebase-locator, codebase-analyzer, pattern-finder, thoughts-locator, thoughts-analyzer
   - Spawn parallel tasks for comprehensive research
   - Each agent has specific instructions in `.claude/agents/*.md`
   - Wait for ALL agents before synthesizing findings

5. **Linear workflow states** (from `.claude/commands/linear.md:32-48`):
   - Triage → Spec Needed → Research Needed → Research in Progress → Research in Review
   - → Ready for Plan → Plan in Progress → Plan in Review → Ready for Dev
   - → In Dev → Code Review → Done
   - Key insight: Multiple review gates before implementation starts

### Tool Capabilities Mapped

**Kit MCP (repo_2):**
- `open_repository` - Load multiple repos into memory (we opened 3 repos)
- `extract_symbols` - Fast cached symbol extraction
- `find_symbol_usages` - Cross-file usage tracking
- `grep_code` - Literal string search with filters
- `grep_ast` - Semantic code pattern search
- `review_diff` - AI-powered diff analysis
- Architecture: Incremental analysis with caching (`src/kit/incremental_analyzer.py`)

**Ref MCP:**
- `ref_search_documentation` - Web + GitHub + private docs search
- `ref_read_url` - Fetch and parse documentation
- `deep_research_package` - Comprehensive package research with Q&A

**Linear MCP:**
- Full CRUD for issues, comments, projects, labels
- Status workflow automation
- Team and user management
- Label auto-assignment based on content (hld, wui, meta)

**Claude Code:**
- Subagent system with Task tool
- TodoWrite for session task tracking
- Handoff system for session continuity
- Slash commands for standardized workflows

## Artifacts

This session produced strategic analysis only (no code artifacts). Key conceptual deliverables:

1. **HumanLayer Development Cycle Analysis**:
   - 3-phase workflow: Research → Planning → Implementation
   - Linear status state machine with 14+ states
   - GitHub Actions automation patterns
   - Thoughts system architecture

2. **Tool Capability Matrix**:
   - Kit MCP: 10+ tools for codebase intelligence
   - Ref MCP: Documentation research capabilities
   - Linear MCP: Full project management integration
   - Claude Code: Orchestration and workflow automation

3. **Solo Development Cycle Design**:
   - Adapted workflow removing team coordination
   - Manual trigger approach (no CI initially)
   - Smaller scope per session with frequent handoffs
   - Review gates for you to approve research/plans/implementation
   - Simplified thoughts structure

4. **Implementation Recommendations**:
   - Phase 1: Foundation setup (Week 1)
   - Phase 2: Optimization (Weeks 2-3)
   - Phase 3: Automation (Month 2+)
   - Path A (recommended): Start small with one ticket
   - Path B: Set up infrastructure first
   - Path C: Full automation

## Action Items & Next Steps

### Immediate Actions (For Next Agent):

1. **Design Linear Workspace**:
   - Create custom workflow states adapted from HumanLayer's 14-state flow
   - Simplified for solo: Triage → Research → Plan → Implement → Review → Done
   - Set up labels: Pick domain-specific labels (adapt hld/wui/meta pattern)
   - Configure team settings and default project
   - Document status progression rules

2. **Adapt Slash Commands**:
   - Review `.claude/commands/research_codebase.md` - remove team references
   - Review `.claude/commands/create_plan.md` - adapt for solo workflow
   - Review `.claude/commands/implement_plan.md` - add manual approval gates
   - Keep: `create_handoff.md`, `resume_handoff.md`, `commit.md`, `describe_pr.md`
   - Remove/simplify: `linear.md` (team-specific), CI-specific commands

3. **Set Up Thoughts Directory**:
   - Decide: Local directory OR separate git repo (like HumanLayer)
   - Create structure: `thoughts/{research,plans,handoffs}/`
   - If git repo: Set up sync mechanism
   - If local: Ensure it's in .gitignore

4. **Create Starter Ticket**:
   - Pick a small, real task from the user's backlog
   - Create Linear ticket manually
   - Run through full cycle: Research → Plan → Implement
   - Document pain points and iteration opportunities

5. **Validate Workflow End-to-End**:
   - Use Kit MCP to research (`open_repository` → `grep_code` → `extract_symbols`)
   - Use `/research_codebase` command
   - Review research output, iterate
   - Use `/create_plan` command
   - Review plan, iterate
   - Use `/implement_plan` command with approval gates
   - Create PR with `/describe_pr`

### Questions to Ask User (Next Session):

1. Linear workspace preferences:
   - Use simplified 6-state workflow or full HumanLayer 14-state?
   - What labels/tags are relevant to their projects?
   - Default project name?

2. Thoughts system:
   - Separate git repo (HumanLayer style) or local directory?
   - Auto-sync or manual sync?
   - Public or private repo?

3. Automation timeline:
   - Stick with manual workflow initially (recommended)?
   - Or jump straight to GitHub Actions setup?

4. Starter ticket:
   - What's a good first task to validate the workflow?
   - Should it be from existing backlog or a test task?

## Other Notes

### Key Files to Reference (from repos opened):

**HumanLayer repo (repo_1):**
- `.claude/commands/` - All slash command definitions
- `.claude/agents/` - Subagent prompt templates
- `.github/workflows/linear-*.yml` - CI automation examples (lines 1-206 each)
- `hlyr/THOUGHTS.md` - Thoughts system documentation
- `CLAUDE.md` - Workflow documentation

**Kit repo (repo_2):**
- `src/kit/mcp/dev_server.py` - MCP server implementation
- `src/kit/incremental_analyzer.py` - Symbol caching system
- `README.md` - Usage patterns and best practices

**This repo (repo_0):**
- `.claude/` - Already copied from HumanLayer
- `CLAUDE.md` - Kit MCP usage guidelines (already present)

### Implementation Philosophy Insights:

1. **Plans are executable**: Include specific file paths, code snippets, and checkable success criteria
2. **Success criteria split**: Automated (make test, lint, etc.) vs Manual (UI testing, performance)
3. **Subagents maximize efficiency**: Parallel research tasks prevent linear blocking
4. **Handoffs enable continuity**: Critical for solo work across sessions
5. **Linear is the source of truth**: Ticket status drives next actions

### Recommended First Steps After This Handoff:

1. Open new Claude Code session with: `/resume_handoff thoughts/shared/handoffs/general/2025-10-04_15-44-24_ai-dev-cycle-design.md`
2. Start with Linear workspace design (30 min)
3. Review and adapt 3 core commands: research, plan, implement (45 min)
4. Create first real ticket (15 min)
5. Run through research phase manually (60 min)
6. Iterate based on what feels smooth vs. clunky

### Context from This Session:

- User already has Linear account (needs workspace design)
- User already copied HumanLayer's `.claude/` folder
- User chose Path A: Start small, iterate gradually
- User wants to use Kit MCP + Ref MCP + Linear MCP together
- User is building solo development workflow (no team coordination needed)
- Three repos are already opened: hlyr-reveng (repo_0), humanlayer (repo_1), kit (repo_2)

### Workflow Visualization Created:

```
IDEA/BUG
   ↓
[Linear Ticket] → Triage → Research Needed
   ↓
[/research_codebase] → Research doc → Research in Review (USER REVIEWS)
   ↓
[Approve] → Ready for Plan
   ↓
[/create_plan] → Plan doc → Plan in Review (USER REVIEWS)
   ↓
[Approve/Refine] → Ready for Dev
   ↓
[/implement_plan] → Executes → Creates PR → Code Review
   ↓
[Merge] → Done
```

This is a human-in-the-loop adaptation of HumanLayer's fully automated CI workflow.
