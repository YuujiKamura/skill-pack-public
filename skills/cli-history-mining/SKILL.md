---
name: cli-history-mining
description: "Miscellaneous. (Dual-Mode CLI History Reporting Architecture) Use when user mentions: ."
---

# Miscellaneous

Patterns: 1

## 1. Dual-Mode CLI History Reporting Architecture

A design pattern for command history tools that balances precise data verification ('Raw') with AI-powered insight generation ('Summary').

### Steps

1. Maintaining a 'Raw' output format is a critical design principle for AI-first CLI tools. It serves as a 'source of truth' that allows developers to verify AI interpretations and debug filtering logic, preventing user distrust of the summarized output.
2. Processing activity logs from line-delimited formats like `.jsonl` (e.g., `~/.claude/history.jsonl`) is essential for performance, enabling streaming or incremental parsing to handle files that grow indefinitely without exhausting memory.
3. Early metadata-driven filtering (by 'days' or 'project_id') at the data ingestion layer is a mandatory optimization to minimize token costs and prevent context-window overflow during the AI summarization phase.

### Examples

```bash
# Use Raw for debugging filtering logic
skill-miner today --format raw --days 1
# Use Summary for high-level reporting
skill-miner today --format summary --granularity coarse
```



