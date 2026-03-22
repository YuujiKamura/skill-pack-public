---
name: ai-tool-quota-navigation
description: "CLI & Tooling. (Navigating Quota-Locked AI CLI Metadata) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Navigating Quota-Locked AI CLI Metadata

Determining AI tool usage and status when the primary binary is locked due to quota limits requires bypassing standard CLI commands in favor of direct cache inspection.

### Steps

1. Distinguish between a 'command not found' error and a 'quota exceeded' message; the latter confirms the command's validity but prevents its execution.
2. Inspect local JSON cache files (e.g., ~/.claude/usage.json or ~/.claude/stats-cache.json) to retrieve usage data and reset times when the CLI binary refuses to run.
3. Identify slash-commands (e.g., /usage) intended for the interactive shell as distinct from OS-level subcommands to avoid unnecessary search loops.

### Examples

```
cat ~/.claude/usage.json
```

```
claude usage 2>&1 | grep -i 'limit'
```



