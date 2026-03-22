---
name: hybrid-agent-governance
description: "Documentation. (Balancing General Plugins with Machine-Specific Rules) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. Balancing General Plugins with Machine-Specific Rules

Distinguishing when to use general-purpose plugins versus keeping machine-specific constraints in local configuration files like CLAUDE.md.

### Steps

1. Plugins like 'superpowers' or 'code-review' provide general process guidelines but cannot replace CLAUDE.md for enforcing rules that depend on local filesystem paths or machine-specific tool flags.
2. A rule is not obsolete just because a plugin covers its high-level process (e.g., code review) if the local rule covers the implementation details (e.g., the exact path to the review binary on the current OS).

### Examples

```
// Plugin: Handle general review workflow
// CLAUDE.md: Run C:\tools\local-check.exe --strict-mode
```



