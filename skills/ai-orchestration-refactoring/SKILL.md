---
name: ai-orchestration-refactoring
description: "Miscellaneous. (Agent-Team Audit for Architectural Debt) Use when user mentions: ."
---

# Miscellaneous

Patterns: 1

## 1. Agent-Team Audit for Architectural Debt

When a single agent produces 'meaningless complexity,' the task should be refactored into a comparative audit by a team of agents to identify where the implementation has diverged from the intended blueprint.

### Steps

1. Recognize 'complex investigation output' as a sign of architectural debt or poor task scoping.
2. Deploy multiple agents in parallel to perform a gap analysis between the current state and a known-good reference (e.g., compare ghostty-win vs. Windows Terminal).
3. Use a 'team' approach specifically to cross-reference implementation details when the root cause is a deep design mismatch rather than a simple bug.

### Examples

```bash
# Instruction to pivot from individual research to team audit
"チームを組んでやれ。Windows Terminalの実装と比較して乖離を洗い出せ。"
```



