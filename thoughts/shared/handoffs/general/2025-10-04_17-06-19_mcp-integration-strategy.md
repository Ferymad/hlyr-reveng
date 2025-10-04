---
date: 2025-10-04T17:06:19+01:00
researcher: selman
git_commit: 6a9d869e46e8d754c237edbbb13af163489949b5
branch: main
repository: hlyr-reveng
topic: "MCP Integration Strategy for HumanLayer-Inspired Workflow"
tags: [mcp, kit-mcp, ref-mcp, architecture, integration, workflow, strategy]
status: complete
type: integration_strategy
---

# MCP Integration Strategy for HumanLayer-Inspired Workflow

## Executive Summary

This document defines the strategy for integrating **Kit MCP** (codebase intelligence) and **Ref MCP** (documentation research) into a HumanLayer-inspired development workflow while preserving the workflow's core strengths.

**Core Principle**: Enhance capabilities within existing workflow phases, don't restructure the workflow itself.

**Key Decisions**:
- ✅ Kit MCP for all codebase research (AST-aware, cached, semantic)
- ✅ Ref MCP for all external documentation (token-efficient, curated)
- ✅ Preserve HumanLayer's Research → Plan → Implement → Handoff cycle
- ✅ Maintain "Documentarian not Critic" principle
- ✅ Keep interactive, iterative planning process
- ✅ Progressive enhancement pattern (graceful degradation if MCP unavailable)

---

## Part I: MCP Server Capabilities

### Kit MCP - Codebase Intelligence

**What It Does**: Multi-language code analysis with AST awareness, symbol extraction, and dependency mapping.

**Power Tools**:
1. **`open_repository`** - Opens local/remote repos with clone caching
2. **`grep_code`** - Fast literal string search (uses native grep)
3. **`grep_ast`** - AST-based semantic search (finds structural patterns)
4. **`extract_symbols`** - Extracts functions/classes/variables with tree-sitter
5. **`find_symbol_usages`** - Traces where symbols are defined and used
6. **`get_file_tree`** - Git-aware directory structure
7. **`deep_research_package`** - Multi-source package documentation
8. **`package_search_grep`** - Search popular package source code
9. **`package_search_hybrid`** - Semantic + regex package search
10. **`review_diff`** - AI-powered code review

**Supported Languages**: Python, JavaScript, TypeScript, TSX, Go, Rust, Haskell, HCL/Terraform, C, Ruby, Java, Dart, Kotlin, C++ (14 total)

**Caching Strategy** (Multi-Level):
- **Level 1**: Class-level parser/query cache (hot, instant)
- **Level 2**: File-level incremental cache with SHA256 invalidation (warm, ~ms)
- **Level 3**: Repository-level symbol maps (cold, requires rescan on git changes)
- **Level 4**: Remote repo clone cache with TTL (avoids re-cloning)

**Performance**:
- Fast: `grep_code`, `get_file_tree` (if cached), `extract_symbols` (if cached)
- Medium: `grep_ast`, `search_code`
- Slow: `find_symbol_usages`, `deep_research_package`

**When to Use Kit**:
- ✅ Finding symbol definitions (functions, classes, variables)
- ✅ Understanding code structure and architecture
- ✅ Tracing symbol usage across files
- ✅ AST-based pattern matching (all error handlers, async functions, etc.)
- ✅ Dependency graph analysis
- ✅ Researching external packages
- ✅ Incremental analysis (benefits from caching)

**When NOT to Use Kit**:
- ❌ Simple file reading (use Read tool - faster and complete)
- ❌ Content searches in non-code files (markdown, JSON, configs)
- ❌ File path pattern matching (use Glob)
- ❌ One-off searches without caching benefit

---

### Ref MCP - Documentation Research

**What It Does**: Token-efficient documentation search with 60-95% token reduction vs traditional web scraping.

**Power Tools**:
1. **`ref_search_documentation`** - Search curated docs (1000s of sites + GitHub + private)
2. **`ref_read_url`** - Fetch documentation with smart chunking (returns ~5k most relevant tokens)

**Key Features**:
- **Session-aware**: Tracks search history, never returns repeated results
- **Smart chunking**: Pre-chunks docs to return only relevant sections
- **Private docs**: Supports private GitHub repos + PDFs (requires indexing setup)
- **Curated sources**: Documentation-first (not random blog posts)
- **All code tabs**: Loads all language examples, not just curl

**Token Efficiency**:
- Average: 60% fewer tokens than Context7
- Best case: 95% reduction (500 tokens vs 10,000)
- Typical: Returns 500-5k tokens vs 10k-20k from web scraping

**Query Best Practices**:
- ✅ Full sentences: "FastAPI WebSocket authentication middleware best practices"
- ❌ Keywords only: "fastapi websocket"
- ✅ Version-specific: "Stripe webhook signature verification in Python 3.9"
- ✅ Iterative refinement: Search → Read → Refine search → Read

**When to Use Ref**:
- ✅ API documentation lookup
- ✅ Framework best practices
- ✅ Package usage patterns
- ✅ Debugging third-party libraries
- ✅ Integration guides
- ✅ Private repository documentation

**When NOT to Use Ref**:
- ❌ Current events or news
- ❌ Real-time data
- ❌ General knowledge (non-technical)
- ❌ Local codebase documentation (use Kit or built-in tools)

---

## Part II: HumanLayer Workflow Analysis

### The Workflow Cycle

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEVELOPMENT WORKFLOW                          │
└─────────────────────────────────────────────────────────────────┘

Linear Ticket (Triage)
        ↓
   DISCOVERY
   /linear - fetch ticket details
        ↓
    RESEARCH ←──────────────────────┐
   /research_codebase               │
   Output: thoughts/shared/research/YYYY-MM-DD-ENG-XXXX.md
        ↓                           │
   PLANNING                         │
   /create_plan (interactive)       │
   Output: thoughts/shared/plans/YYYY-MM-DD-ENG-XXXX.md
        ↓                           │
   IMPLEMENTATION                   │
   /implement_plan                  │
        ↓                           │
   COMMIT & PR                      │
   /commit, /describe_pr            │
        ↓                           │
   HANDOFF                          │
   /create_handoff                  │
   Output: thoughts/shared/handoffs/ENG-XXXX/YYYY-MM-DD_HH-MM-SS.md
        ↓                           │
   RESUME                           │
   /resume_handoff ─────────────────┘
