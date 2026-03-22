---
name: winui3-custom-titlebar-stability
description: "Testing & QA. (Non-Client Island Drag-Bar Pattern) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Non-Client Island Drag-Bar Pattern

Native WinUI3 APIs like `SetTitleBar` combined with `ExtendsContentIntoTitleBar` frequently cause UI thread deadlocks (Issue #42). A more stable approach is to use a separate child window or a 'drag-bar' component for non-client interactions.

### Steps

1. Avoid the built-in `SetTitleBar` API in complex or 'island' scenarios to prevent UI thread freezes during window chrome interactions.
2. Implement title bar dragging using a dedicated child window (NonClientIslandWindow pattern), as seen in Windows Terminal.
3. Decouple the XAML layout from the window's non-client logic to ensure the UI remains responsive during window moves and resizes.

### Examples

```
// SetTitleBar is NOT used — drag-bar child window handles titlebar dragging
// instead (Windows Terminal NonClientIslandWindow pattern).
// SetTitleBar requires ExtendsContentIntoTitleBar which causes UI thread freeze (Issue #42).
```



