---
name: cli-history-interop
description: "CLI & Tooling. (Heterogeneous CLI History Abstraction) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Heterogeneous CLI History Abstraction

Architecting history analysis tools to handle diverse storage formats and transient directory structures across different CLI providers.

### Steps

1. Hardcoding file paths for conversation mining (e.g., .jsonl for Claude) leads to brittle tools. Different providers use transient temp directories (Gemini) versus project-local persistence (Claude).
2. Introduce a HistorySource enum and DiscoveryStrategy trait to decouple the mining logic from the carrier format. This architectural shift allows a single 'skill-miner' to ingest knowledge from both persistent project logs and ephemeral system temp files.

### Examples

```
enum HistorySource { Gemini, ClaudeCode }
```

```
trait DiscoveryStrategy { fn find_chats(&self) -> Vec<PathBuf>; }
```



