---
name: self-referential-cli-mining
description: "CLI & Tooling. (State Recovery via Tool Dogfooding) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. State Recovery via Tool Dogfooding

Using a CLI tool to analyze its own execution history (mining its own logs) is a primary strategy for recovering lost session context or generating 'meta-summaries' of the tool's own development.

### Steps

1. When a conversational context window is lost, use the tool's own mining capabilities to parse recent execution logs and regenerate a summary of progress.
2. Verify log-parsing accuracy using a '--dry-run' flag combined with a strict time filter (e.g., '--max-days 1') before committing to a full analysis or state update.
3. The tool's local output directory (e.g., ~/.claude/skills/drafts/) serves as a persistent 'long-term memory' that can be bootstrapped back into a live session.

### Examples

```
cargo run -- mine --max-days 1 --dry-run
```

```
cargo run -- today --format summary
```



