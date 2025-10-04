---
date: 2025-10-04
topic: Dependency Analysis Options - Kit MCP Limitations & Alternatives
tags: [mcp, kit-mcp, dependency-analysis, research, python-api, dependency-mcp]
status: complete
type: technical_research
---

# Dependency Analysis Options: Kit MCP vs Alternatives

## Problem Statement

**The `get_dependency_graph` tool does NOT exist in Kit MCP.**

The integration strategy referenced `mcp__kit-dev__get_dependency_graph` as a Kit MCP tool, but testing revealed this tool is unavailable. This document explains the confusion and provides alternatives.

---

## Root Cause: Python API ≠ MCP Tools

Kit has **TWO separate interfaces** with different capabilities:

### 1. Kit Python API (Direct Library Usage)

**Has full dependency analysis:**
```python
from kit import Repository

repo = Repository("/path/to/codebase")
analyzer = repo.get_dependency_analyzer('python')  # or 'terraform'

# Available methods:
graph = analyzer.build_dependency_graph()
analyzer.export_dependency_graph(output_format='dot', output_path='deps.dot')
cycles = analyzer.find_cycles()
analyzer.visualize_dependencies('deps_graph.svg', format='svg')
context = analyzer.generate_llm_context(max_tokens=4000)
```

**Key classes:**
- `DependencyAnalyzer` - Main dependency analysis class
- Methods: `build_dependency_graph()`, `find_cycles()`, `visualize_dependencies()`, `export_dependency_graph()`

**Language support:** Python, Terraform (as of version 2.1.2)

### 2. Kit MCP Server (kit-dev-mcp)

**Does NOT expose dependency graph tools.**

**Available MCP tools (15 total):**
1. `open_repository` - Open local/remote repos
2. `get_file_tree` - Directory structure
3. `get_file_content` - Read files
4. `search_code` - Pattern-based search
5. `grep_code` - Fast literal search
6. `grep_ast` - AST pattern matching
7. `extract_symbols` - Extract functions/classes/symbols
8. `find_symbol_usages` - Find where symbols are used
9. `get_code_summary` - AI code summaries
10. `review_diff` - AI diff review
11. `deep_research_package` - Package docs
12. `package_search_grep` - Search package source
13. `package_search_hybrid` - Semantic package search
14. `package_search_read_file` - Read package files
15. `get_git_info` - Git metadata

**NOT available:**
- ❌ `get_dependency_graph`
- ❌ `DependencyAnalyzer` API
- ❌ Automated graph generation
- ❌ Cycle detection
- ❌ Dependency visualization

---

## Option 1: Manual Dependency Tracing (Current Implementation)

**What:** Use available Kit MCP tools to manually trace dependencies.

**How:**
1. Use `extract_symbols` to find imports in each file
2. Use `find_symbol_usages` to trace where symbols are used
3. Manually aggregate import relationships
4. Build dependency chains by following imports recursively

**Pros:**
- ✅ Works with existing Kit MCP setup
- ✅ No additional dependencies
- ✅ `extract_symbols` is cached (fast on repeat calls)
- ✅ Provides granular control over analysis

**Cons:**
- ❌ Manual aggregation required
- ❌ No automatic graph generation
- ❌ No cycle detection
- ❌ No visualization
- ❌ `find_symbol_usages` returns dependency noise (node_modules, vendor, etc.)

**Best for:**
- Understanding specific import chains
- Tracing how modules connect
- Documenting dependency relationships
- When you need detailed, manual control

**Example workflow:**
```
1. extract_symbols("src/api/routes.js") → See imports: ["../handlers/webhook", "express"]
2. find_symbol_usages("WebhookHandler") → Find where it's imported
3. Read each importing file to verify usage
4. Repeat for each module in the chain
5. Document relationships manually
```

**Implementation:** `codebase-dependency-tracer` agent uses this approach.

---

## Option 2: Separate dependency-mcp Server

**What:** Install a dedicated MCP server for dependency analysis.

**Package:** `dependency-mcp` (GitHub: mkearl/dependency-mcp)

