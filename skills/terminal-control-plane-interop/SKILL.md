---
name: terminal-control-plane-interop
description: "AI & Machine Learning. (Control Plane IPC for AI-Terminal Integration) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Patterns: 1

## 1. Control Plane IPC for AI-Terminal Integration

Implementing a 'Control Plane' protocol via Named Pipe IPC to allow external AI agents to programmatically control terminal behaviors like tab management and input injection.

### Steps

1. Adopt established protocols (like Ghostty's control-plane) when porting features to new terminals like Windows Terminal to ensure agent logic remains portable.
2. Verify all build metadata (e.g., .vcxproj, .sln) after adding IPC components; agents often miss project configuration files which leads to linking errors even if source code is correct.
3. Use specific platform message IDs (e.g., WM_USER+6) to bridge the gap between low-level terminal logic and modern UI frameworks like WinUI3. (Note: terminal-control-plane repo is archived.)

### Examples

```cpp
// ControlPlane.cpp integration
#define WM_USER_CLOSE_TAB (WM_USER + 6)
PostMessageW(self->hwnd, WM_USER_CLOSE_TAB, 0, 0);
```



