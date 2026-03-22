---
name: zig-toolchain-stability
description: "WinUI3, WinRT, Zig, COM. (Compiler Evolution Awareness) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# CLI & Tooling

Patterns: 1

## 1. Compiler Evolution Awareness

Mitigating breaking changes in low-level binding generators caused by Zig standard library refactors.

### Steps

1. Minor version increments in Zig (e.g., 0.15.2) frequently refactor internal entry points like 'std.start' and 'io.zig', which breaks low-level WinRT/COM binding generators.
2. When a generator fails compilation after a toolchain update, prioritize auditing the interaction with getStdOut and callMain signatures rather than user-land logic. Success depends on maintaining version-locked awareness of the initialization sequence.

### Examples

```
zig build-exe generator.zig --pkg-var winmd=... 
```

```
std.io.getStdOut() // Check for signature changes in 0.15.2
```



