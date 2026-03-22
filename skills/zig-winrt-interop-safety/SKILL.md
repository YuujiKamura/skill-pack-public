---
name: zig-winrt-interop-safety
description: "Testing & QA. (Strict Alignment Casting for WinRT Callbacks) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Strict Alignment Casting for WinRT Callbacks

WinUI3/WinRT callbacks in Zig receive arguments as `anyopaque` pointers. Safe interop requires explicit alignment validation and pointer casting to prevent runtime crashes when crossing the COM boundary.

### Steps

1. Always validate pointers for null (`orelse return`) before attempting to cast WinRT event arguments.
2. Combine `@alignCast` with `@ptrCast` when converting `anyopaque` to specific COM interface pointers to satisfy Zig's strict type system.
3. Ensure the cast target matches the expected interface defined in the WinMD-generated bindings for the specific callback.

### Examples

```zig
const ea: *com.IKeyRoutedEventArgs = @ptrCast(@alignCast(args orelse return));
```



