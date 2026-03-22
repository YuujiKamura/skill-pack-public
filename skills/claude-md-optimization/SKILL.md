---
name: claude-md-optimization
description: "Documentation. (Context-Efficient Local Instruction Engineering) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. Context-Efficient Local Instruction Engineering

Trimming CLAUDE.md to a minimal set of mandatory rules to maximize context window efficiency while maintaining strict control over agent behavior.

### Steps

1. Instructional density is more critical than breadth; reducing CLAUDE.md to a minimal set (~20 lines) of high-impact rules (e.g., Interrupt prohibition, Plan-to-Team execution, Verification) prevents context dilution and improves adherence.
2. The 'Verification Loop' rule is most effective when it explicitly mandates the use of a specific local binary or environment-specific command to catch failures that general cloud-based linting misses.

### Examples

```
1. No interruptions during execution.
2. Plan first, then execute with the team.
3. Always run `C:\path\to\ai-code-review.exe` before finishing.
```



