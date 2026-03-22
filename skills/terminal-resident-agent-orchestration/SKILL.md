---
name: terminal-resident-agent-orchestration
description: "AI & Machine Learning. (Terminal-Resident AI Orchestration) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Terminal-Resident AI Orchestration

Using a terminal process to host persistent AI agents (Claude/Gemini/Codex) as 'resident' workers to maintain execution context and allow for asynchronous task 'relaying'.

### Steps

1. Use 'Resident' (常駐) mode to keep an AI agent alive in a specific terminal session to preserve long-term context that is lost in one-shot executions.
2. Distinguish between terminal hosts (e.g., Windows Terminal vs Ghostty) because session stability and IPC mechanisms vary significantly between platforms.
3. Implement a 'Relay' pattern (Start/Send/Status/Stop) to decouple heavy research or long-running tasks from the main conversation loop, effectively treat the terminal as a background compute node.

### Examples

```
pwsh -File winui3-ai-worker.ps1 -Action start -Agent claude -Session wt-ai-claude
```

```
pwsh -File winui3-ai-worker.ps1 -Action send -Agent claude -Text "Research the deadlock in ControlPlane.cpp"
```



