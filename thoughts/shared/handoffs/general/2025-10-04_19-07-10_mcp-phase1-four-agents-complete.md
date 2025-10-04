---
date: 2025-10-04T19:07:10+01:00
researcher: Selman AYAN
git_commit: 620f63b013f5bbbd95835981fa275644c1a39a7d
branch: main
repository: hlyr-reveng
topic: "MCP Phase 1: Four Agents Enhanced - Two New Agents Remaining"
tags: [mcp, kit-mcp, ref-mcp, agent-enhancement, phase-1, implementation]
status: in_progress
last_updated: 2025-10-04
last_updated_by: Selman AYAN
type: implementation_strategy
---

# Handoff: MCP Phase 1 - Four Agents Enhanced, Two New Agents + CLAUDE.md Remaining

## Task(s)

**✅ COMPLETED (Phase 1 - 4 of 6 agent enhancements):**
1. Enhanced `codebase-locator` with Kit MCP tools (4 tools: grep_ast, get_file_tree, extract_symbols, open_repository)
2. Enhanced `codebase-analyzer` with Kit MCP tools (5 tools: grep_ast, find_symbol_usages, extract_symbols, get_dependency_graph, open_repository)
3. Enhanced `codebase-pattern-finder` with Kit MCP tools (3 tools: extract_symbols, find_symbol_usages, open_repository)
   - **Critical learning:** grep_ast removed after testing - not suitable for pattern finding
4. Renamed and enhanced `web-search-researcher` → `external-doc-researcher` with Ref MCP (2 tools: ref_search_documentation, ref_read_url)
5. Tested all 4 agents successfully on real codebases (HumanLayer Go/TypeScript for Kit MCP, Stripe docs for Ref MCP)
6. All enhancements committed in git commit `620f63b`

**⏭️ REMAINING (Phase 1 - 2 new agents + documentation):**
1. Create `mcp-package-researcher` agent (NEW - combines Ref MCP + Kit MCP package search)
2. Create `codebase-dependency-tracer` agent (NEW - uses get_dependency_graph)
3. Update `CLAUDE.md` with MCP integration guidelines (tool selection, graceful degradation)
4. Validate Phase 1 success criteria (all agents work with/without MCP)

**📊 PHASE 1 PROGRESS:** 4 of 6 agent enhancements complete (67%)

## Critical References

**Primary Integration Strategy** (READ THIS FIRST):
- `thoughts/shared/handoffs/general/2025-10-04_17-06-19_mcp-integration-strategy.md` - Complete 12-part blueprint
  - Lines 736-822: mcp-package-researcher specification (NEW AGENT - next task)
  - Lines 824-913: codebase-dependency-tracer specification (NEW AGENT - next task)
  - Lines 1082-1158: CLAUDE.md enhancement specification (final task)

**Previous Handoffs** (for context):
- `thoughts/shared/handoffs/general/2025-10-04_18-48-05_mcp-phase1-agents-enhanced.md` - Previous handoff (3 agents complete)
- `thoughts/shared/handoffs/general/2025-10-04_17-59-42_codebase-locator-mcp-enhancement.md` - First agent enhancement

**Testing Evidence:**
- Agents tested on `/home/selman/dev/humanlayer` repository (Go + TypeScript codebase)

## Recent Changes

**Files Modified:**
1. `.claude/agents/codebase-locator.md` (committed in 620f63b)
   - Lines 4: Added 4 Kit MCP tools to frontmatter
   - Lines 10-16: Added Initial Setup section
   - Lines 72-101: Added AST-Aware Search Strategy
   - Lines 110-112: Added Search Method Used reporting requirement
   - Lines 138-145: Added Symbol-Level Locations output example

2. `.claude/agents/codebase-analyzer.md` (committed in 620f63b)
   - Lines 4: Added 5 Kit MCP tools to frontmatter
   - Lines 11-16: Added Initial Setup section
   - Lines 50-65: Updated Analysis Strategy to use extract_symbols, get_dependency_graph, find_symbol_usages
   - Lines 82-84: Added Search Method Used reporting requirement
   - Lines 131-142: Added Dependency Analysis output section