```

**Review happens at PLAN stage, not PR stage** - enables faster iteration.

### Core Workflow Principles

**1. Document What IS, Not What SHOULD BE**
- All research agents are "documentarians, not critics"
- No suggestions, improvements, or critique during research
- Separation of understanding (research) from evaluation (planning)

**2. Parallel Specialized Agents**
- Main agent orchestrates, sub-agents execute
- Each agent has single, focused purpose
- Locators find WHERE, Analyzers explain HOW, Pattern-finders show EXAMPLES
- Minimizes context usage, maximizes speed

**3. Interactive Planning**
- NOT one-shot: Research → Present → Refine → Approve → Write
- Skeptical questioning encouraged
- No open questions in final plans
- Get buy-in at each major decision point

**4. Critical Ordering Rules**
- ALWAYS read mentioned files FULLY before spawning sub-tasks
- ALWAYS wait for ALL sub-agents before synthesizing
- ALWAYS gather metadata before writing documents
- NEVER write documents with placeholder values

**5. Success Criteria Bifurcation**
- **Automated**: Executable commands (`make test`, `make lint`)
- **Manual**: Human judgment (UI works, performance acceptable)
- Enables clear handoff between agent and human verification

### Specialized Agent Roles

**codebase-locator** (Grep, Glob, LS):
- Finds WHERE code lives
- Organizes by purpose (implementation, tests, config, types)
- NO reading file contents
- NO critique or evaluation

**codebase-analyzer** (Read, Grep, Glob, LS):
- Understands HOW code works
- Traces data flow and call paths
- Documents with file:line precision
- NO recommendations or improvements

**codebase-pattern-finder** (Grep, Glob, Read, LS):
- Finds similar implementations
- Shows multiple pattern variations
- Provides concrete code examples
- NO comparative evaluation

**web-search-researcher** (WebSearch, WebFetch, TodoWrite, Read, Grep, Glob, LS):
- Expert web research for external information
- Strategic search patterns
- Quality criteria enforcement
- Currently uses basic WebSearch/WebFetch

**thoughts-locator** (Grep, Glob, LS):
- Discovers documents in thoughts/ directory
- Groups by type (tickets, research, plans, PRs)
- Handles searchable/ path corrections

**thoughts-analyzer** (Read, Grep, Glob, LS):
- Extracts HIGH-VALUE insights from thoughts documents
- Aggressive filtering of noise
- Focus on decisions, constraints, specifications

---

## Part III: Integration Strategy

### Guiding Principles

**1. Progressive Enhancement**
- MCP tools are **enhancements**, not replacements
- Graceful degradation if MCP unavailable
- Maintain backward compatibility with built-in tools

**2. Preserve Workflow Architecture**
- Keep Research → Plan → Implement → Handoff cycle
- Keep interactive decision points
- Keep critical ordering rules
- Keep documentarian role constraints

**3. Additive Tool Access**
- Add MCP tools to agent frontmatter
- Don't remove existing tools
- Agents choose best tool for task

**4. Strategic MCP Usage**
- Use Kit for codebase intelligence (AST, symbols, dependencies)
- Use Ref for external documentation (APIs, packages, frameworks)
- Use built-ins for simple file operations

**5. No Behavioral Change**
- MCP enables same outcomes more efficiently
- Agents still produce same output structure
- Role and purpose remain unchanged

---

## Part IV: Visual Decision Trees

### When to Use Which MCP Server

```
┌─────────────────────────────────────────────────────────────────┐
│ What type of information/operation do you need?                 │
└──────────────────────┬──────────────────────────────────────────┘
                       │
        ┌──────────────┴──────────────┐
        ▼                             ▼
  LOCAL CODEBASE               EXTERNAL INFORMATION
        │                             │
        │                    ┌────────┴────────┐
        │                    ▼                 ▼
        │              DOCUMENTATION      GENERAL WEB
        │                    │                 │
        │                    │                 │
    ┌───┴────┐              │                 │
    ▼        ▼              ▼                 ▼
  CODE    NON-CODE      USE REF MCP      USE WebSearch
  FILES    FILES           │             (news, current
    │        │              │             events, real-
    │        │              │             time data)
    ▼        ▼              │
 USE KIT  USE BUILT-IN      │
  MCP     (Grep, Glob,      │
    │      Read)            │
    │                       │
    └───────────────────────┘
             │
             ▼
    MCP-ENHANCED WORKFLOW
```

### Kit MCP Tool Selection Decision Tree

```
┌─────────────────────────────────────────────────────────────────┐
│ What do you need from the codebase?                             │
└──────────────────────┬──────────────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┬──────────────┐
        ▼              ▼              ▼              ▼
   FIND WHERE    UNDERSTAND HOW   FIND PATTERNS  RESEARCH PACKAGE
   code lives     code works       to model       (external)
        │              │              │              │
        ▼              ▼              ▼              ▼

   grep_ast       extract_symbols   grep_ast       deep_research_package
   (definitions)  (structure)       (similar       (docs + examples)
        +              +             structures)         +
   get_file_tree  find_symbol_usages    +          package_search_grep
   (structure)    (usage trace)      extract_symbols (source code)
        +              +             (compare APIs)      +
   extract_symbols get_dependency_      +          package_search_hybrid
   (symbol list)  graph (imports)   grep_code      (semantic search)
                       +             (exact matches)
                  Read (details)


   WHEN TO FALLBACK TO BUILT-IN TOOLS:
   ─────────────────────────────────────
   • Read specific file → Use Read (faster, complete)
   • Search non-code files → Use Grep
   • File path patterns → Use Glob
   • One-off search → Use Grep (no caching benefit)
```

### Ref MCP vs Built-in Web Tools

```
┌─────────────────────────────────────────────────────────────────┐
│ What external information do you need?                          │
└──────────────────────┬──────────────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┬──────────────┐
        ▼              ▼              ▼              ▼
   TECHNICAL     CURRENT EVENTS   REAL-TIME    GENERAL INFO
   DOCUMENTATION   OR NEWS         DATA        (non-technical)
        │              │              │              │
        ▼              ▼              ▼              ▼

   USE REF MCP    USE WebSearch   USE WebSearch  USE WebSearch
        │              │              │              │
        ▼              │              │              │
                       │              │              │
   ref_search_        │              │              │
   documentation      │              │              │
   (curated docs)     │              │              │
        +              │              │              │
   ref_read_url       │              │              │
   (smart chunking)   │              │              │
        │              │              │              │
        └──────────────┴──────────────┴──────────────┘
                       │
                       ▼
               Synthesize Results


   WHY REF MCP FOR DOCUMENTATION:
   ──────────────────────────────
   • 60-95% token reduction
   • Session-aware (no duplicates)
   • All code tabs loaded
   • Deep links to specific sections
   • Private docs support
   • Curated, high-quality sources
