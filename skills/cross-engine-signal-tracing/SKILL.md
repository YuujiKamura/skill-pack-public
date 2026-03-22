---
name: cross-engine-signal-tracing
description: "Documentation. (Reference Architecture Tracing (WT to Ghostty)) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. Reference Architecture Tracing (WT to Ghostty)

When fixing fundamental terminal behaviors in a custom engine, mapping the signal path against a reference implementation (like Windows Terminal) is more effective than local debugging.

### Steps

1. Identify the 'Source of Truth' in the reference codebase (e.g., TerminalCore::_WriteBuffer in Windows Terminal).
2. Trace the callback chain from the buffer update to the UI invalidation (Search terms: 'UserScrollViewport', '_scrollToBottom').
3. Compare the architectural 'Gap': determine if the signal is lost at the Terminal-to-AppRuntime interface or the AppRuntime-to-OS-Window interface.

### Examples

```bash
# Search WT for scroll trigger patterns
rg "_scrollToBottom" src/cascadia/TerminalCore
```



