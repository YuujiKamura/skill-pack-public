---
name: ai-agent-cli-design
description: "AI & Machine Learning. (Safety-First CLI Defaults for AI Agents, Subcommand Discovery via Agent Documentation) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 2

## 1. Safety-First CLI Defaults for AI Agents

Prevents accidental side-effects when AI agents map user intent to state-modifying commands.

### Steps

1. AI agents frequently misinterpret informational requests (e.g., 'summarize today') as execution requests for the underlying process. Implementing a 'dry-run by default' policy for destructive subcommands ensures that agent errors remain non-destructive even when intent mapping fails.
2. Requiring an explicit '--deploy' or '--execute' flag creates a deliberate circuit breaker that requires the agent to confirm high-stakes actions, mitigating the risk of accidental production changes caused by LLM hallucinations.

### Examples

```
cargo run -- mine --max-days 1 # Implicit --dry-run
```

```
cargo run -- mine --max-days 1 --deploy # Explicit execution
```


## 2. Subcommand Discovery via Agent Documentation

Guides AI agents toward 'safe' read-only operations through explicit documentation in tool descriptions.

### Steps

1. Agents lack the 'common sense' to know that a summarization task shouldn't involve a deployment. Explicitly documenting safe, read-only alternatives (like 'today --format summary') in the tool description prevents the agent from defaulting to more powerful, risky subcommands.
2. Clear separation between 'view' and 'modify' commands in the documentation acts as a critical 'Agent UI' layer, reducing the likelihood of the model selecting a high-privilege tool for a low-privilege task.

### Examples

```
// In tool/skill description:
"Use the 'today' subcommand for safe, read-only summaries; use 'mine --deploy' only for permanent skill extraction."
```



