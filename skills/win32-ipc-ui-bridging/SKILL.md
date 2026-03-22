---
name: win32-ipc-ui-bridging
description: "CLI & Tooling. (IPC-to-UI Thread Bridging) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. IPC-to-UI Thread Bridging

Background IPC servers (Named Pipes) must never directly manipulate UI components to maintain thread safety in Win32/WinUI3 applications.

### Steps

1. The critical judgment is never allowing an IPC thread to directly call UI methods (e.g., in TerminalPage or TerminalWindow). This causes race conditions and sporadic crashes.
2. The correct design principle is to 'queue' an intent using PostMessage with custom constants (WM_APP + N). This ensures the UI thread remains the sole owner of application state and processes the IPC request in its own message loop.

### Examples

```cpp
// In background IPC thread
PostMessage(hostingWindow, WM_APP_CONTROL_INPUT, 0, (LPARAM)payload);

// In UI thread (Window Procedure)
case WM_APP_CONTROL_INPUT:
 _HandleInputPayload((BSTR)lParam);
 break;
```



