---
name: cross-terminal-ipc-design
description: "CLI & Tooling. (Bridging Background IPC to UI Threads in Windows Terminal) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Bridging Background IPC to UI Threads in Windows Terminal

Implementing a Ghostty-style control plane in Windows-native terminal applications requires careful synchronization between background IPC pipes and the WinUI3/Win32 message loop.

### Steps

1. Use environment variables (e.g., GHOSTTY_CONTROL_PLANE=1) to conditionally instantiate the IPC server, allowing for side-by-side testing of control plane features without affecting stable releases.
2. Bridge incoming IPC commands (e.g., NEW_TAB, INPUT) to the UI thread via PostMessage with custom WM_APP constants to ensure thread-safe interactions with UI components.
3. Store session and log metadata under %LOCALAPPDATA% to ensure the control plane remains discoverable and persistent across multiple terminal instances and reboots.

### Examples

```
PostMessage(hostingWindow, WM_APP_CONTROL_INPUT, 0, reinterpret_cast<LPARAM>(data));
```

```
// Metadata path pattern
%LOCALAPPDATA%\ghostty\control-plane\winui3
```



