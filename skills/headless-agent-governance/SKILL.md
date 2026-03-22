---
name: headless-agent-governance
description: "CLI & Tooling. (Strict Enum-Based Approval Configuration) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Strict Enum-Based Approval Configuration

Achieving fully autonomous, headless agent workflows by navigating strict configuration schemas.

### Steps

1. Generic 'yolo' or 'force' flags for auto-approval often fail due to strict enum validation in modern CLI schemas. Attempting invalid configuration values can crash the agent before it even starts.
2. To achieve a truly non-interactive workflow, precisely map valid enum variants (e.g., 'auto_edit' or 'plan') via built-in help tools (cli_help) rather than guessing. Success criteria: Headless execution requires matching the exact governance flags expected by the CLI's security layer.

### Examples

```
gemini config set general.defaultApprovalMode auto_edit
```

```
Invalid enum value. Expected 'default' | 'auto_edit' | 'plan'
```



