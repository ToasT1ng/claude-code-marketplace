# Claude Code Marketplace

This is a Claude Code plugin marketplace repository. It contains:

- `.claude-plugin/marketplace.json` — the marketplace catalog
- `plugins/` — individual plugin directories, each with a `.claude-plugin/plugin.json` manifest

When adding or modifying plugins, follow the official Claude Code plugin spec:
- Each plugin lives in `plugins/<name>/`
- Manifest goes in `plugins/<name>/.claude-plugin/plugin.json`
- Skills go in `plugins/<name>/skills/<skill-name>/SKILL.md`
- Hooks go in `plugins/<name>/hooks/hooks.json`
- MCP servers go in `plugins/<name>/.mcp.json`

Do not add web frameworks, build tools, or package managers to this repo.
