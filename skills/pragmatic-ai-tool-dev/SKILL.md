---
name: pragmatic-ai-tool-dev
description: "AI & Machine Learning. (Simplicity in Safety-Critical Refactoring) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Simplicity in Safety-Critical Refactoring

Choosing documentation and logic updates over complex architectural changes for immediate safety improvements.

### Steps

1. When an AI agent makes a dangerous error, the impulse to perform a deep architectural refactor (e.g., crate separation) can introduce more risk than it resolves. A pragmatic fix—improving default flags and tool descriptions—often achieves the same safety goal with less overhead.
2. For AI-facing tools, the most effective 'safety' changes often happen at the interface level (how the agent understands the tool) rather than the internal structural level.