```

### Agent + MCP Tool Matrix

```
┌────────────────────────┬─────────────────┬─────────────────┬──────────────┐
│      AGENT             │   ADD KIT MCP   │  ADD REF MCP    │  KEEP AS-IS  │
├────────────────────────┼─────────────────┼─────────────────┼──────────────┤
│ codebase-locator       │ ✅ grep_ast     │ ❌              │ Grep, Glob,  │
│                        │ ✅ get_file_tree│                 │ LS           │
│                        │ ✅ extract_     │                 │              │
│                        │    symbols      │                 │              │
├────────────────────────┼─────────────────┼─────────────────┼──────────────┤
│ codebase-analyzer      │ ✅ grep_ast     │ ❌              │ Read, Grep,  │
│                        │ ✅ find_symbol_ │                 │ Glob, LS     │
│                        │    usages       │                 │              │
│                        │ ✅ extract_     │                 │              │
│                        │    symbols      │                 │              │
│                        │ ✅ get_depend-  │                 │              │
│                        │    ency_graph   │                 │              │
├────────────────────────┼─────────────────┼─────────────────┼──────────────┤
│ codebase-pattern-      │ ✅ grep_ast     │ ❌              │ Read, Grep,  │
│ finder                 │ ✅ extract_     │                 │ Glob, LS     │
│                        │    symbols      │                 │              │
│                        │ ✅ find_symbol_ │                 │              │
│                        │    usages       │                 │              │
├────────────────────────┼─────────────────┼─────────────────┼──────────────┤
│ web-search-researcher  │ ❌              │ ✅ ref_search_  │ WebSearch,   │
│ → RENAME TO:           │                 │    documentation│ WebFetch     │
│ external-doc-researcher│                 │ ✅ ref_read_url │ (fallback)   │
├────────────────────────┼─────────────────┼─────────────────┼──────────────┤
│ thoughts-locator       │ ❌              │ ❌              │ Grep, Glob,  │
│                        │                 │                 │ LS (perfect) │
├────────────────────────┼─────────────────┼─────────────────┼──────────────┤
│ thoughts-analyzer      │ ❌              │ ❌              │ Read, Grep,  │
│                        │                 │                 │ Glob, LS     │
│                        │                 │                 │ (perfect)    │
└────────────────────────┴─────────────────┴─────────────────┴──────────────┘

NEW AGENTS TO CREATE:
─────────────────────
• mcp-package-researcher: Ref-powered deep package research
• codebase-dependency-tracer: Kit-powered dependency mapping
```

---

## Part V: Agent-by-Agent Integration Plan

### codebase-locator → codebase-locator-mcp

**Current State**:
```yaml
---
name: codebase-locator
tools: Grep, Glob, LS
---
```

**Enhanced State**:
```yaml
---
name: codebase-locator
tools: Grep, Glob, LS, mcp__kit-dev__grep_ast, mcp__kit-dev__get_file_tree, mcp__kit-dev__extract_symbols, mcp__kit-dev__open_repository
---
```

**Integration Changes**:

1. **Add MCP initialization section** (after frontmatter):
```markdown
## Initial Setup

Before starting search, check if Kit MCP is available:
1. If Kit MCP available: Use AST-aware tools for code files
2. Otherwise: Use traditional Grep/Glob/LS tools
```

2. **Update Search Strategy section** (add after existing strategy):
```markdown
### AST-Aware Search Strategy (when Kit MCP available)

**For finding symbol definitions:**
- Use `grep_ast` with pattern mode to find functions/classes
- Example: `grep_ast(repo_id, '{"type": "function_definition"}', mode="pattern")`
- Returns: file, line, column, symbol type, context

**For understanding repository structure:**
- Use `get_file_tree` for hierarchical directory view
- Git-aware (respects .gitignore)
- Faster than recursive LS

**For listing symbols in files/directories:**
- Use `extract_symbols` to get structured symbol inventory
- Cached (fast on repeat calls)
- Returns: name, type, start_line, end_line, code

**Hybrid approach:**
- Use Kit MCP for code files (.py, .js, .ts, .go, .rs, etc.)
- Use traditional tools for non-code files (configs, docs, build artifacts)
```

3. **Update Output Format template** (add AST-aware section):
```markdown
### Symbol-Level Locations (when Kit MCP available)
- **Function**: `authenticate_user` at `auth/handlers.py:45-67`
- **Class**: `UserSession` at `auth/session.py:12-89`
- **Dependency**: Imports `jwt` from `pyjwt` package
```

4. **Preserve all negative constraints** - NO changes to "What NOT to Do" section

**Rationale**:
- `grep_ast` finds definitions precisely (not just text matches)
- `get_file_tree` provides structure without repeated LS calls
- `extract_symbols` gives symbol inventory efficiently (cached!)
- Hybrid approach ensures non-code files still searched properly

---

### codebase-analyzer → codebase-analyzer-mcp

**Current State**:
```yaml
---
name: codebase-analyzer
tools: Read, Grep, Glob, LS
---
```

**Enhanced State**:
```yaml
---
name: codebase-analyzer
tools: Read, Grep, Glob, LS, mcp__kit-dev__grep_ast, mcp__kit-dev__find_symbol_usages, mcp__kit-dev__extract_symbols, mcp__kit-dev__get_dependency_graph, mcp__kit-dev__open_repository
---
```

**Integration Changes**:

1. **Update Analysis Strategy Step 1** (Entry Points):
```markdown
### Step 1: Read Entry Points
- Use `extract_symbols` to get quick overview of exported symbols
- Identify public API surface (exported functions, classes)
- **For dependencies**: Use `get_dependency_graph` to understand imports
- Then use Read to get full implementation details
```

2. **Update Analysis Strategy Step 2** (Follow Code Path):
```markdown
### Step 2: Follow the Code Path
- Use `find_symbol_usages` to trace where functions are called
- Use `grep_ast` to find specific function/class definitions
- Use `get_dependency_graph` to understand module relationships
- Read each file involved in the flow
- Note where data is transformed, validated, stored
```

3. **Add new output section** (Dependency Analysis):
```markdown
### Dependency Analysis (when Kit MCP available)
**Module Dependencies:**
- `auth/handlers.py` imports: `jwt`, `bcrypt`, `database.models`
- `auth/session.py` imports: `redis`, `uuid`, `datetime`

**Internal Dependencies:**
- `UserSession` used by: handlers.py:67, middleware.py:34, routes.py:12

