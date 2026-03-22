---
name: upstream-contribution-safety
description: "AI & Machine Learning. (Fork-First Upstream Contribution Flow) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Fork-First Upstream Contribution Flow

Strict validation criteria for AI agents interacting with high-profile open-source repositories to prevent premature issues or noisy PRs.

### Steps

1. Always verify the remote origin (`git remote -v`) and visibility before allowing an AI to propose or create an issue in a public tracker.
2. Enforce a 'Fork-First' rule: AI must demonstrate a successful build and test cycle on a private fork before any upstream interaction is considered.
3. Avoid 'AI-authored' noise in community trackers by ensuring the agent checks if it is actually operating on its own fork or a direct upstream clone.

### Examples

```
git remote -v
```

```
gh repo view --json visibility
```



