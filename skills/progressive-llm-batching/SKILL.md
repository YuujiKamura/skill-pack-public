---
name: progressive-llm-batching
description: "CLI & Tooling. (Progressive Batching for Large History Mining) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Progressive Batching for Large History Mining

When processing large volumes of historical data (e.g., chat logs) for LLM analysis, fixed-range requests often hit token limits or cause memory pressure. A progressive 'window-based' approach that moves backwards from the present and stops at the first known 'processed' ID ensures structural reliability and prevents structural pipeline failures.

### Steps

1. Define incremental time windows (e.g., last 12h, then previous 24h increments).
2. In each window, filter out already processed conversation IDs using a manifest-based exclusion list.
3. Merge results into the global manifest and terminate once a window contains zero new/unprocessed items.