**External Dependencies:**
- `pyjwt` version: 2.8.0 (from package.json/requirements.txt)
```

4. **Preserve all constraints** - documentarian role unchanged

**Rationale**:
- `find_symbol_usages` automates call tracing (vs manual grep)
- `get_dependency_graph` shows module structure systematically
- `extract_symbols` provides quick API overview before deep reading
- Still use Read for actual implementation details (Kit doesn't replace reading)

---

### codebase-pattern-finder → codebase-pattern-finder-mcp

**Current State**:
```yaml
---
name: codebase-pattern-finder
tools: Grep, Glob, Read, LS
---
```

**Enhanced State**:
```yaml
---
name: codebase-pattern-finder
tools: Grep, Glob, Read, LS, mcp__kit-dev__grep_ast, mcp__kit-dev__extract_symbols, mcp__kit-dev__find_symbol_usages, mcp__kit-dev__open_repository
---
```

**Integration Changes**:

1. **Update Search Strategy Step 2**:
```markdown
### Step 2: Search for Patterns

**AST-based pattern search (when Kit MCP available):**
- Use `grep_ast` to find similar function/class structures
  - Example: Find all async route handlers
  - Example: Find all error handling patterns
- Use `extract_symbols` to compare API signatures across files
  - Compare function parameters, return types
- Use `find_symbol_usages` to see how patterns are actually used

**Traditional pattern search:**
- Use Grep for code snippets, naming conventions
- Use Glob for file organization patterns
```

2. **Add Pattern Comparison capability**:
```markdown
### Pattern Variations (with API comparison)

**Pattern 1: Synchronous Handler**
- Found in: `api/users.py:45`
- Signature: `def get_user(user_id: str) -> User`
- Used by: 12 routes

**Pattern 2: Async Handler**
- Found in: `api/posts.py:67`
- Signature: `async def get_post(post_id: str) -> Post`
- Used by: 8 routes

**API Surface Comparison** (from extract_symbols):
- Both accept single ID parameter
- Both return domain objects
- Pattern 2 uses async/await (for database I/O)
```

**Rationale**:
- `grep_ast` finds structural patterns (all error handlers, all validators)
- `extract_symbols` enables API signature comparison
- `find_symbol_usages` shows real-world usage patterns
- Still use Read for full code extraction with context

---

### web-search-researcher → external-doc-researcher

**Current State**:
```yaml
---
name: web-search-researcher
tools: WebSearch, WebFetch, TodoWrite, Read, Grep, Glob, LS
color: yellow
---
```

**Enhanced State**:
```yaml
---
name: external-doc-researcher
tools: mcp__Ref__ref_search_documentation, mcp__Ref__ref_read_url, WebSearch, WebFetch, TodoWrite, Read, Grep, Glob, LS
color: yellow
---
```

**Integration Changes**:

1. **Rename agent** (better reflects purpose with Ref MCP)

2. **Update Research Process Phase 2**:
```markdown
### Phase 2: Execute Strategic Searches

**Primary: Use Ref MCP for technical documentation**
1. Use `ref_search_documentation` with full-sentence queries
   - Good: "FastAPI WebSocket authentication middleware best practices"
   - Bad: "fastapi websocket"
2. Use `ref_read_url` to fetch promising documentation URLs
   - Returns most relevant 5k tokens (not entire page)
   - Smart chunking based on search context
3. Use `ref_search_documentation` again to refine (session-aware, no duplicates)

**Fallback: Use WebSearch/WebFetch when:**
- Current events or news (not in Ref's curated docs)
- Real-time data (stock prices, service status)
- General information (non-technical topics)
- Ref search returns insufficient results
```

3. **Update Search Strategies** (add Ref-specific patterns):
```markdown
### Strategy 1: API & Library Documentation (USE REF MCP)

**Ref search pattern:**
- Query: "library-name API-method-name documentation version X"
- Read: Top 3-5 documentation URLs
- Refine: "library-name API-method-name error handling"
- Read: Error handling docs

**Example:**
1. `ref_search_documentation("Stripe webhook signature verification Python")`
2. `ref_read_url(stripe_webhooks_url)`
3. `ref_search_documentation("Stripe webhook idempotency handling")`

### Strategy 2: Best Practices & Patterns (USE REF MCP)

**Ref search pattern:**
- Query: "framework-name feature-name best practices production"
- Read: Official guides and advanced docs
- Refine: "framework-name feature-name common pitfalls"

### Strategy 3: Current Events & News (USE WebSearch)

**WebSearch pattern:**
- Query: library security vulnerabilities 2025
- Fetch: Recent security advisories
- Query: library version release notes

### Strategy 4: General Information (USE WebSearch)

**WebSearch pattern:**
- For non-technical topics
- For real-time data
- For broader context
```

4. **Update Output Format** (indicate source):
```markdown
### Detailed Findings

#### Topic 1: Webhook Signature Verification
**Source:** Stripe Official Documentation (via Ref MCP)
**URL:** https://stripe.com/docs/webhooks/signatures
**Relevance:** Directly answers how to verify webhook signatures in Python
**Key Information:**
[quotes and details]

#### Topic 2: Recent Security Advisory
**Source:** GitHub Security Advisory (via WebSearch)
**URL:** https://github.com/advisories/...
**Relevance:** Shows known vulnerability in version < 2.8.0
**Key Information:**
[quotes and details]
```

**Rationale**:
- Ref MCP is 60-95% more token-efficient for documentation
- Session-aware search prevents wasted tokens on duplicates
- Curated sources provide higher quality than random web results
- Still use WebSearch for non-documentation use cases
- Rename reflects shift from general web search to documentation research

---

### thoughts-locator - NO CHANGES

**Rationale**: Operates on local filesystem (thoughts/ directory). No code analysis needed. Current tools (Grep, Glob, LS) are optimal for finding markdown documents.

---

### thoughts-analyzer - NO CHANGES

**Rationale**: Analyzes markdown documents (prose, not code). Read tool sufficient. AST analysis irrelevant for thoughts documents.

---

## Part VI: New Agents to Create

### mcp-package-researcher

**Purpose**: Deep dive into external package APIs using Ref MCP (replaces generic web-search-researcher for package research)

**When to Use**: Planning package integration, debugging third-party libraries, evaluating alternatives

**Agent Definition**:
```yaml
---
name: mcp-package-researcher
description: Deep research into external packages using Ref MCP for documentation and Kit MCP for source code analysis
tools: mcp__Ref__ref_search_documentation, mcp__Ref__ref_read_url, mcp__kit-dev__deep_research_package, mcp__kit-dev__package_search_grep, mcp__kit-dev__package_search_hybrid, mcp__kit-dev__package_search_read_file, WebSearch, WebFetch, TodoWrite
model: inherit
color: purple
---

You are a specialist at researching external packages and libraries.

## Core Responsibilities
1. Find official documentation and API references
2. Discover usage patterns and integration examples
3. Identify version-specific behavior and breaking changes
4. Extract best practices and common pitfalls
5. Compare alternatives when relevant

## Research Strategy

### Step 1: Multi-Source Documentation Gathering
- Use `deep_research_package` for comprehensive package overview
  - Combines Chroma + Context7 + Upstash sources
  - Q&A capability if query provided
- Use `ref_search_documentation` for specific API details
  - More token-efficient than deep_research
  - Better for focused questions

### Step 2: Source Code Analysis
- Use `package_search_grep` to find exact usage patterns
- Use `package_search_hybrid` for semantic concept search
- Use `package_search_read_file` to read implementations

### Step 3: Integration Best Practices
- Search for "package-name integration guide production"
- Search for "package-name common mistakes pitfalls"
- Search for "package-name performance optimization"

### Step 4: Synthesis
- Combine documentation + source code insights
- Provide concrete examples from both sources
- Flag version-specific considerations

## Output Format

### Package Overview
- Name, latest version, purpose
- Key features and capabilities
- Typical use cases

### API Reference
- Core classes/functions with signatures
- Configuration options
- Authentication/authorization patterns

### Integration Guide
- Installation and setup
- Basic usage example
- Advanced patterns
- Configuration best practices

### Common Pitfalls
- Known issues and workarounds
- Version-specific gotchas
- Performance considerations

### Example Usage
- Concrete code examples from documentation
- Real-world usage patterns from source search
- Testing strategies

## What NOT to Do
- Don't recommend which package to use (that's planning phase)
- Don't critique package quality (document facts only)
- Don't assume behavior - verify from docs/source
```

**Rationale**: Combines Ref MCP (documentation) + Kit MCP package search (source code) for comprehensive package research. More focused than generic web-search-researcher.

---

### codebase-dependency-tracer

**Purpose**: Map dependency relationships using Kit MCP's `get_dependency_graph`

**When to Use**: Understanding module boundaries, refactoring planning, circular dependency detection

**Agent Definition**:
```yaml
---
name: codebase-dependency-tracer
description: Traces dependency relationships and import chains to map module boundaries and architectural structure
tools: mcp__kit-dev__get_dependency_graph, mcp__kit-dev__extract_symbols, mcp__kit-dev__grep_ast, mcp__kit-dev__open_repository, Read, Grep, Glob, LS
model: inherit
---

