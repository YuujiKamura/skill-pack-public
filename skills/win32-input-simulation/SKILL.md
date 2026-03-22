---
name: win32-input-simulation
description: "WinUI3, WinRT, Zig, COM. (Robust Input Simulation for Native Stress Testing) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# Testing & QA

Patterns: 1

## 1. Robust Input Simulation for Native Stress Testing

Simulating input via `SendInput` at the OS level can expose race conditions and unimplemented modes in a terminal's message loop that manual QA or high-level event simulation will miss.

### Steps

1. Use `SendInput` with raw Win32 structs (`INPUTUNION`, `KEYBDINPUT`) to verify that the application's internal mailbox can handle rapid-fire input without crashing.
2. Monitor for 'unimplemented mode' errors (e.g., mode 9001) during input simulation, as these often signal that the application is receiving valid OS events it hasn't yet mapped to internal state.
3. Verify that window focus and resizing events don't disrupt the input stream, as the intersection of these events is a primary source of non-deterministic crashes in WinUI3 apps.

### Examples

```csharp
[StructLayout(LayoutKind.Explicit)]
public struct INPUTUNION {
 [FieldOffset(0)] public MOUSEINPUT mi;
 [FieldOffset(0)] public KEYBDINPUT ki;
}
```



