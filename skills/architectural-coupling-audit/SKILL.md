---
name: architectural-coupling-audit
description: "Testing & QA. (Strength-Distance-Variability Review Rubric) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Strength-Distance-Variability Review Rubric

A specific design audit framework used to evaluate module health. It quantifies quality by analyzing the relationship between dependency strength, the 'distance' between modules, and code volatility.

### Steps

1. Favor 'High Cohesion' by keeping strong bindings (direct dependencies) strictly within close proximity (internal module scope).
2. Enforce 'Loose Coupling' across far distances (module boundaries) by using only stable, contract-based public APIs.
3. Flag 'Global Complexity' anti-patterns where distant modules share strong bindings, such as direct internal implementation references.
4. Require stricter isolation for 'High Variability' (volatile core logic) while allowing more relaxed coupling for stable boilerplate.



