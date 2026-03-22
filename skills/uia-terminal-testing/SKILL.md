---
name: uia-terminal-testing
description: "Miscellaneous. (OS-Level UIA Validation for Terminals) Use when user mentions: ."
---

# Miscellaneous

Patterns: 1

## 1. OS-Level UIA Validation for Terminals

Terminal emulators often intercept low-level input; standard unit tests cannot verify if OS-level features like window dragging, resizing, or caption button hits are broken by the application's input loop.

### Steps

1. Use PowerShell-based UI Automation (UIA) to test behaviors that the OS, not the app, should handle (e.g., titlebar double-clicks, dragging).
2. Isolate 'window-visible' and 'resize' as primary smoke tests for WinUI 3 islands, as these frequently fail during migration.
3. Script the verification of non-client area interactions (caption buttons) to ensure custom XAML content hasn't obscured them.

### Examples

```powershell
# Example UIA test script structure
./tests/winui3/test-02-titlebar-drag.ps1
./tests/winui3/test-05-caption-buttons.ps1
```



