---
name: ai-skill-consolidation
description: "CLI & Tooling. (Union-Find Knowledge Grouping) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Union-Find Knowledge Grouping

Fragmented knowledge extraction (e.g., separate files for IME, TSF, and WinUI3) leads to 'knowledge rot'. Merging these requires a graph-based Union-Find approach to link fragments by shared prefixes and cross-references.

### Steps

1. Identify 'fragmented derivation' where sub-topics are split across files but share underlying API or domain context.
2. Use a Union-Find algorithm to cluster skills based on description similarity and cross-reference graphs rather than simple keyword matching.
3. Implement a 'Consolidated' state in the lifecycle to track merged entities and prevent duplicate mining of the same source fragments.

### Examples

```rust
enum DraftStatus {
 Pending,
 Consolidated,
 Archived
}
```

```rust
// Grouping by shared prefix and domain similarity
let groups = consolidator.group_by_union_find(skills);
```



