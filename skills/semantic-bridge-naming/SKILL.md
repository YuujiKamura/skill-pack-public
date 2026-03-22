---
name: semantic-bridge-naming
description: "AI & Machine Learning. (Semantic Bridge Naming for AI Tooling) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Semantic Bridge Naming for AI Tooling

Naming conventions for AI skills that facilitate terminal-agent interaction must explicitly name the 'bridge' (the tool used) and the 'actor' (the agent) to avoid discovery friction and abstraction noise.

### Steps

1. Avoid abstract names like 'worker' or 'agent-relay' which hide the specific platform being targeted; naming should reflect 'What you cross (bridge) and what you do'.
2. Eliminate unnecessary abstraction layers (e.g., a generic routing skill) if there are only a few implementations, as they increase decision latency and mis-routing risk for the agent.
3. Match skill names to repository names or project identifiers (e.g., `windows-terminal-agent-relay`) to ensure consistency between the codebase and the agent's mental model.

### Examples

```
C:\Users\yuuji\.codex\skills\windows-terminal-agent-relay
```



