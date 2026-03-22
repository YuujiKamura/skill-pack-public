---
name: cli-workflow-design
description: "CLI & Tooling. (Decoupling CLI Tools from Ephemeral File Structures) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Decoupling CLI Tools from Ephemeral File Structures

Avoid designing CLI workflows that depend on intermediate directory structures (side-effects) created by previous commands. Passing complete state via data structures (like JSON containing absolute paths) allows commands to be re-run or tested in isolation without reconstructing the entire temporary environment.

### Steps

1. Identify intermediate artifacts (like temporary 'P-folders') that serve as visual side-effects rather than essential data.
2. Enable direct data injection via metadata files that contain all necessary file references and full paths.
3. Make file-system-based structure discovery an optional fallback for convenience rather than a requirement for core logic.



