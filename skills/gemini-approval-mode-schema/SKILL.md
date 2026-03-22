---
name: gemini-approval-mode-schema
description: "CLI & Tooling. (Approval mode is schema-constrained, not free-form) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Approval mode is schema-constrained, not free-form

Gemini settings validation rejected `general.defaultApprovalMode: "yolo"`; only enumerated values are accepted. In no-prompt workflows, correctness depends on using valid enum values that match automation intent.

### Steps

1. Treat approval mode as a strict schema field; invalid labels fail fast and block automation.
2. Map 'no manual approvals' requirements onto supported enum values, not invented modes.
3. Validate config parse errors first before debugging higher-level tool behavior.

### Examples

```json
{
 "general": {
 "defaultApprovalMode": "auto_edit"
 }
}
```



