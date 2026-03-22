---
name: ai-centric-cli-ux
description: "CLI & Tooling. (Persistent CLI Design for AI Workflows) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Persistent CLI Design for AI Workflows

Design criteria for CLI tools that are frequently operated by or reviewed by LLMs to prevent data loss and facilitate iterative analysis.

### Steps

1. Stdout is transient and easily lost in long chat contexts; CLI tools should implement a 'dump-to-file' pattern by default for long-form reports.
2. Route automated dumps to a structured temporary directory (e.g., `$TEMP/tool-name/`) to allow the AI to cross-reference previous outputs in subsequent turns.
3. Decouple data formatting logic from printing logic to allow the same data to be simultaneously emitted to the console and a markdown/JSON file for persistence.
4. Implement a `--no-dump` flag for users who want to opt-out, but keep persistence as the default to support the AI's 'lack of memory' for transient terminal output.

### Examples

```rust
// Example of routing output to a versioned temp file
let dump_path = std::env::temp_dir().join("skill-miner").join(format!("{}.md", timestamp));
std::fs::write(&dump_path, formatted_output)?;
```



