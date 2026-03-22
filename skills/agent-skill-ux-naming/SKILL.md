---
name: agent-skill-ux-naming
description: "AI & Machine Learning. (Intent-Based Naming for Boundary Crossings) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Intent-Based Naming for Boundary Crossings

Skill names for AI-to-AI or AI-to-Terminal relays must prioritize the 'crossing' action and the 'boundary' over internal implementation terms like 'worker'.

### Steps

1. Avoid internal-facing labels like 'worker' in skill names; they are too generic and fail to signal the 'handoff' or 'delegation' nature of the operation to the AI.
2. Use 'relay' or 'bridge' to explicitly indicate the movement of context across boundaries (e.g., moving from a local session into a persistent terminal-hosted agent).
3. Include the transport layer and the acting entity in the name (e.g., 'windows-terminal-agent-relay'). This ensures the skill is discoverable when the user mentions the environment and distinguishes it from standard CLI automation scripts.

### Examples

```
// Renaming based on user correction to match repository and intent
// From: wt-ai-worker
// To: windows-terminal-agent-relay
```



