---
name: autonomous-context-recovery
description: "AI & Machine Learning. (Search-Based Context Recovery Strategies) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Search-Based Context Recovery Strategies

Truly autonomous agents should attempt to 'unblock' themselves by discovering missing context within their permitted environment before requiring human intervention.

### Steps

1. Use discovery tools (grep, glob, list_directory) to map out non-obvious project structures that might contain missing logs or data.
2. Synthesize fragments from multiple sources (prompts, debug logs, scripts) to reconstruct the original user intent.
3. Key insight: Proactive discovery is significantly more efficient than asking the user for data that is already present in the system environment.

### Examples

```bash
glob --pattern "C:\Users\yuuji\.gemini\tmp\**\*.json"
```



