---
name: local-tool-integration
description: "Documentation. (Local Quality Assurance Loop Policy) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. Local Quality Assurance Loop Policy

Integrating local, proprietary review tools (e.g., custom cargo-based scanners) requires explicit documentation of command signatures and retry logic to ensure the agent uses them correctly rather than relying on generic linting.

### Steps

1. Document specific binary paths and command flags for different review scopes (file vs. git diff).
2. Define success/failure criteria and a maximum retry count (e.g., 3 attempts) for autonomous fixes.
3. Specify fallback reporting instructions if the automated loop fails to reach a passing state.



