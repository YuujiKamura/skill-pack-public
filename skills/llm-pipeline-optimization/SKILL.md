---
name: llm-pipeline-optimization
description: "AI & Machine Learning. (Decoupling Noise Reduction from Semantic Extraction) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Decoupling Noise Reduction from Semantic Extraction

High-token summarization and judgment-heavy extraction tasks should be separated into distinct pipeline stages to prevent 'instruction dilution' and context bloat from degrading reasoning performance.

### Steps

1. Identify 'prompt bloat' where noise-reduction and formatting instructions consume a majority (e.g., >70%) of the prompt. This indicates that the reasoning model's attention is being over-leveraged on preprocessing rather than judgment.
2. Distinguish between 'high-token' tasks (processing large raw logs) and 'judgment-heavy' tasks (extracting domain insights). Processing these together often results in the 'lost in the middle' phenomenon where critical nuances are missed.
3. Resist the urge to use low-cost/small models (e.g., Gemini Flash) for the initial summarization stage if the downstream task is nuance-sensitive. High-fidelity summarization is critical for preserving the semantic signal required for complex extraction.

### Examples

```
// skill-miner/prompts/summarize.txt - Dedicated noise reduction
// skill-miner/prompts/extract.txt - Focused reasoning logic
```

```
let summarized = if !args.no_summarize {
 cli_ai_analyzer::prompt("summarize", raw_log)?
} else {
 raw_log
};
```



