---
name: windows-smoke-verification
description: "Testing & QA. (Granular Permission & Shell Verification Sequence) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Granular Permission & Shell Verification Sequence

Systematically verifying Windows environment readiness by using sequential, uniquely-named 'smoke' files to isolate permission failures and shell compatibility issues.

### Steps

1. Raw CMD redirection (`>`) is the most direct method to test write permissions, as it bypasses shell-specific abstraction layers that might mask environment issues.
2. Transitioning from basic CMD to PowerShell-based existence checks (`Test-Path`) verifies that the environment supports polyglot scripting, which is essential for modern Windows QA pipelines.
3. Using sequential identifiers (smoke-1, smoke-2, etc.) allows for tracing the exact point of failure in an environment setup sequence, whereas a monolithic check would fail without granular diagnostic context.

### Examples

```
cmd.exe /c echo smoke-1 > C:\Users\yuuji\wt-smoke-1.txt
```

```
if (Test-Path 'C:\Users\yuuji\wt-smoke-4.txt') { Get-Item 'C:\Users\yuuji\wt-smoke-4.txt' }
```



