---
name: feature-build
description: >
  Unified feature development workflow that auto-triggers on feature requests.
  Combines codebase exploration, architecture design, parallel implementation, and code review.
  Use this skill whenever the user asks to add a feature, implement something new, build a component,
  or make non-trivial changes to the codebase. Triggers on: "implement", "add feature", "build",
  "create", "develop", "make it so that", "add support for", or any request that involves writing
  new functionality across multiple files. Do NOT trigger for: bug fixes (use systematic-debugging),
  simple one-file edits, config changes, or questions about the codebase.
---

# Feature Build

End-to-end feature development: explore → design → implement → review.

This skill replaces the separate brainstorming and feature-dev workflows with a single
streamlined pipeline tuned for this user's preferences:
- No one-question-at-a-time interviews. Decide yourself or ask everything at once.
- No spec documents. Design lives in plan mode, not in markdown files.
- Parallel team execution for implementation.
- Automated code review before completion.

## When NOT to use this skill

- Bug fixes → use `superpowers:systematic-debugging`
- Single-file changes → just do it directly
- Questions about code → use Explore agent
- Refactoring → just do it directly

## Phase 1: Explore (2-3 minutes)

**Goal**: Understand the codebase before touching it.

Launch 2-3 code-explorer agents in parallel. Each targets a different aspect:

```
Agent 1: "Find features similar to [X] in this codebase. Trace their implementation.
          Return the 5-10 most important files."

Agent 2: "Map the architecture around [area]. What are the key abstractions,
          data flows, and extension points? Return the 5-10 most important files."

Agent 3: (if UI involved) "Analyze UI patterns used for similar features.
          What components, layouts, and interaction patterns exist?"
```

After agents return, **read every file they identified**. Build a mental model
of existing patterns and conventions before designing anything.

Summarize findings in 3-5 bullet points. Do not ask the user to confirm —
just proceed to design.

## Phase 2: Design (Plan Mode)

**Goal**: Decide the approach and get user buy-in.

Enter plan mode. Present:

1. **What you learned** — key patterns, conventions, constraints from Phase 1
2. **2-3 approaches** — with trade-offs. Lead with your recommendation.
   - Minimal: smallest change, maximum reuse
   - Clean: best architecture, more work
   - Pragmatic: balance of speed and quality
3. **Your pick and why**
4. **File list** — which files to create/modify

If something is genuinely ambiguous (affects the user, not just implementation),
ask all questions in a single block. Otherwise, decide yourself.

Wait for user approval of the plan before proceeding.

## Phase 3: Implement (Parallel Teams)

**Goal**: Build it with parallel agents.

After plan approval, break the work into independent chunks and dispatch
teams in parallel:

- Each team gets a clear, self-contained task
- Each team gets the relevant file list and conventions from Phase 1
- Teams should follow existing patterns strictly

Monitor progress. When all teams complete, verify integration points
where their work connects.

## Phase 4: Review

**Goal**: Catch issues before claiming done.

Launch 2-3 code-reviewer agents in parallel:

```
Reviewer 1: "Review for bugs, logic errors, and edge cases."
Reviewer 2: "Review for adherence to project conventions and DRY violations."
Reviewer 3: "Review for security issues and performance problems."
```

Also run the project's own review tool if available:
```bash
cargo run --bin review -- --diff
```

If reviewers find issues:
- Fix critical issues immediately
- Report non-critical issues to user with recommendation

## Phase 5: Complete

**Goal**: Clean finish.

1. Verify build passes
2. Run relevant tests
3. Summarize what was built, key decisions, files changed
4. Do NOT commit unless asked

## Key Rules

- **Read before write**: Never propose changes to code you haven't read
- **Follow existing patterns**: The codebase already has conventions. Use them.
- **Parallel by default**: If tasks are independent, run them in parallel
- **No unnecessary questions**: Decide yourself. The user trusts your judgment
  on implementation details. Only ask about things that affect user experience
  or public API.
- **No spec documents**: The plan IS the spec. Don't write markdown design docs.
