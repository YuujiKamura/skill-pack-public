---
name: claude-plugin-development
description: "CLI & Tooling. (marketplace.json Source Format Compliance) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. marketplace.json Source Format Compliance

Claude Code plugin manifests require strict adherence to the specific 'source' field schema in marketplace.json to ensure correct discovery and platform compatibility.

### Steps

1. Validate the 'source' field format against platform-specific URI or path resolution requirements.
2. Ensure the manifest points to the correct entry point recognized by the plugin runtime.



