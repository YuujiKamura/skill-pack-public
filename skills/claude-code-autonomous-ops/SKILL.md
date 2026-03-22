---
name: claude-code-autonomous-ops
description: "Documentation. (Autonomous Decision Loop via Sub-LLM) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. Autonomous Decision Loop via Sub-LLM

Overriding Claude's default 'ask for help' behavior by implementing an autonomous consultation loop using a secondary LLM process to resolve ambiguity without user interruption.

### Steps

1. Explicitly forbid interruptions in `CLAUDE.md` for non-critical bugs or minor decisions to override the default system prompt behavior.
2. Execute `claude -p "<query>" --max-turns 1` via the terminal to consult a sub-LLM for a 'second opinion' when faced with architectural or logic choices.
3. Provide sufficient background context (current file state, goals) to the sub-LLM call to ensure the decision is grounded in the project's specific state.

### Examples

```
claude -p "質問内容" --max-turns 1
```

```
- ユーザーに質問して作業を中断するな。自分で判断して進めろ
- 判断に迷ったら `claude -p "質問内容" --max-turns 1` をBashで呼んでサブLLMに相談しろ
```



