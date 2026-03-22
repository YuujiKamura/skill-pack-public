---
name: win32-powershell-testing
description: "WinUI3, WinRT, Zig, COM. (Low-Level Native UI Testing with PowerShell and P/Invoke) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# Testing & QA

Patterns: 1

## 1. Low-Level Native UI Testing with PowerShell and P/Invoke

For high-performance Win32/WinUI3 applications, standard UI automation frameworks are often too slow or detached. A native approach using PowerShell with embedded C# P/Invoke allows for precise window manipulation and synchronization via internal log parsing.

### Steps

1. Prefer pure PowerShell with `Add-Type` for C# Win32 API access to avoid the overhead and instability of external testing frameworks in native development environments.
2. Use internal `stderr` log markers (e.g., 'initXaml step 7 OK') as synchronization primitives for test scripts rather than non-deterministic `Sleep` calls or UI-only polling.
3. Implement test suites as incremental stages (e.g., process startup → window visibility → component init → interaction) to isolate failures in the complex Win32/WinUI3 initialization handshake.

### Examples

```powershell
$Signature = @"
[DllImport("user32.dll")]
public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
"@
Add-Type -MemberDefinition $Signature -Name "Win32" -Namespace "Native"
```

```powershell
# Wait for internal state via stderr
while ($line = $process.StandardError.ReadLine()) {
 if ($line -match "initXaml step 7 OK") { break }
}
```



