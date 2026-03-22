---
name: winui3-xaml-islands-titlebar
description: "Miscellaneous. (Non-Client Area via Child Windows) Use when user mentions: ."
---

# Miscellaneous

Patterns: 1

## 1. Non-Client Area via Child Windows

Native WinUI3 titlebar extension (ExtendsContentIntoTitleBar) can cause UI thread deadlocks or freezes in XAML Island scenarios. Experienced practitioners use a dedicated 'drag-bar' child window to handle non-client interactions.

### Steps

1. Identify UI thread freeze risks when using ExtendsContentIntoTitleBar in XAML Islands (Issue #42)
2. Abandon native SetTitleBar(TabView) in favor of the 'NonClientIslandWindow' pattern found in Windows Terminal
3. Implement a transparent child window (drag-bar) that sits over the custom title area to handle WM_NCHITTEST and dragging independently of the XAML message loop

### Examples

```
// SetTitleBar is NOT used — drag-bar child window handles titlebar dragging
// instead (Windows Terminal NonClientIslandWindow pattern).
// window.setTitleBar(tv_inspectable) catch |err| { ... };
```



