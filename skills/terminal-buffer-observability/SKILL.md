---
name: terminal-buffer-observability
description: "CLI & Tooling. (Decoupled Terminal Buffer Scraping for Remote Monitoring) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Decoupled Terminal Buffer Scraping for Remote Monitoring

Implementing 'tail' or 'snapshot' features in IPC-enabled terminals without injecting heavy dependencies or disrupting the rendering pipeline.

### Steps

1. Expose a specialized ViewportText method on terminal controls to allow the IPC layer to 'scrape' only the visible buffer, minimizing performance overhead compared to full pty tailing.
2. Base64-encode raw input bytes before routing them through the IPC pipe to prevent control character interference or shell-level mangling during transport.
3. Map UI state queries (LIST_TABS, STATE) directly to existing UI component accessors (TerminalPage::NumberOfTabs) to avoid redundant state management in the IPC thread.

### Examples

```
// Scrape visible text for IPC 'TAIL' command
auto text = termControl.ViewportText();
```

```
// Base64 decode IPC input before routing to PTY
auto decoded = base64_decode(input_msg);
```



