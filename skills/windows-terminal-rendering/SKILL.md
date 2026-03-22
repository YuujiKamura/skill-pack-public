---
name: windows-terminal-rendering
description: "WinUI3, WinRT, Zig, COM. (D3D11 vs. OpenGL VSync Jitter Mitigation) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# Miscellaneous

Patterns: 1

## 1. D3D11 vs. OpenGL VSync Jitter Mitigation

Traditional OpenGL rendering with DwmFlush causes frame drops in Windows terminals due to compositor VSync jitter; D3D11 with Flip Model is the high-performance alternative.

### Steps

1. DwmFlush blocks on the DWM compositor, which is prone to timing jitter. Relying on it for high-performance terminal rendering often leads to perceived lag or dropped frames.
2. The gold standard for Windows terminal performance (pioneered by Windows Terminal) is a D3D11 backend using DXGI_SWAP_EFFECT_FLIP_DISCARD and DirectWrite.
3. If stuck with a legacy renderer, decouple the terminal's internal state updates from the DwmFlush VSync loop using a platform-specific compile-time branch to prevent UI stalls.

### Examples

```zig
// Short-term mitigation: Decouple from DwmFlush
comptime if (!@hasDecl(renderer, "win32_vsync")) {
 // Decouple terminal text updates from strict VSync loop
}
```



