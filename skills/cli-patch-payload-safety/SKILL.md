---
name: cli-patch-payload-safety
description: "CLI & Tooling. (Escaped Payload Handling in Patches) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Escaped Payload Handling in Patches

Automated patch tools often fail or mangle code when the content contains heavy escaping, backslashes, or base64 data common in IPC protocols.

### Steps

1. The 'apply_patch' tool is fragile when payloads contain protocol-specific escaping (like Ghostty/IPC payloads). The CLI's quoting layer can 'swallow' backslashes or interpret quotes inside the patch body, leading to corrupted implementations.
2. When the payload is quoting-heavy, practitioners favor manual file-range edits or raw-string heredocs over diff/patch tools to guarantee code integrity.

### Examples

```bash
# Avoid patches for this; use manual replacement
# "INPUT": "SGVsbG8gV29ybGQ=\n"
```



