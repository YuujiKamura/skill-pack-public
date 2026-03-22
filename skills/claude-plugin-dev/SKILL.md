---
name: claude-plugin-dev
description: "CLI & Tooling. (Cross-Platform Polyglot Hook Implementation) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Cross-Platform Polyglot Hook Implementation

Claude Code plugins requiring execution across Windows and Unix-like systems should use a polyglot .cmd wrapper for hooks. This avoids maintaining separate scripts for different shells and ensures consistent behavior by searching for a compatible bash environment (like Git Bash) on Windows environments.

### Steps

1. Implement a run-hook.cmd using a hybrid syntax compatible with both Windows CMD and Bash.
2. Programmatically locate the preferred bash executable in the environment PATH or common locations.
3. Execute the target hook script (e.g., session-end) through the discovered bash interpreter to maintain environment parity.



