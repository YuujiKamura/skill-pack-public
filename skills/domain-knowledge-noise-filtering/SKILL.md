---
name: domain-knowledge-noise-filtering
description: "AI & Machine Learning. (Agentic Noise Filtering in Knowledge Extraction) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Agentic Noise Filtering in Knowledge Extraction

Distinguishing between generic developer workflows (git, build cycles) and actual domain knowledge using explicit negative constraints and frequency-based hard filters.

### Steps

1. Define explicit exclusion lists in prompts for 'meta-agent' operations like sub-LLM validation loops and version control
2. Implement a hard frequency filter (e.g., frequency >= 2) to eliminate one-off anomalies from being identified as persistent patterns
3. Direct the extractor to focus on 'experienced practitioner' criteria such as specific interpretation heuristics



