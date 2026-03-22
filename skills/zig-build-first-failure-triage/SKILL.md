---
name: zig-build-first-failure-triage
description: "WinUI3, WinRT, Zig, COM. (Prioritize direct compiler diagnostics over speculative source digging) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# CLI & Tooling

Patterns: 1

## 1. Prioritize direct compiler diagnostics over speculative source digging

In Zig build failures that surface through `std/start.zig` wrappers (`callMain`, `WinStartup`), the useful signal is usually in the first compiler-reported error chain, not in ad-hoc stdlib inspection. This prevents losing time on indirect hypotheses.

### Steps

1. Treat `std/start.zig` stack frames as dispatch context, not root cause, unless evidence points there.
2. When Zig says `1 compilation errors` with `-freference-trace`, expand the reference trace before changing code; this is the fastest path from wrapper frame to user-code/type mismatch.
3. Reject investigations that cannot tie back to the exact emitted diagnostic line and symbol causing the failure.

### Examples

```bash
zig build -freference-trace=4
```



