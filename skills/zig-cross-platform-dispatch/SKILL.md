---
name: zig-cross-platform-dispatch
description: "WinUI3, WinRT, Zig, COM. (Value-Based Build Configuration) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# Miscellaneous

Patterns: 1

## 1. Value-Based Build Configuration

Strategies for managing platform-specific logic in shared code (like renderers) using build-time values rather than type-level references.

### Steps

1. Type-level references in switch statements often lead to 'prong does not exist' errors during cross-compilation. Using a value-based switch on 'build_config.app_runtime' allows the compiler to prune unreachable branches while maintaining a consistent interface across different platform-specific enum definitions.
2. Shared components (like OpenGL renderers) must have exhaustive switches that exactly match the 'Runtime' enum tags. Including 'just-in-case' cases like '.embedded' that are not present in the target enum will break the build, even if that branch is logically unreachable.
3. Centralizing build decisions in a 'build_config.zig' avoids the pitfall of spreading '@import("builtin")' or OS-detection logic across the codebase, making it easier to simulate different runtime configurations on a single host.

### Examples

```zig
const build_config = @import("build_config.zig");

// Correct: Switch on the value of the runtime enum
switch (build_config.app_runtime) {
 .winui3 => handleWinUI3(),
 .gtk => handleGTK(),
 .none => {},
 // No '.embedded' if it's not in the Runtime enum
}
```



