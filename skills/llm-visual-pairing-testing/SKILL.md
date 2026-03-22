---
name: llm-visual-pairing-testing
description: "Testing & QA. (Contact Sheet Visual Matching Strategy) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Contact Sheet Visual Matching Strategy

To improve LLM accuracy in image pairing tasks (e.g., before/after construction photos), combine candidates into a single numbered grid (contact sheet) rather than uploading dozens of individual files to avoid attention dispersion. Query matches one-by-one rather than bulk-matching to prevent performance degradation.

### Steps

1. Generate numbered grid images (contact sheets) for candidate sets using image processing libraries.
2. Exclude textual metadata like file names or site IDs during the matching phase as they can distract the visual reasoning model.
3. Execute targeted one-to-many queries (e.g., 'Which number in the grid matches this specific reference image?') instead of 'match all' prompts.
4. Verify matching based on spatial cues like vanishing points, silhouettes, and lane markings.



