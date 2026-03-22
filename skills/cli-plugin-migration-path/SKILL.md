---
name: cli-plugin-migration-path
description: "CLI & Tooling. (Hook-Based vs. MCP Plugin Architecture) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Hook-Based vs. MCP Plugin Architecture

Choosing lifecycle hooks over full MCP servers for initial CLI-to-plugin ports to avoid transport-layer complexity.

### Steps

1. Building a full MCP Server in Rust for local plugins introduces significant overhead due to complex stdio transport requirements.
2. Session-end hooks (SessionEnd) are a superior 'v1' choice for background automation as they don't require rewriting core CLI logic.
3. Polyglot wrappers (e.g., .cmd scripts for Windows/Linux compatibility) ensure that background mining tools execute reliably regardless of the host shell.

### Examples

```json
{
 "hook": "SessionEnd",
 "script": "run-hook.cmd"
}
```



