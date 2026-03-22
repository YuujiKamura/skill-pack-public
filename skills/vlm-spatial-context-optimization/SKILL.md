---
name: vlm-spatial-context-optimization
description: "CLI & Tooling. (Spatial Context via Contact Sheets for VLMs) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Spatial Context via Contact Sheets for VLMs

Improving VLM accuracy for cross-file queries by switching from sequential processing to side-by-side spatial comparisons.

### Steps

1. Individual-file analysis often yields low accuracy (approx. 10%) because the model lacks the context to compare files effectively across turns.
2. Providing spatial context through contact sheets (grids of thumbnails in one image) focuses model attention and raises accuracy significantly (up to 75%).
3. Implementing an ensemble approach (3 passes + majority vote) combined with spatial context is required to reach high-confidence results (90%+) in domain-specific mining.

### Examples

```
// Logic update in prompts/extract.txt to favor spatial context over sequential files
```



