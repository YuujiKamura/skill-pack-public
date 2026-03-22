---
name: claude-agent-governance
description: "Documentation. (Distinguishing Mandatory Enforcement from Process Guidance, Autonomous Decision Delegation via CLI, Mandatory Plan-to-Team Workflow) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 3

## 1. Distinguishing Mandatory Enforcement from Process Guidance

Expert practitioners distinguish between CLAUDE.md for machine-specific enforcement (overriding default agent behavior) and plugins for general process guidelines. CLAUDE.md should only contain rules that override defaults or specify local paths, as redundant instructions create unnecessary constraints.

### Steps

1. Analyze if a rule is already covered by the agent's system prompt or auto-memory.
2. Identify behaviors requiring an explicit override (e.g., 'never interrupt' vs 'ask if unsure').
3. Use CLAUDE.md to mandate specific tool paths and execution sequences that plugins only suggest as options.


## 2. Autonomous Decision Delegation via CLI

To maintain high autonomy without user interruption, agents can be instructed to consult a secondary LLM instance via CLI for low-risk decision support. This pattern bridges the gap between total autonomy and user dependency.

### Steps

1. Define a 'No Interruption' mandate in the project configuration.
2. Use shell commands like 'claude -p "..." --max-turns 1' to consult a sub-LLM for tie-breaking decisions.
3. Limit user escalation to irreversible operations like financial or destructive system changes.


## 3. Mandatory Plan-to-Team Workflow

Forcing a transition from single-agent planning to parallel multi-agent execution ensures complex tasks are not handled sequentially. This is a specific workflow constraint that must be documented to override the agent's natural tendency to work alone.

### Steps

1. Require a structured Plan mode session for non-trivial tasks.
2. Enforce the immediate creation of a team following plan approval.
3. Explicitly prohibit the agent from proceeding with implementation individually after the planning phase.



