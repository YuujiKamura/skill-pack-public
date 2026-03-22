---
name: zig-winrt-vtable-mastery
description: "WinUI3, WinRT, Zig, COM, vtable, ghostty. (WinRT COM VTable Mastery in Zig) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# WinRT COM VTable Mastery in Zig

Patterns: 4

## 1. WinMD-Driven VTable Slot Alignment

Prevents memory corruption and segmentation faults in Zig/WinUI3 applications by ensuring manual COM interface vtables exactly match the Windows Metadata (WinMD) structure.

### Steps

1. Manual vtable mapping is extremely brittle; a single missing placeholder (e.g., slot 10 vs 41) causes method calls to overwrite unrelated memory with pointers, leading to crashes during object cleanup (e.g., `.release()`).
2. When a setter like `put_Background` causes a segfault, verify if the vtable index aligns with the WinMD definition by checking the exact number of inherited methods and hidden padding slots.
3. Overwriting memory with a brush pointer is a specific signature of a vtable offset error where a method call is dispatched to a slot currently occupied by a different pointer type.
4. VTable misalignment is the most frequent cause of 'silent' crashes after a WinUI3 UI overhaul. While placeholders allow quick prototyping, the final VTable must strictly match the WinMD layout; even one missing slot shifts all subsequent method calls to the wrong memory address.

### Examples

```zig
// Correcting vtable by shifting put_Background to slot 41
p6: [34]usize, // Placeholder for slots 6 through 39
get_Background: *const fn(self: *IControl, value: **IBrush) callconv(WINAPI) HRESULT,
put_Background: *const fn(self: *IControl, value: *IBrush) callconv(WINAPI) HRESULT,
```

## 2. Concrete Function Signatures and callconv(.winapi)

Ensuring binary compatibility with WinRT by using concrete function signatures with explicit calling conventions.

### Steps

1. Concrete function signatures using `callconv(.winapi)` are non-negotiable for WinRT stability. Generic pointers fail to handle the stack and registers correctly for HRESULT-returning methods, leading to stack corruption that only manifests several calls later.
2. Manual definition of IIDs (Interface IDs) for event handlers is a critical fallback when binding generators are incomplete. Without explicit IIDs, subscription methods like `add_SizeChanged` cannot uniquely identify the callback interface, causing runtime registration failures.

### Examples

```zig
// Concrete function pointer type for WinRT VTable
put_CanReorderTabs: *const fn (*anyopaque, bool) callconv(.winapi) HRESULT,
```

```zig
// Defining missing IID for event registration
const IID_SizeChangedEventHandler = GUID.parse("...");
```

## 3. Out-Parameter Pointer Alignment for Interface Getters

Correcting Zig-to-WinRT VTable mappings for interface getters to match the COM ABI requirement for output parameters.

### Steps

1. Low-level COM bindings in Zig often fail if getter methods don't account for the explicit out-parameter pointer expected by the ABI.
2. Interface pointers returned by getters (e.g., in IResourceDictionary) must be defined as `*?*anyopaque` (pointer to an optional opaque pointer) rather than simple return values.
3. Missing these output parameters in the VTable definition causes memory corruption or 'broken vtable' errors because the caller and callee disagree on the stack/register layout.

### Examples

```zig
// Correct VTable slot for a getter returning an interface
get_Source: *const fn (*anyopaque, **?anyopaque) callconv(.Inline) HRESULT,
```

## 4. Crash Triage: Lifecycle vs Layout Separation

When debugging WinUI3 startup failures, distinguish lifecycle failures (events not firing, COM contract corruption) from layout/geometry issues (header height, alignment).

### Steps

1. A vtable slot drift crash may appear at an unrelated line (e.g., `brush.release()` after `put_Background` was dispatched to the wrong slot). Always verify slot indices against WinMD before trusting the crash frame.
2. Treat 'Loaded not firing' and 'header height wrong' as distinct failure classes. Gate layout experiments behind vtable/IID integrity checks — contract corruption invalidates all UI-level observations.
3. Use symptom-specific evidence (event firing logs vs panel dimensions) to decide which branch to pursue.
