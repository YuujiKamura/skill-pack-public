---
name: heuristic-scoring-validation
description: "Testing & QA. (Activity-Based Heuristic Scoring Verification) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Activity-Based Heuristic Scoring Verification

Testing scoring engines that weight frequency, freshness, and productivity. Experienced practitioners focus on 'dormancy penalties' for inactive items and verifying 'productivity' through look-ahead analysis of subsequent interactions.

### Steps

1. Verify dormancy penalty math: ensure multipliers are correctly applied based on the age of the last activity (e.g., >14 days).
2. Validate productivity detection: check if the subsequent assistant message successfully utilized the extracted item (e.g., tool usage detection).
3. Ensure multi-factor weightings (e.g., 60% frequency / 40% pattern match) are calculated accurately without rounding errors.



