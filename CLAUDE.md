# MCP Integration Guidelines

This project uses Kit MCP (codebase intelligence) and Ref MCP (documentation research) to enhance code analysis and research capabilities.

## Tool Selection Decision Tree

### For Local Codebase Work

**Use Kit MCP when:**
- Finding symbol definitions → `grep_ast`, `extract_symbols`
- Understanding code structure → `get_file_tree`, `extract_symbols`
- Tracing symbol usage → `find_symbol_usages`
- Analyzing dependencies → `get_dependency_graph`
- Pattern matching on AST → `grep_ast`

**Use built-in tools when:**
- Reading specific files → `Read` (faster, complete content)
- Searching non-code files → `Grep` (configs, markdown, JSON)
- File path patterns → `Glob` (finding files by name)
- One-off searches → `Grep` (no caching benefit from Kit MCP)

**Note:** Kit MCP tools work best with Python and TypeScript. For other languages (Go, Rust, etc.), built-in tools may be more reliable.

### For External Documentation

**Use Ref MCP when:**
- Researching APIs → `ref_search_documentation`
- Reading documentation → `ref_read_url`
- Finding integration guides → `ref_search_documentation`
- Debugging third-party packages → `ref_search_documentation`

**Use WebSearch when:**
- Current events or news (post-January 2025)
- Real-time data
- General information (non-technical)
- Documentation not in Ref's curated index

**Use WebFetch when:**
- Reading specific URLs not available via Ref MCP
- Fetching content from blogs, tutorials, or non-official sources

## Repository Initialization

At the start of codebase work:
1. `open_repository` to load context (enables Kit MCP caching)
2. `get_file_tree` to understand structure (Git-aware, respects .gitignore)
3. `extract_symbols` for fast symbol overview (results are cached)
4. Then proceed with focused analysis using appropriate tools

**Important:** Repository IDs (e.g., `repo_0`) reset between Claude Code sessions. Re-initialize when starting a new session.

## Agent Usage

Commands will automatically use MCP-enhanced agents when available:

- **`codebase-locator`**: Locates files and components using Grep/Glob/LS + Kit MCP AST tools
  - Use when: Finding where features are implemented, locating specific code

- **`codebase-analyzer`**: Analyzes implementation details using Read + Kit MCP symbol/dependency tools
  - Use when: Understanding how something works, analyzing code structure

- **`codebase-pattern-finder`**: Finds similar implementations using Grep/Read + Kit MCP pattern tools
  - Use when: Looking for examples, finding existing patterns to follow

- **`external-doc-researcher`**: Researches external documentation using Ref MCP + WebSearch
  - Use when: Researching frameworks, APIs, best practices from official docs

- **`mcp-package-researcher`**: Deep package research using Ref MCP (docs) + Kit MCP package search (source)
  - Use when: Planning package integration, debugging third-party libraries, evaluating packages

- **`codebase-dependency-tracer`**: Maps dependency relationships using Kit MCP's get_dependency_graph
  - Use when: Understanding module boundaries, tracing import chains, analyzing architecture

## Graceful Degradation

All workflows work with or without MCP:
- **If MCP available:** Enhanced AST awareness, token efficiency, cached symbol extraction
- **If MCP unavailable:** Traditional Grep/Read/Glob tools work as fallback
- **No workflow changes:** Agents automatically choose best available tools

## Best Practices

1. **Incremental Symbol Extraction:** `extract_symbols` results are cached—use it liberally
2. **Context Before Analysis:** Use `get_file_tree` and `extract_symbols` before diving deep
3. **Research Dependencies:** Use `deep_research_package` before integrating new packages
4. **Verify Findings:** Kit MCP tools may return large result sets—always review and filter
5. **Language Support:** Kit MCP works best with Python/TypeScript; use built-in tools for other languages

Remember: Better context = Better code suggestions