---
name: llm-context-optimization
description: "AI & Machine Learning. (Actionable Pattern Mining via High-Density Context) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Actionable Pattern Mining via High-Density Context

Increasing truncation limits to capture concrete code and schema details essential for technical pattern mining.

### Steps

1. Default truncation limits (e.g., 300-500 chars) are often too aggressive for technical domains, stripping away the code snippets and JSON schemas that provide the 'how' and 'why' of a pattern.
2. With high token headroom (e.g., <10% utilization of a 1M token window), increasing character limits (2000-3000 chars) and history depth (20 to 40 turns) transforms generic summaries into 'actionable' skills by preserving the concrete implementation details.

### Examples

```
// Strategic context expansion in src/extractor.rs:
// user_messages: 300 -> 2000 chars
// assistant_messages: 500 -> 3000 chars
// conversation_history: take(20) -> take(40)
```



