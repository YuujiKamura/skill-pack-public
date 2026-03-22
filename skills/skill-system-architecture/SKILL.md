---
name: skill-system-architecture
description: "Documentation. (Skill Lifecycle and Overlap Resolution) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. Skill Lifecycle and Overlap Resolution

Understanding the functional boundaries between proactive skill creation and historical knowledge mining from chat history.

### Steps

1. Skill 'Creator' (template-based definition) and 'Miner' (history-based extraction) are distinct lifecycle stages; mining identifies emergent patterns that users haven't explicitly documented yet.
2. Custom skills are isolated from core agent logic and typically reside in local directories (e.g., ~/.claude/skills/), necessitating an explicit discovery or 'loading' step if the agent is unaware of them.

### Examples

```
ls ~/.claude/skills/skill-management/SKILL.md
```

```
~/.claude/skills/skill-miner/SKILL.md
```



