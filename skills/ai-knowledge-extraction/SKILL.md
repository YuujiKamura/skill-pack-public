---
name: ai-knowledge-extraction
description: "CLI & Tooling. (Noise-Resilient Skill Mining) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Noise-Resilient Skill Mining

Filtering ephemeral agent instructions and system-level reminders to extract stable, high-value patterns from LLM conversation logs.

### Steps

1. Strip content within `<system-reminder>` tags or text matching `CLAUDE.md` patterns from the conversation context before LLM processing to prevent 'rule contamination' in extracted skills.
2. Implement topic-based naming by requiring the LLM to generate a `skill_slug` (e.g., `claude-plugin-dev`) instead of using fixed, coarse-grained domain categories.
3. Capture the immediate user message preceding a tool call as 'trigger context' to identify the specific intent or problem that activated the knowledge pattern.
4. Differentiate between 'patterns' (the actual solution) and 'source' (the verbatim tool usage) to maintain clean documentation while preserving technical details.

### Examples

```rust
// Example of stripping system reminders from context
let content = strip_system_reminders(&msg.content);
```

```json
[
 {
 "skill_slug": "kebab-case-topic-name",
 "title": "Pattern name",
 "description": "Concrete details, not abstract summaries"
 }
]
```