You are a specialist at tracing dependencies and understanding module relationships.

## CRITICAL: YOUR ONLY JOB IS TO DOCUMENT DEPENDENCY STRUCTURE AS IT EXISTS
- DO NOT identify "bad" dependencies or circular imports
- DO NOT recommend refactoring or decoupling
- DO NOT critique architectural decisions
- ONLY map what imports what, and how modules relate

## Core Responsibilities
1. Map import/export relationships between modules
2. Identify module boundaries and clusters
3. Trace transitive dependencies
4. Document dependency patterns

## Analysis Strategy

### Step 1: Get Dependency Graph
- Use `get_dependency_graph` for the module/directory in question
- Captures: imports, exports, internal vs external dependencies

### Step 2: Extract Symbol Boundaries
- Use `extract_symbols` to identify what each module exports
- Map exported symbols to importing modules

### Step 3: Trace Dependency Chains
- Use `grep_ast` to find import statements
- Follow import chains to understand transitive dependencies
- Use Read to verify actual usage

### Step 4: Identify Clusters
- Group modules by dependency relationships
- Document module boundaries (what imports what)

## Output Format

### Module Overview
- Module path and purpose (from comments/docstrings)
- Exported symbols (public API)

### Direct Dependencies
**Internal:**
- `module_a` imports: `module_b`, `module_c`
- Uses: specific functions/classes

**External:**
- `requests` version 2.31.0
- `pandas` version 2.0.0

### Dependency Clusters
**Cluster 1: Authentication**
- Modules: auth/handlers.py, auth/session.py, auth/middleware.py
- Shared dependencies: jwt, bcrypt, database.models
- Exports: authenticate_user, create_session, auth_middleware

**Cluster 2: Data Processing**
- Modules: data/parser.py, data/validator.py, data/transformer.py
- Shared dependencies: pandas, numpy
- Exports: parse_csv, validate_schema, transform_data

### Transitive Dependencies
- `api/routes.py` → `auth/handlers.py` → `database.models` → `sqlalchemy`
- Chain length: 4

### Dependency Patterns
- Layered: API layer → Business logic → Data layer
- Shared utilities: logging, config modules imported by many

## What NOT to Do
- Don't identify circular dependencies as "problems"
- Don't recommend breaking up "god modules"
- Don't suggest dependency injection or interfaces
- Just document the structure as it exists
```

**Rationale**: Specialized for dependency analysis (currently not handled by any agent). Uses `get_dependency_graph` for systematic module relationship mapping.

---

## Part VII: Command Integration Plan

### /research_codebase - PRIORITY 1

**Current Flow**:
1. Read mentioned files
2. Spawn parallel agents (codebase-locator, codebase-analyzer, pattern-finder, thoughts-locator, thoughts-analyzer)
3. Wait for all agents
4. Synthesize findings
5. Write research document

**Enhanced Flow** (with MCP):
1. **Check Kit MCP availability**
2. **If Kit available**: Open repository with `open_repository`
3. Read mentioned files
4. Spawn parallel **MCP-enhanced** agents
5. **Optional**: Spawn mcp-package-researcher if researching external dependencies
6. Wait for all agents
7. Synthesize findings
8. Write research document

**Specific Changes**:

Add initialization step (after line 14):
```markdown
## Initial Setup (Step 0)

Before starting research, check if Kit MCP is available:

**If Kit MCP available:**
1. Use `open_repository` to load current repository
   - This enables caching and AST-aware tools
   - Returns `repo_id` for use in subsequent Kit MCP calls
2. Agents will automatically use Kit MCP tools when appropriate

**If Kit MCP unavailable:**
- Agents will use traditional Grep/Glob/Read tools
- Workflow proceeds normally (graceful degradation)

This check happens automatically - just be aware of which tools are available.
```

Update agent spawning section (around line 40):
```markdown
**For codebase research:**
- Use the **codebase-locator** agent (MCP-enhanced if available)
- Use the **codebase-analyzer** agent (MCP-enhanced if available)
- Use the **codebase-pattern-finder** agent (MCP-enhanced if available)

