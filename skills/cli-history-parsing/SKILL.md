---
name: cli-history-parsing
description: "CLI & Tooling. (Progressive Time-Window Mining) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Progressive Time-Window Mining

Efficiently processing massive CLI chat histories by iterating through expanding time windows and maintaining processing state to avoid token limits.

### Steps

1. Maintain a `mined_ids` HashSet in the tool's manifest using `#[serde(skip_serializing_if = "HashSet::is_empty")]` to skip already-processed conversation blocks.
2. Implement a recursive mining loop that searches backward in expanding windows (e.g., 12h, 24h, 48h) starting from the current time.
3. Terminate the mining process when an empty time window is encountered or when the `max_windows` limit is reached to ensure predictable execution time.
4. Store time-based cursors in hours to handle the fragmentation of conversation files across project directories.

### Examples

```rust
#[derive(Debug, Serialize, Deserialize)]
pub struct Manifest {
 #[serde(default, skip_serializing_if = "HashSet::is_empty")]
 pub mined_ids: HashSet<String>,
}
```

```bash
cargo run -- mine --max-days 7 --max-windows 10 --dir ~/.claude/projects
```



