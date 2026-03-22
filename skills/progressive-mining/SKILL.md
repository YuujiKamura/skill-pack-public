---
name: progressive-mining
description: "AI & Machine Learning. (Progressive Temporal Window Mining) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Progressive Temporal Window Mining

Handling large-scale log analysis by incrementally stepping backwards in time and stopping at previously mined boundaries.

### Steps

1. Divide the total processing timeline into expanding windows (e.g., first 12h, then 24h increments) to keep token counts within batch limits.
2. Maintain a `mined_ids` HashSet in the manifest to track processed conversations across different mining sessions.
3. Terminate the pipeline early once a window contains zero new conversations, indicating the miner has reached already processed history.

### Examples

```rust
// Incremental window progression
let window_hours = [12, 24, 24, 24, ...];
pub mined_ids: HashSet<String>,
#[serde(default, skip_serializing_if = "HashSet::is_empty")]
```



