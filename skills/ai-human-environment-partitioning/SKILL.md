---
name: ai-human-environment-partitioning
description: "CLI & Tooling. (GUI-CLI Task Boundary Strategy) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. GUI-CLI Task Boundary Strategy

Strategies for partitioning work between humans and AI when dealing with complex system SDKs (like WinUI3/MSBuild) where AI lacks interactive GUI access.

### Steps

1. AI attempts to manage GUI-heavy or interactive environment setups (e.g., Visual Studio workloads) often lead to hallucinated CLI flags and 'guessing' loops.
2. Strictly partition the 'Environment Owner' role to the human: the human performs SDK installation and IDE-based configuration.
3. The AI's role is restricted to 'Log Analyst': it should wait for the human to provide static output (e.g., buildlog.txt) rather than attempting to fix the environment itself.
4. Avoid 'environment-stalling' where the AI tries to troubleshoot missing local dependencies it cannot see; verify dependency presence via standard CLI probes before suggesting complex fixes.

### Examples

```bash
# Human runs this and provides the output to AI
msbuild /v:diag > buildlog.txt
```



