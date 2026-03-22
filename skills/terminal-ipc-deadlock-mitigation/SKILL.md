---
name: terminal-ipc-deadlock-mitigation
description: "CLI & Tooling. (Terminal IPC Deadlock Prevention) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Terminal IPC Deadlock Prevention

Integrating IPC servers into a terminal's core introduces high risk of deadlocking the renderer or input loops if threads block each other.

### Steps

1. A hidden deadlock occurs if the IPC server waits for a UI lock while the UI thread is blocked by a synchronous IPC call. The terminal will hang indefinitely.
2. The design principle is to treat all IPC requests as asynchronous 'request-callback' patterns. Implement the IPC listener as a detached background thread and ensure buffer scrapes are performed under extremely short-lived locks to maintain UI responsiveness.

### Examples

```cpp
// Issue titled: "ControlPlane: Deadlock in UI thread"
// Fix: Move heavy logic out of the pipe listener's main loop
```



