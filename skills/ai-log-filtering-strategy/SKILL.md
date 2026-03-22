---
name: ai-log-filtering-strategy
description: "AI & Machine Learning. (Strategic Pre-Summarization Filtering) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Strategic Pre-Summarization Filtering

Filtering large activity streams before AI processing minimizes token usage and improves summary relevance by eliminating noise that would otherwise distract the LLM.

### Steps

1. Apply multi-dimensional filters (date range, project context, and search keywords) directly on the raw data stream (e.g., JSONL history) prior to batching for the AI.
2. Decouple 'Raw' log viewing from 'Summary' generation logic; providing a direct stream for raw data avoids unnecessary LLM overhead when a user only needs to find a specific entry.
3. Filter by project identifiers to maintain context purity, preventing the AI from conflating distinct work domains into a single summary.



