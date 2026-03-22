---
name: winui3-islands-ime-redirection
description: "Testing & QA. (TSF Focus Redirection for WinUI3 Islands) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. TSF Focus Redirection for WinUI3 Islands

In WinUI3 Islands, the SwapChainPanel lacks a native IME context and the XAML message loop intercepts physical keyboard events. To support IME composition, focus must be programmatically shifted to a hidden TextBox that handles the Text Services Framework (TSF) lifecycle.

### Steps

1. Physical keyboard messages are intercepted by the XAML loop, preventing the underlying HWND from receiving them directly.
2. Shift keyboard focus to a dedicated XAML TextBox (ime_text_box) when IME-related virtual keys (e.g., VK_PROCESSKEY 0xE5 or mode toggles) are detected in PreviewKeyDown.
3. Utilize the TextBox's built-in TSF support (CompositionStarted/Changed/Ended) to manage input while rendering results to the SwapChainPanel.

### Examples

```zig
if (isImePassthroughVirtualKey(vk_u32)) {
 self.app.keyboard_focus_target = .ime_text_box;
 _ = self.focusImeTextBox();
 return; // Let IME process.
}
```



