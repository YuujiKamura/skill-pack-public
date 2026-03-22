---
name: cli-tooling
description: "CLI & Tooling. (Local-First Plugin Wrap-around Strategy, Refactoring Integrity Checklist for CLI Libraries, Recursive Agent Execution Prevention, Temporal VCS Context Harvesting) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Conversations: 4 | Patterns: 4

## 1. Local-First Plugin Wrap-around Strategy

When integrating established CLI tools into AI agent ecosystems (like Claude Code), practitioners prioritize simple shell hooks and local binary wrappers over complex protocol migrations (like MCP). This preserves existing tool stability while enabling automated lifecycle triggers.

### Steps

1. Identify lifecycle hooks (e.g., session-start, session-end) supported by the host agent.
2. Create shell-based wrapper scripts (e.g., .cmd or .sh) to invoke the existing CLI binary.
3. Map tool-specific configurations to the agent's plugin manifest (plugin.json) without refactoring the core logic.


## 2. Refactoring Integrity Checklist for CLI Libraries

Experienced practitioners apply a specific validation suite when moving logic between CLI modules (e.g., to util.rs or lib.rs) to ensure the tool remains usable as both a binary and a library.

### Steps

1. Verify public API (pub) compatibility to ensure no breaking changes for external callers.
2. Explicitly check for circular dependencies introduced by extracting 'helpers'.
3. Validate that logic extraction into common methods (e.g., render_pattern) produces identical output to the original inline logic.


## 3. Recursive Agent Execution Prevention

In CLI environments where an agent might be tasked to verify changes or run scripts that invoke other AI calls, users implement strict 'no-recursion' constraints to prevent infinite loops and token waste.

### Steps

1. Explicitly instruct the agent to ignore persistent project-level instructions (e.g., CLAUDE.md) that trigger sub-tasks.
2. Mandate binary (YES/NO) reporting for verification tasks to limit conversational drift.
3. Restrict the agent from making additional tool-based 'recursive' calls during validation phases.


## 4. Temporal VCS Context Harvesting

CLI tools designed for developer productivity leverage version control metadata combined with temporal filters to generate high-relevance context for AI-assisted summaries.

### Steps

1. Query git logs with specific temporal constraints (e.g., --since='midnight').
2. Filter commits by author and project scope to isolate daily contributions.
3. Pipe filtered VCS data into domain-specific miners (e.g., skill-miner) to extract actionable patterns.



