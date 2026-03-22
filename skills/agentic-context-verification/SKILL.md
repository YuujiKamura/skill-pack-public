---
name: agentic-context-verification
description: "AI & Machine Learning. (Proactive Context Gap Identification) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Proactive Context Gap Identification

Experienced AI agents must verify the presence and sufficiency of source data before beginning complex analytical tasks. This 'look-before-you-leap' principle prevents hallucinations and ensures the high-fidelity extraction required for structured knowledge bases.

### Steps

1. Explicitly validate the input state against the user's high-level goal (e.g., matching a domain like 'AI & ML' to the actual content of the provided summaries).
2. Detect 'context gaps' where the prompt requests extraction from a source that is absent from the provided text or workspace.
3. Pause execution to alert the user of missing data rather than attempting to fulfill the request with generic information or instruction examples.

### Examples

```json
// Example of a 'Failure Summary' identifying missing context
{
 "status": "on hold",
 "reason": "user omitted the actual conversation logs required for extraction"
}
```



