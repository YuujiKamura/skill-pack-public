---
name: knowledge-utility-scoring
description: "CLI & Tooling. (Skill Dormancy and Productivity Scoring) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Skill Dormancy and Productivity Scoring

Ranking extracted knowledge based on its actual utility and recency in live coding sessions.

### Steps

1. Apply a dormancy penalty to skills that haven't been 'fired' (invoked) recently: use a factor of 0.5 if inactive for >7 days and 0.2 if >14 days.
2. Determine 'Productive Rate' by checking if the assistant message immediately following a skill invocation contains a tool use (indicating the skill successfully guided an action).
3. Filter out 'noise' patterns by setting a minimum score threshold (e.g., 0.05) during the consolidation phase.
4. Include `fire_count` and `last_fired_at` metadata in the exported skill bundles to preserve the provenance and reliability of the knowledge.

### Examples

```rust
// Scoring formula: (0.6*fire_score + 0.4*pattern_score) * (0.5 + 0.5*productive_rate)
let score = (0.6 * fs + 0.4 * ps) * (0.5 + 0.5 * pr);
if score < threshold { continue; }
```



