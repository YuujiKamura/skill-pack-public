---
name: qa-output-integrity-checks
description: "Testing & QA. (Completeness and schema validity are QA outcomes, not formatting niceties) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Completeness and schema validity are QA outcomes, not formatting niceties

A truncated JSON result is a QA failure even if the content direction is good. In extraction tasks, structural completeness and strict field conformance are part of correctness criteria.

### Steps

1. Fail outputs that are structurally incomplete (e.g., truncation) regardless of partial quality of insights.
2. Validate that each required field is present and semantically aligned with the stated extraction criteria.
3. Treat format integrity as a release gate for analysis artifacts.

### Examples

```bash
jq empty extracted.json
```



