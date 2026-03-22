---
name: rust-incremental-data-mining
description: "Testing & QA. (Incremental Time-Window Mining Validation) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Incremental Time-Window Mining Validation

Verification of temporal data traversal robustness by checking sliding window math and multi-factor stopping conditions. This ensures reliable data extraction from logs without infinite loops or data gaps.

### Steps

1. Validate mathematical correctness of temporal offsets (e.g., cursor_hours, window_hours) to ensure seamless window transitions.
2. Verify robust stopping conditions: handle empty results, maximum recursion depth, and maximum window count limits.
3. Check that existing data entries correctly merge with new findings without losing metadata like status or activity counts.