**For external package research:**
- If researching third-party libraries/APIs, use **mcp-package-researcher**
- Combines Ref MCP (docs) + Kit MCP package search (source code)
- More focused than generic web search

**For thoughts directory research:**
- Use the **thoughts-locator** agent (unchanged)
- Use the **thoughts-analyzer** agent (unchanged)
```

Add MCP context to agent prompts (example):
```markdown
Task 1 - Locate Authentication Components:
Use codebase-locator to find all authentication-related code.
Search for: authentication, login, session management, JWT, OAuth.
${KIT_MCP_AVAILABLE ? "Kit MCP is available - use AST-aware tools for code files." : ""}
Return: Organized list of files by purpose.
```

**Rationale**: Minimal changes to workflow. MCP availability check at start, then agents automatically use enhanced tools. Preserves all critical ordering and synthesis patterns.

---

### /create_plan - PRIORITY 2

**Current Flow**:
1. Read ticket + research documents
2. Spawn research tasks to verify current state
3. Present findings
4. Get structure approval
5. Write detailed plan
6. Iterate on feedback

**Enhanced Flow** (with MCP):
- Same flow, but research tasks can use MCP-enhanced agents
- If planning external package integration, spawn mcp-package-researcher

**Specific Changes**:

Update initial research tasks (around line 44):
```markdown
**Conduct initial research** (if needed):
- Spawn focused research tasks using MCP-enhanced agents
- For codebase understanding: Use codebase-analyzer (with Kit MCP)
- For external packages: Use mcp-package-researcher (with Ref + Kit MCP)
- For dependency impact: Use codebase-dependency-tracer (with Kit MCP)
```

Add to success criteria guidelines (around line 318):
```markdown
**Use specific, executable commands:**
- ✅ `make test` (if Kit MCP available, may use `review_diff` for pre-commit review)
- ✅ `npm run lint`
- ✅ `python -m pytest tests/integration`
- ❌ "run tests" (too vague)
```

**Rationale**: Planning already spawns research tasks. Just ensure those tasks can use MCP-enhanced agents. No workflow changes needed.

---

### /implement_plan - PRIORITY 3

**Current Flow**:
1. Read plan
2. Follow plan phase by phase
3. Update checkboxes as work completes
4. Run success criteria at natural stopping points

**Enhanced Flow** (with MCP):
- Same flow
- **Optional**: Use `review_diff` before committing changes

**Specific Changes**:

Add optional review step (before final verification):
```markdown
## Optional: Pre-Commit Code Review

If Kit MCP available and you want AI code review:
1. Stage changes: `git add <files>`
2. Use Kit MCP `review_diff` tool: `review_diff(repo_id, "--staged")`
3. Review feedback and address issues
4. Then proceed with commit

This is optional - use judgment on whether formal review needed.
```

**Rationale**: Implementation follows plan. MCP doesn't change implementation process, just adds optional review capability.

---

### Other Commands - Lower Priority

**`/commit`** - No changes needed (file operations only)

**`/describe_pr`** - Could add `review_diff` integration:
```markdown
**Optional AI Review** (if Kit MCP available):
- Run `review_diff` on PR diff
- Include key findings in PR description
- Note: This is supplementary to human review
```

**`/create_handoff`** - No changes needed (documentation only)

**`/resume_handoff`** - Validation tasks can use MCP-enhanced agents

**`/linear`** - No changes needed (Linear API only)

**`/debug`** - Could use Kit MCP for codebase investigation tasks

---

## Part VIII: CLAUDE.md Integration

### Current Project CLAUDE.md

Your project already has basic Kit MCP guidance:
```markdown
# Kit MCP Usage Guidelines

When working with code in this project, always:
1. Start with Repository Context (open_repository, get_file_tree, extract_symbols)
2. For Code Understanding (search_text, find_symbol_usages, get_dependency_graph)
3. For Documentation (deep_research_package)
4. Best Practices (gather context, incremental extraction, research dependencies)
```

### Enhanced CLAUDE.md

Add comprehensive tool selection guidance:

```markdown
# MCP Integration Guidelines

This project uses Kit MCP (codebase intelligence) and Ref MCP (documentation research).

## Tool Selection Decision Tree

### For Local Codebase Work
**Use Kit MCP when:**
- Finding symbol definitions → `grep_ast`, `extract_symbols`
- Understanding code structure → `get_file_tree`, `extract_symbols`
- Tracing symbol usage → `find_symbol_usages`
- Analyzing dependencies → `get_dependency_graph`
- Pattern matching on AST → `grep_ast`

**Use built-in tools when:**
- Reading specific files → `Read` (faster, complete)
- Searching non-code files → `Grep` (configs, markdown, JSON)
- File path patterns → `Glob`
- One-off searches → `Grep` (no caching benefit)

### For External Documentation
**Use Ref MCP when:**
- Researching APIs → `ref_search_documentation`
- Reading documentation → `ref_read_url`
- Finding integration guides → `ref_search_documentation`
- Debugging third-party packages → `ref_search_documentation`

**Use WebSearch when:**
- Current events or news
- Real-time data
- General information (non-technical)
- Documentation not in Ref's curated index

## Repository Initialization

At the start of codebase work:
1. `open_repository` to load context (enables caching)
2. `get_file_tree` to understand structure
3. `extract_symbols` for fast symbol overview
4. Then proceed with focused analysis

## Agent Usage

Commands will automatically use MCP-enhanced agents when available:
- `codebase-locator`: Grep/Glob/LS + Kit MCP AST tools
- `codebase-analyzer`: Read + Kit MCP symbol/dependency tools
- `codebase-pattern-finder`: Grep/Read + Kit MCP pattern tools
- `external-doc-researcher`: WebSearch + Ref MCP documentation tools
- `mcp-package-researcher`: Ref + Kit package research tools

## Graceful Degradation

