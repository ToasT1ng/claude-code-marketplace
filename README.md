# Claude Code Marketplace

A community plugin marketplace for [Claude Code](https://claude.ai/code).

## Add this marketplace

```
/plugin marketplace add ToasT1ng/claude-code-marketplace
```

## Install plugins

```
/plugin install git-helper@community-plugins
/plugin install spring-kotlin-clean-code-helper@community-plugins
```

## Available plugins

### 🔧 git-helper

Git workflow skills for commits, branches, and PR reviews.

| Skill | Description |
|-------|-------------|
| `git-commit-convention` | Write a gitmoji commit message for staged changes |
| `git-branch-convention` | Generate a branch name following convention |
| `git-pr-review-local` | Review a PR locally against the base branch |
| `git-pr-review-comments` | Respond to or resolve PR review comments |

### 🌱 spring-kotlin-clean-code-helper

Clean code and architecture convention skills for Spring Kotlin.

| Skill | Description |
|-------|-------------|
| `spring-kotlin-clean-code-convention` | Apply clean code conventions to Spring Kotlin code |
| `spring-kotlin-clean-architecture-convention` | Apply clean architecture conventions to Spring Kotlin projects |

## Structure

```
.claude-plugin/
  marketplace.json              ← marketplace catalog
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json               ← plugin manifest
    skills/
      <skill-name>/
        SKILL.md                ← skill prompt
```

## Contributing

Add a plugin: create a directory under `plugins/` following the structure above, then add an entry to `.claude-plugin/marketplace.json`.
