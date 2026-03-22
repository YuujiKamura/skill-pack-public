---
name: shell-polyglot-scripts
description: "CLI & Tooling. (Cross-Platform Polyglot .cmd Construction) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Cross-Platform Polyglot .cmd Construction

Creating CLI tools that run natively on both Windows and POSIX systems often requires polyglot .cmd files that use specific syntax tricks to remain valid in multiple shell environments simultaneously.

### Steps

1. Implement a script header that is syntactically valid in both Windows CMD and POSIX sh.
2. Use environment-specific redirection to route execution to the appropriate runtime or interpreter without requiring platform-specific wrappers.