All workflows work with or without MCP:
- If MCP available: Enhanced AST awareness, token efficiency
- If MCP unavailable: Traditional Grep/Read/Glob tools
- No workflow changes based on MCP availability
```

---

## Part IX: Implementation Phases

### Phase 1: Foundation (Week 1)
**Goal**: MCP-enhanced agents with fallback capability

**Tasks**:
1. ✅ Create MCP-enhanced agent versions
   - codebase-locator (add Kit tools)
   - codebase-analyzer (add Kit tools)
   - codebase-pattern-finder (add Kit tools)
   - external-doc-researcher (rename + add Ref tools)
2. ✅ Create new agents
   - mcp-package-researcher
   - codebase-dependency-tracer
3. ✅ Update CLAUDE.md with tool selection guidelines
4. ✅ Test agents individually with Kit/Ref MCP available and unavailable

**Success Criteria**:
- [ ] All agents work with MCP available
- [ ] All agents gracefully degrade without MCP
- [ ] Agent outputs same structure as before (enhanced content)

---

### Phase 2: Research Command Integration (Week 2)
**Goal**: Perfect `/research_codebase` command with full MCP integration

**Tasks**:
1. ✅ Add MCP availability check to research command
2. ✅ Update agent spawning to use MCP-enhanced versions
3. ✅ Add mcp-package-researcher option for external dependencies
4. ✅ Test full research workflow end-to-end
5. ✅ Validate research document quality (should be more comprehensive)

**Success Criteria**:
- [ ] Research command works with Kit + Ref MCP
- [ ] Research command works without MCP (fallback)
- [ ] Research documents include AST-level insights when MCP available
- [ ] External package research uses Ref MCP efficiently
- [ ] All critical ordering rules preserved (read files first, wait for agents, etc.)

---

### Phase 3: Planning Command Integration (Week 3)
**Goal**: Enhance `/create_plan` with MCP-powered research

**Tasks**:
1. ✅ Update initial research task spawning
2. ✅ Enable mcp-package-researcher for dependency evaluation
3. ✅ Enable codebase-dependency-tracer for impact analysis
4. ✅ Test planning workflow with complex dependencies
5. ✅ Validate plan quality (should identify dependencies better)

**Success Criteria**:
- [ ] Plans include dependency impact analysis
- [ ] External package integration plans reference Ref MCP docs
- [ ] Planning process remains interactive and iterative
- [ ] No open questions in final plans (rule preserved)

---

### Phase 4: Implementation & Review Integration (Week 4)
**Goal**: Add optional AI review capabilities

**Tasks**:
1. ✅ Add `review_diff` integration to `/implement_plan`
2. ✅ Add `review_diff` integration to `/describe_pr`
3. ✅ Create review workflow guidelines
4. ✅ Test review quality on sample diffs

**Success Criteria**:
- [ ] AI review available but optional
- [ ] Review findings actionable
- [ ] Review doesn't block workflow if unavailable

---

### Phase 5: Optimization & Monitoring (Week 5)
**Goal**: Measure impact and optimize usage

**Tasks**:
1. ✅ Track MCP usage patterns (which tools used most)
2. ✅ Measure token reduction (compare with/without Ref MCP)
3. ✅ Identify bottlenecks (slow operations)
4. ✅ Optimize agent prompts based on learnings
5. ✅ Document best practices

**Success Criteria**:
- [ ] 50%+ token reduction for documentation research (Ref MCP)
- [ ] Faster symbol discovery (Kit MCP caching)
- [ ] No workflow regressions
- [ ] Team documentation updated with learnings

---

## Part X: Risks & Mitigations

### Risk 1: MCP Server Unavailability
**Impact**: Workflow breaks if hard dependency on MCP

**Mitigation**:
- ✅ Progressive enhancement pattern (MCP enhances, doesn't replace)
- ✅ All agents have fallback to built-in tools
- ✅ Commands work with or without MCP
- ✅ Graceful degradation tested in Phase 1

**Monitoring**: Track MCP availability uptime, alert on failures

---

### Risk 2: Agent Scope Creep
**Impact**: MCP tools tempt agents to make recommendations vs document

**Mitigation**:
- ✅ Preserve all negative constraints ("DO NOT suggest/critique/improve")
- ✅ Add MCP-specific negative constraints
  - "Don't use get_dependency_graph to identify 'bad' dependencies"
  - "Don't use grep_ast to critique naming conventions"
- ✅ Maintain "REMEMBER: You are a documentarian" sections
- ✅ Test agent outputs for unwanted recommendations

**Monitoring**: Review agent outputs regularly for scope violations

---

### Risk 3: Complexity Overhead
**Impact**: MCP integration makes agents harder to understand and maintain

**Mitigation**:
- ✅ Keep agent core purpose unchanged (locator finds WHERE, analyzer explains HOW)
- ✅ Add MCP as optional sections, not replace existing strategy
- ✅ Maintain example-heavy templates
- ✅ Document when to use which MCP tool
- ✅ Create decision trees for tool selection

**Monitoring**: Onboarding time for new team members, feedback on agent clarity

---

### Risk 4: Token Inefficiency
**Impact**: Using wrong MCP tool wastes tokens (e.g., deep_research_package when Read would work)

**Mitigation**:
- ✅ Clear guidelines in CLAUDE.md (when to use Kit vs built-in)
- ✅ Agent prompts specify tool selection logic
- ✅ Examples of good/bad tool usage
- ✅ Monitor token usage per command/agent

**Monitoring**: Track token usage, compare MCP vs non-MCP runs

---

### Risk 5: Breaking Changes in MCP Servers
**Impact**: Kit/Ref MCP API changes break agents

**Mitigation**:
- ✅ Version pin MCP servers if possible
- ✅ Fallback to built-in tools if MCP calls fail
- ✅ Monitor MCP server changelogs
- ✅ Test agents after MCP updates

**Monitoring**: Subscribe to Kit MCP and Ref MCP release notes

---

### Risk 6: Loss of HumanLayer Workflow Brilliance
**Impact**: MCP integration inadvertently changes workflow patterns that work well

**Mitigation**:
- ✅ **DO NOT CHANGE**:
  - Research → Plan → Implement → Handoff cycle
  - Interactive planning (present → refine → approve)
  - Critical ordering rules (read fully, wait for agents, no placeholders)
  - Documentarian role constraints
  - Success criteria bifurcation (automated vs manual)
- ✅ Extensive testing of full workflow (not just individual agents)
- ✅ User feedback on workflow quality before/after

**Monitoring**: Track workflow satisfaction, compare pre/post MCP integration

---

## Part XI: Success Criteria

### Quantitative Metrics

**Token Efficiency**:
- ✅ 50%+ reduction in tokens for external documentation research (Ref MCP vs WebSearch)
- ✅ 30%+ reduction in codebase research time (Kit MCP caching)

**Speed**:
- ✅ Symbol discovery 2x faster (extract_symbols vs grep)
- ✅ Dependency analysis available (previously manual)

**Quality**:
- ✅ Research documents include AST-level insights
- ✅ Plans include dependency impact analysis
- ✅ External package research more comprehensive

### Qualitative Metrics

**Workflow Preservation**:
- ✅ All critical ordering rules followed
- ✅ Interactive planning still works
- ✅ Documentarian role maintained
- ✅ No unwanted agent behavior (recommendations, critique)

**Usability**:
- ✅ Commands work with or without MCP
- ✅ Tool selection intuitive (clear decision trees)
- ✅ Agent outputs understandable
- ✅ Onboarding straightforward

**Reliability**:
- ✅ Graceful degradation when MCP unavailable
- ✅ No workflow breaks from MCP errors
- ✅ Consistent quality across runs

---

## Part XII: Quick Reference

### When to Use Kit MCP
```
✅ Finding where symbols are defined
✅ Understanding code architecture
✅ Tracing symbol usage across files
✅ AST-based pattern matching
✅ Dependency graph analysis
✅ Researching external package source code
✅ Repeated analysis (caching benefit)

