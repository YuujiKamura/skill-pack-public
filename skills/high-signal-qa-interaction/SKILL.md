---
name: high-signal-qa-interaction
description: "Testing & QA. (Zero-Verbosity Protocol for Log Purity) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Zero-Verbosity Protocol for Log Purity

Enforcing a minimalist interaction style during QA execution to ensure that agent output consists only of functional results, preventing log pollution in automated test environments.

### Steps

1. Strict 'no explanation' mandates (説明不要) are used by practitioners to maintain a high signal-to-noise ratio in execution logs, making them deterministic and easier to parse for automated pass/fail verification.
2. Repeating constraints in every turn is a deliberate correction strategy to suppress the AI's default 'helpfulness' layer, forcing it to prioritize the command payload over conversational filler.
3. Conversational noise in QA logs can break downstream regex parsers or log analyzers; suppressing it is a judgment call that prioritizes system integration over human-friendly interaction.

### Examples

```
U: ... を実行しろ。説明は禁止。
```

```
U: ... だけを書き込め。説明不要。
```



