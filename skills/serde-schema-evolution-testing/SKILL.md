---
name: serde-schema-evolution-testing
description: "Testing & QA. (Backward-Compatible Data Roundtrip Testing) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Backward-Compatible Data Roundtrip Testing

Ensuring persistent data formats (TOML/JSON) evolve without breaking existing installations. Focuses on safe schema growth using Serde attributes and verification through serialization roundtrips.

### Steps

1. Verify the use of `#[serde(default)]` and `#[serde(skip_serializing_if = "...")]` for new fields to prevent breaking old data files.
2. Perform TOML/JSON roundtrip tests: ensure reading old formats and writing them back is deterministic and non-destructive.
3. Check that optional metadata fields are omitted from serialization when empty to maintain clean and minimal data files.



