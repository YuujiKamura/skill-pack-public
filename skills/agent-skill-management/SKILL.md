---
name: agent-skill-management
description: "CLI & Tooling. (Cross-Agent Skill Synchronization) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Cross-Agent Skill Synchronization

Managing domain knowledge as a decoupled, linkable ecosystem rather than project-specific configuration files.

### Steps

1. Treating skills as single-file metadata (like CLAUDE.md) fails for large-scale engineering where hundreds of specialized tools exist. Success requires linking external catalogs via the agentskills.io standard to maintain a single source of truth across different agent runtimes.
2. Manual restarts to update linked skill catalogs are inefficient and disrupt context. Use specific runtime commands like `/memory refresh` to force a sync of the knowledge index without losing the current session state.

### Examples

```
gemini skills link "C:\Users\yuuji\.claude\skills"
```

```
/memory refresh
```



