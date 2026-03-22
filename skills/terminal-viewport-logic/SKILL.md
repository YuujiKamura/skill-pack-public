---
name: terminal-viewport-logic
description: "Documentation. (Sticky Scroll-to-Bottom Heuristics) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. Sticky Scroll-to-Bottom Heuristics

Terminal scrolling isn't a binary 'on/off' state; it requires preserving user intent. The 'sticky' behavior depends on the current viewport position relative to the buffer end.

### Steps

1. Define 'Auto-scroll' as conditional: only trigger if the viewport is already at the bottom when new output arrives.
2. Implement a 'User Scroll Lock': if the user has manually scrolled up even by one line, disable auto-scroll to prevent the viewport from jumping while they are reading history.
3. Trace the 'signal loss' path: in complex renderers (like D3D11/WinUI3), verify the scroll signal travels from TerminalCore (buffer write) through the swap chain to the actual ScrollViewer.

### Examples

```rust
// Decision logic for auto-scroll
if viewport.is_at_bottom() {
 viewport.scroll_to_bottom();
} else {
 // Preserve user position if they scrolled up
 viewport.maintain_relative_position();
}
```



