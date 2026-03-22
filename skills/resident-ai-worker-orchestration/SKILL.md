---
name: resident-ai-worker-orchestration
description: "AI & Machine Learning. (Decoupled Persistent Session Management) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Decoupled Persistent Session Management

Managing persistent AI workers across multiple backends (Claude, Gemini, Codex) requires decoupling the session runner from the agent-specific logic to allow concurrent, cross-agent workflows.

### Steps

1. Use a unified orchestrator script with parameterized inputs (-Agent, -Session) to maintain different persistent AI instances. This prevents configuration drift between specialized skills for different LLM backends.
2. Wrap the core orchestrator in agent-specific 'wrapper' skills (e.g., wt-ai-claude, wt-ai-gemini). This ensures the primary AI understands the specific constraints and capabilities of the 'worker' agent it is delegating to.
3. Session-based isolation is critical for complex multi-agent 'relays'. By using unique session IDs, a controller can orchestrate a Gemini instance for 'exploratory research' and a Claude instance for 'production code generation' simultaneously within the same terminal host.

### Examples

```
pwsh -File winui3-ai-worker.ps1 -Action start -Agent gemini -Session wt-ai-gemini
```

```
pwsh -File winui3-ai-worker.ps1 -Action send -Agent claude -Session wt-ai-claude -Text "Fix the race condition"
```



