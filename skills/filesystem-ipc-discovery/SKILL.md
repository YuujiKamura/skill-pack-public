---
name: filesystem-ipc-discovery
description: "CLI & Tooling. (Filesystem-Based IPC Discovery) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Filesystem-Based IPC Discovery

Reliable IPC discovery in Windows relies on publishing endpoint metadata in predictable application data paths for external tool access.

### Steps

1. For CLI tools to 'find' a running terminal instance, the server must publish its endpoint metadata in the filesystem. Using %LOCALAPPDATA% is superior to registry keys as it naturally aligns with the app's data sandbox.
2. Tying the lifecycle of the discovery token (e.g., a pipe name file) to the active session allows agents to locate instances without elevated permissions or complex registry lookups.

### Examples

```cpp
// Path: %LOCALAPPDATA%\ghostty\control-plane\winui3
auto pipeName = L"\\.\\pipe\\wt-control-" + sessionId;
```



