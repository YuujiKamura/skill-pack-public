---
name: heuristic-to-llm-transition
description: "CLI & Tooling. (Replacing Heuristic Chains with Structured LLM Inference) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Replacing Heuristic Chains with Structured LLM Inference

Hardcoded logic (if-chains) for identifying tasks or projects in a CLI environment is fragile and fails as the ecosystem grows. Shifting classification to an LLM with structured input improves robustness.

### Steps

1. Abolish complex heuristic if-chains that try to guess project types based on path patterns; these are high-maintenance and prone to edge-case failures.
2. Design a 'Context Struct' (e.g., SlotContext) that aggregates metadata, allowing the LLM to perform high-level reasoning rather than simple pattern matching.
3. The transition from heuristics to LLM inference requires a 'Context Builder' phase where raw logs are transformed into a descriptive prompt, rather than being parsed by regex.

### Examples

```
fn build_slot_contexts(history: &Vec<Entry>) -> Vec<SlotContext> { ... }
```

```
// Replace: if path.contains("npm") { "Node" } else { ... }
```



