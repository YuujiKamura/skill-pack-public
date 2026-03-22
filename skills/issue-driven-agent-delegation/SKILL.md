---
name: issue-driven-agent-delegation
description: "Testing & QA. (Issue-Driven Infrastructure for Multi-Agent Workflows) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Issue-Driven Infrastructure for Multi-Agent Workflows

Infrastructure-heavy testing tasks (like WinUI3 automated setups) are best managed by decomposing them into a dependency-aware issue map before a single line of code is written. This allows for parallel agent execution without context drift.

### Steps

1. Interrupt implementation flow to establish a source-of-truth issue tracker via CLI (e.g., gh) as soon as multiple foundational components (COM, Input, Harness) are identified.
2. Classify components by dependency level (P0 independent vs P1/P2 sequential) to maximize parallel agent throughput while maintaining architectural integrity.
3. Use a tracking issue as the central 'brain' for the delegation strategy, preventing agents from hallucinating dependencies that haven't been implemented yet.

### Examples

```powershell
gh issue create --title "[P0] Infrastructure: com_runtime.zig" --body "Implement thread-safe COM wrapper for WinUI3 testing..."
```



