---
name: winui3-custom-titlebar
description: "Miscellaneous. (Interactive Title Bar Extension Safety) Use when user mentions: ."
---

# Miscellaneous

Patterns: 1

## 1. Interactive Title Bar Extension Safety

Using ExtendsContentIntoTitleBar in WinUI 3 without a dedicated, correctly layered drag component effectively breaks basic OS-level window management, such as dragging and resizing.

### Steps

1. Verify that enabling ExtendsContentIntoTitleBar is paired with an explicit drag bar component implementation.
2. Check for conflicts where custom UI elements overlap the non-client area, which can trap mouse input and prevent window movement.
3. Ensure the drag bar is high enough in the Z-order or specifically designated as a caption-area to maintain OS-level responsiveness.

### Examples

```
// Issue: ExtendsContentIntoTitleBar enabled but drag_bar.zig missing or non-functional
```

```
src/apprt/winui3/drag_bar.zig
```



