---
name: backward-compatible-data-mining
description: "AI & Machine Learning. (Robust Schema Evolution for Mined Data) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Robust Schema Evolution for Mined Data

When evolving schemas for data mining (e.g., adding discovery IDs to manifests), backward compatibility must be maintained to prevent logic failures when processing legacy data stores.

### Steps

1. New fields in a manifest (like 'mined_ids') must be handled with default values to ensure that legacy files without these fields still parse correctly.
2. Combining default attributes with conditional serialization (e.g., skip_serializing_if) prevents breaking old parsers while keeping the new output clean of empty, redundant fields.
3. The 'why' behind this design is to ensure that the ingestion pipeline remains resilient as the underlying mining algorithms evolve and introduce new metadata.

### Examples

```rust
#[serde(default, skip_serializing_if = "Vec::is_empty")]
pub mined_ids: Vec<String>,
```