**Features:**
- ✅ Has actual `get_dependency_graph` tool
- ✅ Multi-language support (TypeScript, JavaScript, C#, Python, Go, etc.)
- ✅ Automatic graph generation (JSON or DOT format)
- ✅ Architectural analysis capabilities
- ✅ File metadata extraction

**Installation:**
```bash
npm install -g dependency-mcp

# Add to Claude Desktop config (or equivalent):
{
  "mcpServers": {
    "dependency-mcp": {
      "command": "dependency-mcp"
    }
  }
}
```

**Usage:**
```javascript
const result = await client.callTool("DependencyMCP", "get_dependency_graph", {
  path: "/path/to/project",
  format: "dot"  // or "json" (default)
});
```

**Pros:**
- ✅ Automated dependency graph generation
- ✅ Multi-language support
- ✅ Export to DOT/JSON formats
- ✅ No manual aggregation needed

**Cons:**
- ❌ Requires additional MCP server installation
- ❌ Separate configuration needed
- ❌ Different tool ecosystem
- ❌ May not integrate with Kit MCP caching

**Best for:**
- Projects needing automated dependency graphs
- Multi-language codebases
- When visualization is required
- Production dependency analysis workflows

---

## Option 3: Kit Python API (Direct Usage)

**What:** Use Kit library directly via Python scripts instead of MCP.

**Installation:**
```bash
pip install kit
```

**Usage:**
```python
from kit import Repository

# Initialize repository
repo = Repository("/path/to/codebase")

# Get dependency analyzer (Python or Terraform)
analyzer = repo.get_dependency_analyzer('python')

# Build dependency graph
graph = analyzer.build_dependency_graph()

# Export in various formats
analyzer.export_dependency_graph(
    output_format='dot',    # or 'json', 'graphml'
    output_path='dependencies.dot'
)

# Find circular dependencies
cycles = analyzer.find_cycles()

# Create visualization (requires Graphviz)
analyzer.visualize_dependencies(
    output_path='deps_graph',
    format='svg'  # or 'png', 'pdf'
)

# Generate LLM-friendly context
context = analyzer.generate_llm_context(
    max_tokens=4000,
    output_format='markdown'
)
```

**Full API Reference:**

**`build_dependency_graph()`**
- Returns: Dictionary of component dependencies
- Example: `{'module_a': ['module_b', 'module_c']}`

**`export_dependency_graph(output_format="json", output_path=None)`**
- `output_format`: "json", "dot", "graphml"
- `output_path`: Optional file path
- Returns: File path or formatted string

**`find_cycles()`**
- Returns: List of circular dependency cycles
- Example: `[['module_a', 'module_b', 'module_a']]`

**`visualize_dependencies(output_path, format="png")`**
- `format`: "png", "svg", "pdf"
- **Requires:** Graphviz installed (`apt install graphviz` or `brew install graphviz`)
- Returns: Path to generated visualization

**`generate_llm_context(max_tokens=4000, output_format="markdown", output_path=None)`**
- Creates natural language dependency description
- Token-limited for LLM consumption
- `output_format`: "markdown" or "text"

**Language-specific methods:**

**Python:**
- `get_module_dependencies(module_name)` - Dependencies for specific module
- `get_file_dependencies(file_path)` - Dependencies for specific file

**Terraform:**
- Similar methods for Terraform modules

**Pros:**
- ✅ Full dependency analysis features
- ✅ Automated graph generation
- ✅ Cycle detection
- ✅ Visualization support
- ✅ Multiple export formats
- ✅ LLM context generation

**Cons:**
- ❌ Not available via MCP
- ❌ Requires Python scripting
- ❌ Separate workflow from agent-based analysis
- ❌ Limited language support (Python, Terraform only)
- ❌ Visualization requires Graphviz installation

**Best for:**
- Python or Terraform projects
- Automated CI/CD dependency checks
- Generating dependency reports
- Creating dependency visualizations
- One-off dependency analysis tasks

**Example Python script:**
```python
#!/usr/bin/env python3
from kit import Repository

def analyze_dependencies(repo_path, output_path):
    """Analyze dependencies and generate report."""
    repo = Repository(repo_path)
    analyzer = repo.get_dependency_analyzer('python')

    # Build graph
    graph = analyzer.build_dependency_graph()

    # Find cycles
    cycles = analyzer.find_cycles()

    # Export DOT format
    analyzer.export_dependency_graph('dot', f'{output_path}/deps.dot')

    # Generate visualization
    analyzer.visualize_dependencies(f'{output_path}/deps', format='svg')

    # Print summary
    print(f"Total modules: {len(graph)}")
    print(f"Circular dependencies: {len(cycles)}")
    if cycles:
        print("Cycles found:")
        for cycle in cycles:
            print(f"  {' -> '.join(cycle)}")

if __name__ == '__main__':
    analyze_dependencies('/path/to/project', './dependency-report')
```

---

## Comparison Matrix

| Feature | Manual Tracing (Option 1) | dependency-mcp (Option 2) | Kit Python API (Option 3) |
|---------|---------------------------|---------------------------|---------------------------|
| **MCP Integration** | ✅ Via Kit MCP | ✅ Separate server | ❌ Python only |
| **Auto Graph Generation** | ❌ Manual | ✅ Yes | ✅ Yes |
| **Cycle Detection** | ❌ Manual | ✅ Yes | ✅ Yes |
| **Visualization** | ❌ No | ⚠️ Limited | ✅ Yes (requires Graphviz) |
| **Language Support** | ✅ All (via grep) | ✅ Multi-language | ⚠️ Python, Terraform only |
| **Setup Complexity** | ✅ None (uses Kit MCP) | ⚠️ Additional MCP server | ⚠️ Python environment + Graphviz |
| **Export Formats** | ❌ None | ✅ JSON, DOT | ✅ JSON, DOT, GraphML |
| **Caching** | ✅ Kit MCP cache | ❓ Unknown | ✅ Yes |
| **Best Use Case** | Detailed manual analysis | Multi-language automation | Python/Terraform projects |

---

## Recommendation by Use Case

### For Agent-Based Workflow (Current Implementation)
**Use:** Option 1 (Manual Tracing)
- Already integrated via `codebase-dependency-tracer` agent
- Works with existing Kit MCP setup
- Good for understanding specific dependency chains
- No additional dependencies

### For Automated Dependency Reports
**Use:** Option 3 (Kit Python API)
- Full-featured dependency analysis
- Suitable for CI/CD integration
- Generates visualizations and reports
- Best for Python/Terraform projects

### For Multi-Language Projects Needing Automation
**Use:** Option 2 (dependency-mcp)
- Supports TypeScript, JavaScript, C#, Python, Go, etc.
- MCP-based (integrates with Claude workflow)
- Automated graph generation
- Worth the setup overhead for large projects

---

## Current Implementation Status

**As of 2025-10-04:**

1. **`codebase-dependency-tracer` agent:** Uses Option 1 (manual tracing)
   - Tools: `extract_symbols`, `find_symbol_usages`, `Grep`, `Read`
   - Performs manual dependency aggregation
   - Documents import chains and module relationships

2. **`codebase-analyzer` agent:** Updated to remove `get_dependency_graph` references
   - Uses `extract_symbols` + `find_symbol_usages` for dependency tracing
   - Manual analysis approach

3. **`CLAUDE.md`:** Updated to reflect manual tracing approach
   - Removed `get_dependency_graph` from tool list
   - Added reference to this document for automated options

---

## Migration Path

If you need automated dependency graphs in the future:

### Path A: Add dependency-mcp (Multi-language)
```bash
# 1. Install dependency-mcp
npm install -g dependency-mcp

# 2. Add to MCP config
# Edit Claude Desktop config or equivalent

# 3. Update agents to use get_dependency_graph tool from dependency-mcp
```

### Path B: Use Kit Python API (Python/Terraform only)
```bash
# 1. Install Kit Python package
pip install kit

# 2. Create analysis scripts
# Use examples above

# 3. Run as separate workflow step
python analyze_dependencies.py
```

---

## Key Learnings

1. **Always verify tool availability** before referencing in documentation
2. **Python API ≠ MCP tools** - Check both interfaces
3. **grep_ast unreliable** - Returns AST container nodes, not code structures
4. **extract_symbols works well** - Cached, fast, reliable for Python/TypeScript/Go
5. **find_symbol_usages returns noise** - Filter out node_modules, vendor, etc.

---

## References

- Kit MCP GitHub: https://github.com/cased/kit
- Kit MCP Documentation: https://kit-mcp.cased.com
- dependency-mcp GitHub: https://github.com/mkearl/dependency-mcp
- Integration Strategy: `thoughts/shared/handoffs/general/2025-10-04_17-06-19_mcp-integration-strategy.md`
- Phase 1 Handoff: `thoughts/shared/handoffs/general/2025-10-04_19-37-24_mcp-phase1-complete.md`