3. `.claude/agents/codebase-pattern-finder.md` (committed in 620f63b)
   - Lines 4: Added 3 Kit MCP tools to frontmatter (grep_ast initially added, then REMOVED)
   - Lines 11-16: Added Initial Setup section
   - Lines 66-79: Added guidance on when to use extract_symbols and find_symbol_usages
   - Lines 96-98: Added Search Method Used reporting requirement
   - Lines 200-230: Made API Comparison section OPTIONAL with clear criteria

4. `.claude/agents/external-doc-researcher.md` (renamed from `web-search-researcher.md`, committed in 620f63b)
   - Lines 2: Renamed agent name
   - Lines 3: Updated description to reflect Ref MCP specialization
   - Lines 4: Added 2 Ref MCP tools to frontmatter
   - Lines 11-24: Added Initial Setup section with Ref MCP benefits
   - Lines 37-50: Updated Execute Strategic Searches to prioritize Ref MCP
   - Lines 67-137: Added 5 comprehensive search strategies (API docs, best practices, news, general, hybrid)
   - Lines 146-182: Updated output format with source attribution and Search Method Used

**File Renamed:**
- `web-search-researcher.md` → `external-doc-researcher.md`

## Learnings

### Kit MCP Tool Effectiveness

**1. grep_ast - NOT suitable for pattern finding**
- **Tested:** All 3 modes (simple, pattern, query) on Go and TypeScript code
- **Result:** Returns AST container nodes (package, import, module) instead of actual code patterns
- **Evidence:** Searched for error handling patterns, returned package/import statements
- **Verdict:** Traditional Grep with regex FAR more effective for pattern finding
- **Action Taken:** Removed from codebase-pattern-finder agent after testing
- **Kept in:** codebase-locator (for finding symbol definitions) and codebase-analyzer (for finding specific structures)

**2. extract_symbols - Highly effective and CACHED**
- Returns structured symbol inventory (name, type, start_line, end_line, code snippet)
- **CACHED** - subsequent calls are nearly instant
- Perfect for API comparison without reading entire files
- Example: Extracted 6 symbols from `hld/store/errors.go` with exact line ranges
- **Use cases:** Function signature comparison, getting module overview, symbol inventory

**3. find_symbol_usages - Use with extreme caution**
- Returns ALL symbols matching name across:
  - ✅ Target codebase files
  - ❌ JavaScript dependencies (node_modules)
  - ❌ Python dependencies (test_env, venv)
  - ❌ Unrelated codebases in same repo
- **Best for:** Unique, project-specific symbol names (e.g., "WebhookProcessor", "AuthHandler")
- **Avoid for:** Common terms (e.g., "Error", "Handler", "Config") - returns 100+ irrelevant results
- **Recommendation:** Review results carefully and filter out dependency noise

**4. get_file_tree - Can return massive output**
- HumanLayer repo returned 4.3M tokens (exceeded 25k limit)
- Use on smaller repos or specific subdirectories
- Git-aware (automatically respects .gitignore)
- Fast alternative to recursive LS calls

**5. Repository initialization required**
- Must call `mcp__kit-dev__open_repository` first (returns repo_id like `repo_0`)
- All subsequent Kit MCP calls require this repo_id
- Repository IDs reset between Claude Code sessions

### Ref MCP Tool Effectiveness

**1. ref_search_documentation - Token-efficient and session-aware**
- Tested on Stripe webhook documentation research
- Used 4 searches + 5 reads to get comprehensive Stripe webhook docs
- **Session-aware:** No duplicate results across searches
- **Smart chunking:** Returns most relevant ~5k tokens (not entire page)
- **Query style:** Full sentences work best ("Stripe webhook signature verification Python" > "stripe webhooks")

**2. ref_read_url - Curated, official sources only**
- Returns deep links to specific documentation sections
- All code tabs loaded (not just curl examples)
- Official documentation prioritized over random blog posts
- Test results showed 100% official Stripe docs (no Medium/Stack Overflow noise)

### Agent Intelligence Validated

**Critical discovery from codebase-pattern-finder testing:**
- Agent attempted grep_ast as instructed by agent definition
- Recognized grep_ast returned "AST tokens rather than code patterns"
- Autonomously switched to Grep as more effective tool
- **Verdict:** Agent prioritized RESULTS over COMPLIANCE with instructions
- **Lesson:** Agents are intelligent enough to choose best tool for task, even if definition suggests otherwise

