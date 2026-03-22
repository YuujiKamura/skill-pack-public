---
name: windows-low-level-terminal-perf
description: "WinUI3, WinRT, Zig, COM. (High-Performance Windows UI/Terminal Engineering) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# CLI & Tooling

Patterns: 1

## 1. High-Performance Windows UI/Terminal Engineering

Critical low-level adjustments required to overcome default Windows OS limitations for high-refresh-rate terminal applications.

### Steps

1. The default Windows timer resolution (~15.6ms) is too coarse for 120FPS (8.3ms) targets; `timeBeginPeriod(1)` must be used to force the scheduler into a 1ms resolution.
2. CPU-GPU serialization via `gl.finish()` causes micro-stalls; `gl.flush()` combined with a dedicated VSync thread (using `DwmFlush`) provides smoother frame pacing.
3. Terminal I/O throughput is often bottlenecked by mutex contention in the read loop; increasing the internal read buffer (e.g., from 1KB to 16KB) significantly reduces lock-unlock overhead.
4. UI freezing during window resizing is typically caused by message pump starvation; upgrading from `GetMessage` to `MsgWaitForMultipleObjectsEx` allows the app to respond to I/O while the OS processes window messages.

### Examples

```zig
// Force 1ms timer resolution for high FPS
_ = win32.timeBeginPeriod(1);

// Non-blocking wait for messages or IO
_ = MsgWaitForMultipleObjectsEx(count, pHandles, timeout, mask, flags);
```



