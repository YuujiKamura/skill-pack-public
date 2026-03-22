---
name: evidence-bounded-chronological-summaries
description: "Documentation. (Separate sourced facts from inferred narrative, User redirection is a hard scope reset) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 2

## 1. Separate sourced facts from inferred narrative

In documentation summaries, trust depends on strict source-bounded reporting. Adding plausible but unshown context (plans, prior history, side stories) was treated as lower quality than a shorter, evidence-anchored summary.

### Steps

1. If a detail is not explicitly in the provided conversation excerpt, exclude it or label it as inference.
2. Prefer chronology built from verifiable artifacts over broad retrospective storytelling.
3. In this domain, factual boundary control is a stronger quality signal than narrative completeness.


## 2. User redirection is a hard scope reset

When the user says the interpretation is wrong, prior assumptions must be discarded immediately. Continuing with adjacent tooling theories caused visible quality collapse and incomplete output.

### Steps

1. Treat explicit correction as a hard reset: restate the corrected scope before producing new content.
2. Do not blend neighboring topics (e.g., config internals) when the user asks for rule-based judgment criteria.
3. A truncated rationale (ending with an unfinished 'Because ...') is considered a failed documentation deliverable even if earlier points were useful.