### Testing Methodology Established

**Effective testing pattern:**
1. Add "Search Method Used" reporting requirement to output format BEFORE testing
2. Spawn enhanced agent on real codebase (HumanLayer Go/TS, Stripe docs)
3. Verify tool reporting in agent output
4. Check for issues (errors, unexpected behavior, performance problems)
5. Get user approval before proceeding to next agent

**Tool reporting is critical:**
- Without it, cannot verify which tools subagent actually used
- Enables validation that agents prioritize MCP tools correctly
- Shows graceful degradation when MCP tools not applicable

## Artifacts

### Modified Agent Definitions (all committed in 620f63b)
- `.claude/agents/codebase-locator.md:1-174` - Enhanced with Kit MCP (+46 lines, 38% increase)
- `.claude/agents/codebase-analyzer.md:1-174` - Enhanced with Kit MCP (+30 lines, 21% increase)
- `.claude/agents/codebase-pattern-finder.md:1-294` - Enhanced with Kit MCP, grep_ast removed (+66 lines, 29% increase)
- `.claude/agents/external-doc-researcher.md:1-202` - Renamed from web-search-researcher, enhanced with Ref MCP (+104 lines, 52% increase)

### Reference Documents Read
- `thoughts/shared/handoffs/general/2025-10-04_17-06-19_mcp-integration-strategy.md:1-1578` - Complete integration strategy
- `thoughts/shared/handoffs/general/2025-10-04_18-48-05_mcp-phase1-agents-enhanced.md:1-386` - Previous handoff
- `thoughts/shared/handoffs/general/2025-10-04_17-59-42_codebase-locator-mcp-enhancement.md:1-386` - First agent handoff

### Agents NOT Modified (per strategy)
- `.claude/agents/thoughts-locator.md` - No changes needed (local filesystem, Grep/Glob/LS optimal)
- `.claude/agents/thoughts-analyzer.md` - No changes needed (markdown analysis, Read tool sufficient)

### Test Evidence
- Opened HumanLayer repository as `repo_0` using mcp__kit-dev__open_repository
- Tested extract_symbols on `hld/store/errors.go` (6 symbols returned)
- Tested grep_ast on Go error handling patterns (returned package nodes, not patterns)
- Tested external-doc-researcher on Stripe webhook docs (4 searches, 5 reads, all official docs)
- All agents reported tools used via "Search Method Used" section

## Action Items & Next Steps

### Immediate Next Steps (Phase 1 Completion)

**1. Create `mcp-package-researcher` Agent** (Strategy lines 736-822)

   **Purpose:** Deep package research combining Ref MCP (documentation) + Kit MCP package search (source code)

   **When to use:** Planning package integration, debugging third-party libraries, evaluating alternatives

   **Tools to add:**
   - `mcp__Ref__ref_search_documentation`
   - `mcp__Ref__ref_read_url`
   - `mcp__kit-dev__deep_research_package`
   - `mcp__kit-dev__package_search_grep`
   - `mcp__kit-dev__package_search_hybrid`
   - `mcp__kit-dev__package_search_read_file`
   - `WebSearch` (fallback)
   - `WebFetch` (fallback)
   - `TodoWrite`

   **Key sections to include:**
   - Initial Setup (check Ref MCP + Kit MCP availability)
   - Research Strategy (multi-source documentation gathering, source code analysis, integration best practices)
   - Output Format (Package Overview, API Reference, Integration Guide, Common Pitfalls, Example Usage)
   - "Search Method Used" reporting requirement
   - **Critical constraint:** "DO NOT recommend which package to use (that's planning phase)"

   **Testing:** Research a real package (e.g., FastAPI, Stripe) to verify Ref MCP + Kit MCP package tools work together

