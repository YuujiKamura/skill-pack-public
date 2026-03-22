---
name: visual-ensemble-pairing
description: "AI & Machine Learning. (Visual Grid Ensemble for VLM Matching) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Visual Grid Ensemble for VLM Matching

Improving image pairing accuracy (e.g., from 10% to 90%) by presenting candidates in a spatial grid 'contact sheet' rather than individual files.

### Steps

1. Generate a single 'contact sheet' image containing a grid of labeled thumbnails (e.g., 7x3) to leverage the VLM's spatial reasoning capabilities.
2. Use a '1-shot per target' prompt strategy where the VLM is asked 'Which number in this grid matches this target image?'.
3. Perform an ensemble vote (e.g., 3 passes) and calculate confidence based on agreement (3/3 = 1.0, 2/3 = 0.67) to filter out halluncinations.

### Examples

```rust
// Grid calculation for contact sheets
let sheet_w = cols * CELL_W + (cols - 1) * GAP;
let sheet_h = rows * (CELL_H + LABEL_H) + (rows - 1) * GAP;
let x = col * (CELL_W + GAP);
let y = row * (CELL_H + LABEL_H + GAP);
```

```text
Image 1 is a BEFORE-construction road photo.
Image 2 is a numbered grid (1-21) of AFTER-construction candidates.
Which number shows the SAME road location?
```



