---
name: ai-workflow-infrastructure-first
description: "CLI & Tooling. (Version Control as AI Safety Infrastructure) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Version Control as AI Safety Infrastructure

A judgment criterion that prioritizes repository state over code generation to prevent 'disintegration' during complex refactoring.

### Steps

1. AI agents lack the 'fear of losing code' inherent to humans; they will often begin editing files before establishing a recovery point (Git).
2. Mandating `git init` or a private repo setup is the highest priority 'safety' step before any multi-file architectural change.
3. Complex tasks (like building a WinMD parser) should be explicitly delegated to a 'team' of sub-agents with specialized roles (bugs, features, review) to maintain structural integrity.
4. Immediate coding without a versioning strategy is a failure mode; the AI must be reprimanded if it prioritizes 'file writes' over 'state management'.

### Examples

```bash
# Critical first step before AI starts refactoring
git init && git add . && git commit -m "Initial state before AI refactor"
```



