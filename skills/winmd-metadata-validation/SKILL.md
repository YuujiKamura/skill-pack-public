---
name: winmd-metadata-validation
description: "WinUI3, WinRT, Zig, COM. (Automated Metadata Truth-Checking) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# CLI & Tooling

Patterns: 1

## 1. Automated Metadata Truth-Checking

Shifts from manual 'guess-and-check' vtable debugging to automated validation using WinMD as the absolute source of truth.

### Steps

1. Avoid manual index counting for interfaces with high method counts (like `IFrameworkElement`); use specialized PowerShell scripts or `win-zig-bindgen` to extract the ground-truth IIDs and offsets.
2. Compare manual vtable definitions against 'known good' generated bindings to identify off-by-one errors in interface inheritance that are invisible during compilation but fatal at runtime.
3. Standard documentation can be misleading regarding internal vtable layouts; the binary WinMD file is the only reliable source for slot indices.

### Examples

```
powershell.exe -File scripts/winui3-winmd-iid-check.ps1
```

```
./win-zig-bindgen.exe --winmd Microsoft.UI.Xaml.winmd --iface IFrameworkElement
```



