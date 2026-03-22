---
name: structured-output-semantic-validation
description: "AI & Machine Learning. (Schema-valid does not mean task-valid) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Schema-valid does not mean task-valid

A recurring domain pitfall is conflating structural compliance (valid JSON, allowed enums) with task success. Here, structure passed while content integrity failed, showing the need for semantic validators in AI evaluation workflows.

### Steps

1. Separate validation into two layers: structural checks (JSON/schema) and semantic checks (no fabricated items, source-grounded tags).
2. Prioritize semantic integrity when scores conflict; a perfectly formatted answer with extra fabricated entries should be marked failed.

### Examples

```json
{"index": 6, "domain": "AI & Machine Learning", "tags": ["gemini-cli"]}
```

```text
Failure reason: index 6 was never present in inputs.
```



