---
name: rust-refactor-integrity
description: "Testing & QA. (Rust Structural Refactoring Integrity) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Rust Structural Refactoring Integrity

Ensuring structural changes in Rust projects don't introduce circular dependencies or break public APIs. Practitioners use delegation wrappers to maintain backward compatibility when moving logic between modules.

### Steps

1. Check for circular dependencies introduced by shifting logic between internal modules.
2. Verify public API stability by ensuring legacy entry points remain as delegation wrappers to new function locations.
3. Confirm that refactored code maintains trait implementation consistency across module boundaries.



