---
name: winui3-islands-build-encapsulation
description: "Documentation. (WinUI3 Islands Build Lifecycle Management) Use when user mentions: README, docs, documentation, tutorial, guide, comment."
---

# Documentation

Patterns: 1

## 1. WinUI3 Islands Build Lifecycle Management

Standard 'zig build' commands fail to handle the complex manifest injection and XAML compilation requirements of WinUI3 Islands projects.

### Steps

1. Strictly avoid raw compiler calls; use project-specific wrapper scripts (e.g., build-winui3.sh) that manage MSVC/WinSDK environment variables and resource compilation.
2. Enforce 'First Failure Triage' workflows where build errors are resolved before functional testing, as Islands-specific linking errors often manifest as silent runtime crashes or missing UI components.
3. Prioritize XAML manifest correctness over code changes when UI elements appear in the UIA tree but fail to respond to interaction.

### Examples

```
bash ./build-winui3.sh
```

```
pwsh.exe -File tests/winui3/run-all-tests.ps1
```



