---
name: winui3-xaml-islands-event-workarounds
description: "Documentation. (XAML Islands Event Silence Recovery) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. XAML Islands Event Silence Recovery

Critical events like TabCloseRequested often fail to fire in XAML Islands mode despite being correctly registered. Practitioners must implement manual state synchronization to maintain UI-to-Logic parity.

### Steps

1. Identify 'silent fail' events in Islands mode by verifying UI Automation (UIA) tree visibility against event handler execution; if the button exists but the handler never fires, the event is likely swallowed by the host window.
2. Implement a polling-based state observer (e.g., 500ms timer) to track collection size changes (like TabItems.Size) as a fallback for missing collection-changed events.
3. Use custom Windows Messages (WM_APP_*) to bridge communication between the host WndProc and the Zig app logic when XAML event bubbling is broken.
4. Validate fixes using UIA-driven test scripts in PowerShell to simulate user interaction, as internal unit tests cannot detect platform-level event delivery failures.

### Examples

```
// Custom message bridge in os.zig
pub const WM_APP_CLOSE_TAB = 0x8001;
```

```
// State polling loop for missing events
fn pollTabCloseState(self: *App) void {
 const current_size = self.currentTabItemsSize();
 if (current_size < self.last_polled_size) self.closeActiveTab();
}
```



