---
name: chat-history-source-abstraction
description: "CLI & Tooling. (Abstract conversation discovery by carrier-specific strategy) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Abstract conversation discovery by carrier-specific strategy

When mining conversations across Codex/Gemini, path conventions differ. Carrier-agnostic analysis requires isolating discovery logic behind a strategy boundary instead of hardcoding one provider’s filesystem layout.

### Steps

1. Separate 'where chats live' from 'how chats are analyzed' to avoid provider lock-in in mining tools.
2. Model provider selection explicitly (enum/strategy) so adding a new carrier is configuration, not refactoring.
3. Treat filesystem patterns (e.g., Gemini temp chat glob) as adapter-level knowledge, not core domain logic.

### Examples

```rust
enum HistorySource { Codex, Gemini }
trait DiscoveryStrategy { fn discover(&self) -> Vec<std::path::PathBuf>; }
```

```text
~/.gemini/tmp/**/chats/*.json
```



