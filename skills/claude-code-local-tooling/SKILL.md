---
name: claude-code-local-tooling
description: "Documentation. (Local Verification Tool Integration) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. Local Verification Tool Integration

Integrating custom local CLI tools into the agent's workflow by providing absolute paths and specific command mappings in project documentation.

### Steps

1. Define absolute file paths to local tools (e.g., `C:\Users\yuuji\ai-code-review`) to ensure the agent can execute them regardless of the working directory.
2. Specify concrete command mappings for different scopes, such as `cargo run --bin review -- --diff` for git changes or `--prompt security` for audits.
3. Establish a hard limit on autonomous retry attempts (e.g., max 3 times) for verification failures before the agent is required to report the status to the user.

### Examples

```
cargo run --bin review -- <対象ファイル>
```

```
cargo run --bin review -- --diff
```

```
cargo run --bin review -- --prompt <default|quick|security|architecture|holistic|principles>
```

```
cargo run --bin review -- --backend <gemini|claude>
```