**2. Create `codebase-dependency-tracer` Agent** (Strategy lines 824-913)

   **Purpose:** Map dependency relationships using Kit MCP's get_dependency_graph

   **When to use:** Understanding module boundaries, refactoring planning, circular dependency detection

   **Tools to add:**
   - `mcp__kit-dev__get_dependency_graph`
   - `mcp__kit-dev__extract_symbols`
   - `mcp__kit-dev__grep_ast`
   - `mcp__kit-dev__open_repository`
   - `Read`, `Grep`, `Glob`, `LS`

   **Key sections to include:**
   - Initial Setup (check Kit MCP availability)
   - Analysis Strategy (get dependency graph, extract symbol boundaries, trace chains, identify clusters)
   - Output Format (Module Overview, Direct Dependencies, Dependency Clusters, Transitive Dependencies, Dependency Patterns)
   - "Search Method Used" reporting requirement
   - **CRITICAL CONSTRAINT:** "DO NOT identify circular dependencies as 'problems'" - documentarian role must be preserved
   - **CRITICAL CONSTRAINT:** "DO NOT recommend breaking up 'god modules'" - just document structure as it exists

   **Testing:** Analyze module dependencies in HumanLayer codebase to verify get_dependency_graph works

**3. Update `CLAUDE.md`** (Strategy lines 1082-1158)

   **Sections to add:**

   a. **Tool Selection Decision Tree:**
   ```markdown
   ## MCP Tool Selection

   ### For Local Codebase Work
   **Use Kit MCP when:**
   - Finding symbol definitions → grep_ast, extract_symbols
   - Understanding code structure → get_file_tree, extract_symbols
   - Tracing symbol usage → find_symbol_usages
   - Analyzing dependencies → get_dependency_graph

   **Use built-in tools when:**
   - Reading specific files → Read (faster, complete)
   - Searching non-code files → Grep
   - File path patterns → Glob

   ### For External Documentation
   **Use Ref MCP when:**
   - Researching APIs → ref_search_documentation
   - Reading documentation → ref_read_url

   **Use WebSearch when:**
   - Current events or news
   - Real-time data
   - General information
   ```

   b. **Repository Initialization Pattern:**
   ```markdown
   ## Repository Initialization

   At the start of codebase work:
   1. open_repository to load context
   2. get_file_tree to understand structure
   3. extract_symbols for fast overview
   4. Then proceed with focused analysis
   ```

   c. **Agent Usage Guidance:**
   - List all 6 enhanced agents with when to use each
   - Include the 2 new agents (mcp-package-researcher, codebase-dependency-tracer)

   d. **Graceful Degradation:**
   ```markdown
   ## Graceful Degradation

   All workflows work with or without MCP:
   - If MCP available: Enhanced AST awareness, token efficiency
   - If MCP unavailable: Traditional Grep/Read/Glob tools
   - No workflow changes based on MCP availability
   ```

**4. Validate Phase 1 Success Criteria** (Strategy lines 1176-1182)
   - [ ] All 6 agents work with MCP available (4 of 6 complete, 2 new agents pending)
   - [ ] All 6 agents gracefully degrade without MCP (4 of 6 verified, 2 pending)
   - [ ] Agent outputs same structure as before (enhanced content) (4 of 6 verified, 2 pending)
   - [ ] Tool reporting enables verification of MCP usage (✅ all 4 agents report tools used)
   - [ ] All documentarian constraints preserved (✅ all 4 agents maintain "DO NOT critique" constraints)

### Testing Requirements for New Agents

**For BOTH new agents, use this testing protocol:**
1. Add "Search Method Used" reporting to output format BEFORE testing
2. Spawn the new agent on real code/packages
3. Verify tool reporting in agent output
4. Check for issues (errors, unexpected behavior)
5. Document any learnings about new tools (deep_research_package, package_search_*, get_dependency_graph)

### Phase 1 Estimated Completion Time

- Create mcp-package-researcher: ~30 minutes
- Create codebase-dependency-tracer: ~30 minutes
- Update CLAUDE.md: ~20 minutes
- Testing and validation: ~20 minutes

**Total:** ~2 hours to complete Phase 1

## Other Notes

### Implementation Pattern Established

