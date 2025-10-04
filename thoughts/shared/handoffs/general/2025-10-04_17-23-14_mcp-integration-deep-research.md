---
date: 2025-10-04T17:23:14+01:00
researcher: selman
git_commit: 6a9d869e46e8d754c237edbbb13af163489949b5
branch: main
repository: hlyr-reveng
topic: "MCP Integration Strategy - Deep Research & Architecture Design"
tags: [mcp, kit-mcp, ref-mcp, architecture, integration, workflow, research, strategy, humanlayer]
status: complete
last_updated: 2025-10-04
last_updated_by: selman
type: implementation_strategy
---

# Handoff: MCP Integration Strategy - Deep Research Complete

## Task(s)

**Status: Research Phase Complete ✅ | Next: Implementation Phase**

### Completed Tasks:
1. ✅ **Deep research Kit MCP capabilities** - Comprehensive analysis of all 10+ tools, caching architecture, performance characteristics
2. ✅ **Deep research Ref MCP capabilities** - Token efficiency analysis, session-aware search, query patterns
3. ✅ **Analyze HumanLayer slash commands** - Complete workflow architecture, Linear integration, all 13 commands
4. ✅ **Analyze HumanLayer subagents** - Prompting excellence patterns, tool usage, 6 specialized agents
5. ✅ **Analyze HumanLayer CLAUDE.md patterns** - Instruction architecture, workflow guardrails
6. ✅ **Create comprehensive integration strategy document** - 12-part strategy with visual decision trees
7. ✅ **Investigate deep_research_package vs Ref overlap** - Complementary usage strategy defined

### Current Status:
- All research complete
- Integration strategy documented in `thoughts/shared/handoffs/general/2025-10-04_17-06-19_mcp-integration-strategy.md`
- Ready to begin implementation: Enhanced agents and `/research_codebase` command

### Next Phase:
- Implement Phase 1: Create MCP-enhanced agent definitions
- Implement Phase 2: Perfect `/research_codebase` command with full MCP integration

## Critical References

1. **Integration Strategy Document** (PRIMARY): `thoughts/shared/handoffs/general/2025-10-04_17-06-19_mcp-integration-strategy.md`
   - Complete 12-part strategy with decision trees
   - Agent-by-agent integration plans
   - 5-week implementation phases
   - Risk mitigations and success criteria

2. **Original Handoff** (CONTEXT): `thoughts/shared/handoffs/general/2025-10-04_15-44-24_ai-dev-cycle-design.md`
   - Initial Linear workspace research
   - Development cycle design decisions

3. **HumanLayer Reference** (SOURCE): `/home/selman/dev/humanlayer/.claude/`
   - Commands: `/home/selman/dev/humanlayer/.claude/commands/`
   - Agents: `/home/selman/dev/humanlayer/.claude/agents/`

## Recent Changes

**Created Documents:**
- `thoughts/shared/handoffs/general/2025-10-04_17-06-19_mcp-integration-strategy.md` - Complete integration strategy (12 parts, 2 appendices)

**No code changes** - This was pure research and strategy phase.

## Learnings

### 1. Kit MCP Architecture Understanding
**Multi-level caching is key**:
- Level 1: Class-level parser cache (instant)
- Level 2: File-level incremental cache with SHA256 invalidation (milliseconds)
- Level 3: Repository-level symbol maps (invalidates on git changes)
- Level 4: Remote repo clone cache (avoids re-cloning)

**Tool selection wisdom**:
- `grep_ast` for AST-aware symbol search (14 languages supported)
- `extract_symbols` for fast symbol inventory (cached!)
- `find_symbol_usages` for tracing where symbols used (no caching, slower)
- `get_dependency_graph` for module relationships (Python & Terraform only)
- `deep_research_package` for comprehensive package overview (multi-source)

**When NOT to use Kit**:
- Simple file reading → Use `Read` (faster, complete)
- Non-code files → Use `Grep` (configs, markdown)
- One-off searches → Use `Grep` (no caching benefit)

### 2. Ref MCP Token Efficiency
**60-95% token reduction** vs traditional web scraping:
- Session-aware search prevents duplicates
- Smart chunking returns ~5k most relevant tokens (not entire page)
- Curated documentation index (1000s of sites + GitHub + private)
- All code tabs loaded (not just curl examples)

**Query patterns that work**:
- ✅ Full sentences: "FastAPI WebSocket authentication middleware best practices"
- ❌ Keywords only: "fastapi websocket"

### 3. deep_research_package vs Ref MCP - They Complement!
**NOT a clash, use strategically**:
- `deep_research_package` → Comprehensive multi-source overview (first time with package)
- Ref MCP → Token-efficient focused deep-dive (iterative refinement)
- Kit package search → Source code patterns

**Pattern**: Overview (deep_research) → Details (Ref) → Source (Kit package search)

