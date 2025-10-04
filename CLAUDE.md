# Kit MCP Usage Guidelines

When working with code in this project, always:

1. **Start with Repository Context**
   - Use `open_repository` to load the codebase
   - Use `get_file_tree` to understand structure
   - Use `extract_symbols` for fast analysis

2. **For Code Understanding**
   - Use `search_text` for finding implementations
   - Use `find_symbol_usages` to track usage
   - Use `get_dependency_graph` to understand relationships

3. **For Documentation**
   - Use `deep_research_package` for comprehensive package research

4. **Best Practices**
   - Always gather context before making suggestions
   - Use incremental symbol extraction (it's cached!)
   - Research dependencies before using new packages

Remember: Better context = Better code suggestions