**Repeatable pattern for enhancing agents (used for all 4 agents):**
1. Read existing agent file completely
2. Add Kit MCP/Ref MCP tools to frontmatter (additive, don't remove existing tools)
3. Add "Initial Setup" section if MCP tools added
4. Add MCP-specific strategy sections (AST-aware search, Ref MCP prioritization, etc.)
5. Add "Search Method Used" reporting to output format
6. Add enhanced output examples (symbol-level, dependency analysis, source attribution)
7. **PRESERVE ALL:** Negative constraints, documentarian role, core responsibilities
8. Test with tool reporting verification
9. Get user approval before next agent

**All "DO NOT" constraints must be preserved in new agents:**
- DO NOT suggest improvements
- DO NOT perform root cause analysis (unless explicitly requested)
- DO NOT propose enhancements
- DO NOT critique implementation
- DO NOT comment on quality/architecture
- ONLY describe what exists

**Documentarian role is sacred:** Every agent must maintain "You are a documentarian, not a critic or consultant"

### Repository Context

**Repos Available** (may need to re-open after session restart):
- `/home/selman/dev/humanlayer` - HumanLayer main repository (Go + TypeScript)
  - Use for testing Go code (`hld/daemon/*.go`, `hld/store/*.go`)
  - Use for testing TypeScript code (`hlyr/`, `humanlayer-ts/`)
  - **Warning:** `get_file_tree` returns 4.3M tokens (too large, use subdirectories)

**This Repository:**
- `/home/selman/dev/hlyr-reveng` - Research/strategy repository
  - Contains integration strategy documents
  - Contains agent definitions (`.claude/agents/`)
  - Contains handoff documents (`thoughts/shared/handoffs/`)
  - Mostly markdown files (Kit MCP won't help, Grep/Glob work fine)

### Key Files to Reference

**Agent Files (6 total):**
- `.claude/agents/codebase-locator.md` - ✅ ENHANCED (committed)
- `.claude/agents/codebase-analyzer.md` - ✅ ENHANCED (committed)
- `.claude/agents/codebase-pattern-finder.md` - ✅ ENHANCED (committed)
- `.claude/agents/external-doc-researcher.md` - ✅ RENAMED + ENHANCED (committed)
- `.claude/agents/thoughts-locator.md` - ⏭️ SKIP (no changes needed)
- `.claude/agents/thoughts-analyzer.md` - ⏭️ SKIP (no changes needed)

**New Agent Files to Create (2 total):**
- `.claude/agents/mcp-package-researcher.md` - 🆕 CREATE (next task)
- `.claude/agents/codebase-dependency-tracer.md` - 🆕 CREATE (next task)

**Documentation to Update:**
- `CLAUDE.md` - 📝 UPDATE (final Phase 1 task)

**Strategy Reference:**
- `thoughts/shared/handoffs/general/2025-10-04_17-06-19_mcp-integration-strategy.md` - Complete 12-part blueprint (READ THIS!)

### Important Constraints to Remember

**From Integration Strategy (lines 227-254):**
1. **Progressive Enhancement** - MCP enhances, doesn't replace. Graceful degradation required.
2. **Preserve Workflow Architecture** - Don't change agent core purpose or output structure.
3. **Additive Tool Access** - Add MCP tools, don't remove existing Grep/Glob/Read/LS.
4. **Strategic MCP Usage** - Kit for code, Ref for docs, built-ins for simple operations.
5. **No Behavioral Change** - Agents still documentarians, no critique/suggestions/improvements.

### Next Session Startup Commands

```bash
# Resume from this handoff
/resume_handoff thoughts/shared/handoffs/general/2025-10-04_19-07-10_mcp-phase1-four-agents-complete.md

# Re-open HumanLayer repo for testing (if needed)
# Will be done automatically by agents via mcp__kit-dev__open_repository
```

### Progress Tracking

**Phase 1 Progress:** 4 of 6 agent enhancements + 0 of 2 new agents + 0 of 1 documentation = 57% complete

**Checklist:**
- [x] codebase-locator enhanced + tested + committed
- [x] codebase-analyzer enhanced + tested + committed
- [x] codebase-pattern-finder enhanced + tested + committed
- [x] external-doc-researcher renamed + enhanced + tested + committed
- [ ] mcp-package-researcher created + tested
- [ ] codebase-dependency-tracer created + tested
- [ ] CLAUDE.md updated
- [ ] Phase 1 validation (all agents work together)

**Estimated remaining time:** ~2 hours at established pace

**Git status:** All changes committed in `620f63b feat: enhance codebase agents with Kit MCP tools`
