---
name: automated-report-layout-verification
description: "CLI & Tooling. (Iterative Excel/CSV Layout Refinement) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Iterative Excel/CSV Layout Refinement

Decision criteria for generating human-readable reports where visual layout and cell-merging logic are non-trivial.

### Steps

1. Locale-specific date formats (e.g., '◯月◯日') are often missed by default ISO-centric AI; explicit formatting instructions are required for end-user acceptance.
2. Cell merging logic in generated Excel files often fails on 'edge-case' row spans (e.g., merging 3 rows vs 2 rows); generate a sample file first for human visual verification.
3. Adjusting layout is iterative: the AI should expect to refine merging logic (e.g., moving from row-based merging to section-based merging) based on human visual feedback from the generated artifact.
4. Parsing source images (OCR/VLM) for reports requires a specific sort order (e.g., descending by timestamp) to ensure the report reflects the latest chronological data.

### Examples

```python
# Refined merging logic: merge rows 9-10 across sections, not 9-11
for col in range(1, 10):
 ws.merge_cells(start_row=9, end_row=10, start_column=col, end_column=col)
```