### 4. HumanLayer Workflow Brilliance
**Core principles to preserve**:
1. **"Document what IS, not what SHOULD BE"** - All agents are documentarians, not critics
2. **Research → Plan → Implement → Handoff cycle** - Clear phase separation
3. **Interactive planning** - NOT one-shot: Research → Present → Refine → Approve
4. **Critical ordering rules**:
   - Read files FULLY before spawning sub-tasks
   - Wait for ALL sub-agents before synthesizing
   - No placeholders in documents
   - No open questions in final plans
5. **Success criteria bifurcation** - Automated (executable commands) vs Manual (human judgment)

**Prompting excellence patterns**:
- Identity framing: "You are a specialist at..."
- Critical constraint blocks with DO NOT lists
- Deep thinking directives: "ultrathink", "think deeply"
- Example-heavy templates
- Role reinforcement: "REMEMBER: You are a documentarian..."

### 5. Agent Specialization
**Division of labor works**:
- **Locators**: Find WHERE (no reading) - `codebase-locator`, `thoughts-locator`
- **Analyzers**: Understand HOW (deep reading) - `codebase-analyzer`, `thoughts-analyzer`
- **Pattern-finders**: Show EXAMPLES (code snippets) - `codebase-pattern-finder`
- **External**: Research documentation - `web-search-researcher` (rename to `external-doc-researcher`)

**Tool access grows with analytical depth**:
1. Locators: Grep, Glob, LS only
2. Analyzers/Pattern-finders: Add Read
3. External researcher: Add WebSearch, WebFetch

**MCP enhancement pattern**:
- Codebase agents + Kit MCP (AST, symbols, dependencies)
- External researcher + Ref MCP (documentation)
- Thoughts agents: NO changes (already optimal)

### 6. Integration Strategy Key Decisions
**Guiding principles**:
1. **Progressive enhancement** - MCP enhances, doesn't replace
2. **Preserve workflow architecture** - Keep Research → Plan → Implement cycle
3. **Additive tool access** - Add MCP tools, don't remove existing
4. **Strategic MCP usage** - Kit for codebase, Ref for docs, built-ins for simple ops
5. **No behavioral change** - Same outputs, more efficiently

**Agent changes**:
- `codebase-locator`: Add `grep_ast`, `get_file_tree`, `extract_symbols`
- `codebase-analyzer`: Add `find_symbol_usages`, `get_dependency_graph`
- `codebase-pattern-finder`: Add AST pattern matching
- `web-search-researcher` → `external-doc-researcher`: Add Ref MCP
- New: `mcp-package-researcher` (Ref + Kit for packages)
- New: `codebase-dependency-tracer` (Kit dependency mapping)

**Command changes (Priority order)**:
1. `/research_codebase` - Add MCP check, enhanced agents
2. `/create_plan` - MCP-powered research tasks
3. `/implement_plan` - Optional AI review
4. Others - Lower priority

## Artifacts

### Research Agent Reports (from parallel sub-agents):
1. **Kit MCP Full Capabilities Report** - Comprehensive tool inventory, caching strategy, power patterns, limitations
2. **Ref MCP Full Capabilities Report** - What it does, tool details, power patterns, decision trees
3. **HumanLayer Slash Commands Architecture** - Workflow map, command inventory, reusable patterns, MCP integration opportunities
4. **HumanLayer Subagents Deep Analysis** - Agent inventory, prompting excellence, tool usage, MCP integration plan
5. **HumanLayer CLAUDE.md Patterns** - Instruction architecture, workflow guardrails, integration points

### Strategy Documents:
1. **thoughts/shared/handoffs/general/2025-10-04_17-06-19_mcp-integration-strategy.md** (MAIN DELIVERABLE)
   - Part I: MCP Server Capabilities (Kit & Ref deep-dive)
   - Part II: HumanLayer Workflow Analysis (complete architecture)
   - Part III: Integration Strategy (guiding principles)
   - Part IV: Visual Decision Trees (when to use which tool)
   - Part V: Agent-by-Agent Integration Plan (6 agents)
   - Part VI: New Agents to Create (2 new specialized agents)
   - Part VII: Command Integration Plan (priority 1-3)
   - Part VIII: CLAUDE.md Integration
   - Part IX: Implementation Phases (5-week plan)
   - Part X: Risks & Mitigations (6 risks addressed)
   - Part XI: Success Criteria (quantitative & qualitative)
   - Part XII: Quick Reference (cheat sheet)
   - Appendix A: Tool Selection Examples (3 scenarios)
   - Appendix B: Migration Checklist

### Configuration Context:
- Available MCP servers: Kit MCP (27.7k tokens), Ref MCP (1.3k tokens), Linear MCP (16.6k tokens)
- Existing agents: 6 (codebase-locator, codebase-analyzer, codebase-pattern-finder, web-search-researcher, thoughts-locator, thoughts-analyzer)
- Repository context: 3 repos open via Kit (hlyr-reveng, humanlayer, kit)

