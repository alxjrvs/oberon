# 1password-mcp

The whole plugin is its marketplace entry: one MCP server, the 1Password desktop app's own, by
absolute path. This directory exists so the entry has a root of its own; a plugin whose source
is the marketplace repo itself would also load every skill in `skills/`, under a second name.
