---
name: llm-log-mining-budgeting
description: "AI & Machine Learning. (LLM Context Budgeting for Log Mining) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. LLM Context Budgeting for Log Mining

Increasing context truncation limits to capture high-fidelity patterns (code/JSON) in long-running AI conversations while staying within model safety margins.

### Steps

1. Set user message truncation to 2000 characters and assistant message truncation to 3000 characters to prevent cutting off crucial code examples.
2. Increase the conversation turn limit (e.g., to 40 turns) to capture the final resolution which often happens at the end of a long thread.
3. Calculate total token usage against model limits (e.g., targeting ~72% of 1M tokens for Gemini Flash) to ensure safety margins for batch processing.

### Examples

```rust
// src/extractor.rs
truncate(&cleaned, 2000);
truncate(&cleaned_a, 3000);
.take(40)
```



