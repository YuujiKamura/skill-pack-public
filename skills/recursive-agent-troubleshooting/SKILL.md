---
name: recursive-agent-troubleshooting
description: "AI & Machine Learning. (Meta-Recursive Troubleshooting in Agentic Sessions) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Meta-Recursive Troubleshooting in Agentic Sessions

In complex multi-session environments, agents should recognize when they are in a 'mirror' or recursive scenario (e.g., analyzing a log of an identical failed attempt) and use previous session data as diagnostic input.

### Steps

1. Identify when current system logs or workspace files describe the exact task or failure state currently being experienced.
2. Analyze previous agent thought processes (found in session JSONs) to avoid repeating unsuccessful search or discovery patterns.
3. Leverage hidden or non-standard directory structures (e.g., .gemini/tmp) as a unified knowledge base for autonomous recovery.

### Examples

```powershell
ls C:\Users\yuuji\AppData\Local\Temp\.cli-ai-analyzer-*
```



