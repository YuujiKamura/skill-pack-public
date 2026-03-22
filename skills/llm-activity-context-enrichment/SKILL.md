---
name: llm-activity-context-enrichment
description: "CLI & Tooling. (Context Enrichment for CLI Activity Classification) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Context Enrichment for CLI Activity Classification

LLMs often misclassify project boundaries and task intent in CLI logs because command strings or installation paths (like global npm/cargo directories) mask the actual source root. Providing explicit structural context is required for accurate summarization.

### Steps

1. Command display strings (e.g., HistoryEntry.display) are often deceptive; they may reflect the location of the interpreter or global binary rather than the project being worked on.
2. Individual command snippets lack the 'spatial context' of a project. To fix mislabeling, anchor the LLM with the current working directory (CWD) and the 'top N touched files' during that time slot.
3. Include intent-based anchors by passing the first 500 characters of the user/AI interaction, which provides the 'why' that raw shell commands often obscure.

### Examples

```
struct SlotContext { cwd: PathBuf, first_msg_snippet: String, touched_files: Vec<PathBuf> }
```

```
// Instead of just 'cargo run', provide the context of where it ran and what files changed.
```



