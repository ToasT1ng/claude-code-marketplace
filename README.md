# Claude Code Marketplace

A community plugin marketplace for [Claude Code](https://claude.ai/code).

## Add this marketplace

```
/plugin marketplace add ToasT1ng/claude-code-marketplace
```

## Install plugins

```
/plugin install scaffold@community-plugins
```

## Available plugins

| Plugin | Type | Description |
|--------|------|-------------|
| `scaffold` | Skill | Read codebase patterns and generate matching code |

## Structure

```
.claude-plugin/
  marketplace.json        ← marketplace catalog
plugins/
  scaffold/
    .claude-plugin/
      plugin.json         ← plugin manifest
    skills/
      scaffold/
        SKILL.md          ← skill prompt
```

## Contributing

Add a plugin: create a directory under `plugins/` following the structure above, then add an entry to `.claude-plugin/marketplace.json`.