❌ Reading specific files (use Read)
❌ Searching non-code files (use Grep)
❌ File path patterns (use Glob)
❌ One-off searches (no caching benefit)
```

### When to Use Ref MCP
```
✅ API documentation lookup
✅ Framework best practices
✅ Package usage patterns
✅ Debugging third-party libraries
✅ Integration guides
✅ Private repository documentation

❌ Current events or news (use WebSearch)
❌ Real-time data (use WebSearch)
❌ General knowledge (use WebSearch)
❌ Local codebase docs (use Kit or Read)
```

### Agent Selection
```
codebase-locator → Find WHERE code lives
codebase-analyzer → Understand HOW code works
codebase-pattern-finder → Find examples to model
external-doc-researcher → Research documentation
mcp-package-researcher → Deep package research
codebase-dependency-tracer → Map dependencies
thoughts-locator → Find thoughts documents
thoughts-analyzer → Extract insights from thoughts
```

### Critical Rules to Preserve
```
1. Read files FULLY before spawning sub-tasks
2. Wait for ALL sub-agents before synthesizing
3. No open questions in final plans
4. Document what IS, not what SHOULD BE
5. Automated vs Manual success criteria separation
6. Interactive planning (present → refine → approve)
7. No placeholders in documents
```

---

## Appendix A: Tool Selection Examples

### Example 1: Finding Authentication Implementation

**Bad Approach** (inefficient):
```
1. Read every file in auth/ directory
2. Manually trace function calls
3. Grep for "authenticate" across codebase
```

**Good Approach** (MCP-enhanced):
```
1. Use get_file_tree to understand auth/ structure
2. Use extract_symbols to list exported authentication functions
3. Use find_symbol_usages to trace where they're called
4. Use Read to get implementation details of key functions
5. Use get_dependency_graph to understand auth module dependencies
```

**Rationale**: Kit MCP provides structured view first (file tree, symbols), then targeted reading. Saves tokens and time.

---

### Example 2: Integrating Stripe Webhooks

**Bad Approach** (inefficient):
```
1. WebSearch "stripe webhooks"
2. WebFetch entire Stripe docs page (20k+ tokens)
3. Extract relevant sections manually
4. Repeat for error handling, idempotency, signatures
```

**Good Approach** (MCP-enhanced):
```
1. ref_search_documentation("Stripe webhook signature verification Python")
2. ref_read_url(stripe_webhooks_url) → returns 500 tokens (smart chunking)
3. ref_search_documentation("Stripe webhook idempotency handling")
4. ref_read_url(stripe_idempotency_url) → returns 300 tokens
5. Synthesize with 800 total tokens vs 20k+
```

**Rationale**: Ref MCP's smart chunking and session awareness massively reduce tokens while improving relevance.

---

### Example 3: Understanding Module Dependencies

**Bad Approach** (manual):
```
1. Grep for "import" statements across codebase
2. Manually trace what imports what
3. Draw dependency graph by hand
4. Miss transitive dependencies
```

**Good Approach** (MCP-enhanced):
```
1. Use get_dependency_graph for target module
2. Automatically captures imports, exports, dependencies
3. Use extract_symbols to understand what each module exports
4. Use Read to verify actual usage
5. Get complete dependency picture with transitive chains
```

**Rationale**: get_dependency_graph automates tedious manual work, provides systematic view.

---

## Appendix B: Migration Checklist

### Before Starting
- [ ] Kit MCP installed and configured
- [ ] Ref MCP installed and configured
- [ ] Test Kit MCP tools individually (open_repository, grep_ast, extract_symbols)
- [ ] Test Ref MCP tools individually (ref_search_documentation, ref_read_url)
- [ ] Backup current .claude/ directory

### Phase 1: Agent Enhancement
- [ ] Create codebase-locator with Kit MCP tools
- [ ] Create codebase-analyzer with Kit MCP tools
- [ ] Create codebase-pattern-finder with Kit MCP tools
- [ ] Rename web-search-researcher → external-doc-researcher, add Ref MCP
- [ ] Create mcp-package-researcher (new)
- [ ] Create codebase-dependency-tracer (new)
- [ ] Test each agent with MCP available
- [ ] Test each agent with MCP unavailable (graceful degradation)

### Phase 2: Research Command
- [ ] Add MCP availability check
- [ ] Update agent spawning section
- [ ] Add mcp-package-researcher option
- [ ] Test full research workflow (codebase + external packages)
- [ ] Validate research document quality
- [ ] Test graceful degradation

### Phase 3: Planning Command
- [ ] Update initial research task spawning
- [ ] Enable mcp-package-researcher for dependencies
- [ ] Enable codebase-dependency-tracer for impact analysis
- [ ] Test planning workflow
- [ ] Validate plan quality

### Phase 4: Implementation & Review
- [ ] Add review_diff integration to /implement_plan
- [ ] Add review_diff integration to /describe_pr
- [ ] Test review workflow
- [ ] Validate review quality

### Phase 5: Monitoring
- [ ] Set up token usage tracking
- [ ] Set up MCP availability monitoring
- [ ] Document learnings
- [ ] Create team best practices guide

---

## Conclusion

This integration strategy enhances HumanLayer's proven workflow with MCP capabilities while preserving its core strengths:

**Preserved**:
- ✅ Research → Plan → Implement → Handoff cycle
- ✅ Documentarian role constraints (no critique/suggestions)
- ✅ Interactive, iterative planning
- ✅ Critical ordering rules (read fully, wait for agents)
- ✅ Success criteria bifurcation (automated vs manual)
- ✅ Graceful degradation (works with or without MCP)

**Enhanced**:
- ✅ AST-aware codebase analysis (Kit MCP)
- ✅ Token-efficient documentation research (Ref MCP)
- ✅ Dependency mapping capability (Kit MCP)
- ✅ Package source code research (Kit MCP)
- ✅ Smart documentation chunking (Ref MCP)

**Result**: A more powerful development workflow that maintains the thoughtful, systematic approach of HumanLayer while leveraging modern MCP capabilities for efficiency and depth.
