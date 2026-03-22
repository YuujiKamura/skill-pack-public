---
name: uia-island-verification
description: "Documentation. (UIA-Based Testing for Islands UI) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. UIA-Based Testing for Islands UI

Traditional UI testing frameworks often struggle with XAML Islands. Using User Interface Automation (UIA) to find and 'Invoke' elements is the reliable way to verify event wiring.

### Steps

1. Use AutomationId as the primary key for element discovery in XAML Islands, as the visual tree can be volatile.
2. Simulate user actions via UIA patterns (e.g., InvokePattern) rather than raw mouse clicks to ensure the event reaches the intended XAML element.
3. Verify 'Last Tab Close' behavior: confirm that closing the final tab via UI automation correctly triggers the process exit signal.

### Examples

```powershell
# PowerShell UIA test for Tab Close
$closeBtn = $tabItem.FindFirst($TreeScope_Descendants, $condCloseBtn)
$closeBtn.GetCurrentPattern($InvokePattern).Invoke()
```