## Action Items & Next Steps

### Immediate Next Steps (Phase 1 - Week 1):

1. **Create MCP-Enhanced Agent Definitions**:
   - [ ] `codebase-locator` → Add Kit tools (`grep_ast`, `get_file_tree`, `extract_symbols`)
   - [ ] `codebase-analyzer` → Add Kit tools (`find_symbol_usages`, `get_dependency_graph`)
   - [ ] `codebase-pattern-finder` → Add Kit AST pattern tools
   - [ ] `web-search-researcher` → Rename to `external-doc-researcher`, add Ref tools
   - [ ] Create `mcp-package-researcher` (new agent)
   - [ ] Create `codebase-dependency-tracer` (new agent)

2. **Test Agent Graceful Degradation**:
   - [ ] Test each agent with MCP available
   - [ ] Test each agent with MCP unavailable (fallback to built-in tools)
   - [ ] Validate outputs maintain same structure

3. **Update CLAUDE.md**:
   - [ ] Add MCP integration guidelines
   - [ ] Add tool selection decision tree
   - [ ] Add repository initialization pattern
   - [ ] Add graceful degradation notes

### Phase 2 (Week 2) - Perfect Research Command:

4. **Enhance `/research_codebase` Command**:
   - [ ] Add MCP availability check at start
   - [ ] Update agent spawning to use MCP-enhanced versions
   - [ ] Add `mcp-package-researcher` option for external dependencies
   - [ ] Test full workflow end-to-end
   - [ ] Validate research document quality improvement

### Phase 3+ (Weeks 3-5):
5. [ ] Planning command integration
6. [ ] Implementation & review integration
7. [ ] Optimization & monitoring
8. [ ] Document learnings and best practices

### Critical Success Criteria to Track:
- [ ] 50%+ token reduction for external documentation (Ref MCP)
- [ ] Faster symbol discovery (Kit MCP caching)
- [ ] All workflow guardrails preserved (read fully, wait for agents, no placeholders)
- [ ] Documentarian role maintained (no critique/suggestions)
- [ ] Interactive planning still works

## Other Notes

### Important File Locations:

**HumanLayer Reference (source material)**:
- Commands: `/home/selman/dev/humanlayer/.claude/commands/*.md`
  - `/research_codebase.md` - Research workflow (lines 26-29: critical file reading rule)
  - `/create_plan.md` - Planning workflow (lines 331-336: no open questions rule)
  - `/linear.md` - Linear integration (lines 353-357: label IDs, 381-384: user IDs)
- Agents: `/home/selman/dev/humanlayer/.claude/agents/*.md`
  - All share critical constraint: "DO NOT suggest/critique/improve"
  - All end with "REMEMBER" reinforcement sections

**Kit MCP Source Code** (for understanding tools):
- `/home/selman/dev/kit/src/kit/mcp/dev_server.py` - MCP tool definitions
- `/home/selman/dev/kit/src/kit/repository.py` - Core repository operations
- `/home/selman/dev/kit/src/kit/ast_search.py` - AST search implementation
- `/home/selman/dev/kit/src/kit/tree_sitter_symbol_extractor.py` - Symbol extraction

**Project Structure** (current repo):
- `.claude/agents/` - 6 existing agents to enhance
- `.claude/commands/` - Empty (need to create commands)
- `CLAUDE.md` - Has basic Kit MCP guidance, needs enhancement
- `thoughts/` - Shared handoffs location

### Tool Approval Settings:

The following tools are pre-approved (no user confirmation needed):
- `Bash(./hack/spec_metadata.sh)`
- `mcp__kit-dev__*` (all Kit MCP tools)
- `mcp__Ref__*` (all Ref MCP tools)
- `Read(//home/selman/dev/humanlayer/**)`
- `Read(//home/selman/dev/kit/src/**)`

### Key Design Decisions Made:

1. **No conflict between deep_research_package and Ref MCP** - Use both strategically (overview → details)
2. **Preserve ALL HumanLayer workflow patterns** - Don't change what works
3. **Progressive enhancement pattern** - MCP optional, graceful degradation required
4. **Agent specialization maintained** - Locators find WHERE, Analyzers explain HOW
5. **Priority 1 is /research_codebase** - Perfect this as test case before expanding

### Context About User's Goals:

From initial discussion, user wants to:
- Adopt HumanLayer's slash commands and subagents
- Fully leverage Kit MCP and Ref MCP capabilities
- Create fusion between HumanLayer workflow and MCP servers
- Not break HumanLayer's proven development cycle power
- Eventually apply to SaaS app development (may not need all repo opens)

### Ready to Implement:

All research complete. Next agent has everything needed to:
1. Read strategy document for complete plan
2. Create MCP-enhanced agent definitions following exact patterns
3. Test graceful degradation
4. Enhance `/research_codebase` command as test case
5. Validate against success criteria

The research phase answered all key questions. Implementation can proceed with confidence.
