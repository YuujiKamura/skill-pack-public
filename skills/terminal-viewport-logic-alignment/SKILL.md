---
name: terminal-viewport-logic-alignment
description: "Documentation. (Terminal Scroll-to-Bottom Guarding) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. Terminal Scroll-to-Bottom Guarding

Correct terminal behavior requires a strict distinction between output-driven scrolling and user-initiated viewport locking to prevent jarring jumps during active reading.

### Steps

1. Implement a conditional snap-to-bottom: New output should only trigger a scroll-to-bottom if the viewport was already at the base before the data arrival.
2. Detect 'User Scroll Lock': If the user has scrolled up (even by one line), the viewport must remain locked to that offset regardless of incoming buffer updates.
3. Separate the 'TerminalCore' buffer state from the 'SwapChainPanel' rendering offset to ensure that logical updates don't force immediate, unvalidated UI movements.
4. Trace the 'ScrollToBottom' signal path through the runtime to ensure the signal isn't lost during the transition from the terminal engine to the WinUI3 rendering layer.

### Examples

```
// Pattern from Windows Terminal (WT) architecture
if (userHasScrolledUp) return; 
scrollToBottom();
```



