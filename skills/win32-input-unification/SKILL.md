---
name: win32-input-unification
description: "WinUI3, WinRT, Zig, COM. (Atomic Win32 Keyboard Event Handling) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# Miscellaneous

Patterns: 1

## 1. Atomic Win32 Keyboard Event Handling

Preventing duplicate input events in Win32 (like '/r' becoming '//rr') by unifying WM_KEYDOWN and WM_CHAR messages into a single transaction.

### Steps

1. Processing WM_KEYDOWN and WM_CHAR as independent, unrelated events causes 'input drift' or duplication because both messages carry part of the same physical action.
2. The robust pattern is to 'latch' physical metadata (virtual keys, scancodes) in a 'pending_key' buffer during WM_KEYDOWN.
3. Finalize and dispatch the unified event only when WM_CHAR provides the UTF-8 payload, merging it with the captured physical state. This ensures a single KeyEvent per keystroke.

### Examples

```zig
const PendingKey = struct {
 vk: u16,
 scancode: u16,
 flags: u32,
};
// Latch in WM_KEYDOWN, finalize and dispatch in WM_CHAR
```



