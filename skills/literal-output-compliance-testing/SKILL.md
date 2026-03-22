---
name: literal-output-compliance-testing
description: "AI & Machine Learning. (Literal-output obedience as a hard gate) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Literal-output obedience as a hard gate

For instruction-following evaluation, exact-string tasks (e.g., return one token only) are a strong sanity check. In this domain, passing this test proves decoding/instruction control at the token level before attempting richer tasks.

### Steps

1. Use exact-match prompts as a binary gate: either the model returns the precise required string or it fails instruction obedience.
2. Treat this as necessary but not sufficient; a model can pass literal-output tests and still fail structured reasoning tasks.

### Examples

```text
Return exactly: GEMINI_OK
```

```text
GEMINI_OK
```



