---
name: winrt-interop-debugging
description: "WinUI3, WinRT, Zig, COM. (Diagnosing Opaque WinRT Exceptions (0xC000027B)) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# Testing & QA

Patterns: 1

## 1. Diagnosing Opaque WinRT Exceptions (0xC000027B)

STATUS_STOWED_EXCEPTION is a common but vague error in WinUI3/WinRT interop. Resolving it requires a disciplined diagnostic loop to pinpoint ABI boundary failures rather than applying speculative fixes.

### Steps

1. Adopt a three-step diagnostic loop (inject logs → isolate line → apply fix) specifically targeting the 'stowed exception' which often masks the true source of a WinRT failure.
2. Focus investigation on WinRT 'Activation Factories' and vtable slot calls (e.g., `CreateString` at slot 18) when crashes occur during high-frequency UI updates like title changes or text input.
3. Distinguish between logic errors and ABI violations; a crash immediately following a 'mailbox' or 'resize' event often indicates a threading or state conflict in the WinRT/Unmanaged boundary.

### Examples

```zig
// Targeted logging around WinRT boundaries
log.info("calling getActivationFactory for IPropertyValueStatics", .{});
const factory = try getActivationFactory(IPropertyValueStatics);
log.info("factory acquired", .{});
```



