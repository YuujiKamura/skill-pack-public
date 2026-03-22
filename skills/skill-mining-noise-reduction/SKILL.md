---
name: skill-mining-noise-reduction
description: "AI & Machine Learning. (Instruction-Noise Filtering in Skill Mining) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Instruction-Noise Filtering in Skill Mining

Excluding operational noise from project-specific instructions (e.g., CLAUDE.md) and generic dev loops to isolate true domain knowledge.

### Steps

1. Add explicit negative constraints to the extraction prompt to exclude sub-agent verification loops (e.g., `claude -p`) and generic build/test cycles.
2. Implement a hard frequency filter in the code (e.g., `frequency >= 2`) to discard accidental one-off occurrences that are not true patterns.
3. Filter out common CLI operations like `git status`, `cargo test`, and `npm test` from the pattern candidates to prevent infrastructure noise from being labeled as domain knowledge.

### Examples

```text
除外すべきもの（ドメイン固有ではない）:
- サブLLMによるコード検証ループ（claude -p で別AIを呼んで差分検証する手順）
- cargo build / cargo test / cargo check などのビルド・テストサイクル
- git status / git diff / git commit などのバージョン管理操作
```



