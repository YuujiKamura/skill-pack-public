---
name: claude-plugin-architecture
description: "CLI & Tooling. (Cross-Platform Event Hooks) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Cross-Platform Event Hooks

Designing Claude Code plugins that execute background tasks on session events while maintaining compatibility across Windows and Unix environments.

### Steps

1. Define event triggers in `hooks/hooks.json` mapping system events like `SessionEnd` to executable command strings.
2. Implement a polyglot `run-hook.cmd` script that uses shell detection logic to find and invoke the correct interpreter (Bash vs. CMD) regardless of the host OS.
3. Execute long-running or non-critical tasks (like background mining) by redirecting all output to a log file within the plugin directory to ensure the terminal session terminates smoothly.
4. Use environment variables like `${CLAUDE_PLUGIN_ROOT}` to resolve absolute paths to scripts within the plugin structure.

### Examples

```json
{
 "hooks": {
 "SessionEnd": [
 {
 "hooks": [
 {
 "type": "command",
 "command": "'${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd' session-end"
 }
 ]
 }
 ]
 }
}
```

```bash
# run-hook.cmd (Polyglot example)
@echo off
set "BASH_PATH=bash"
where git >nul 2>nul && for /f "delims=" %%i in ('where git') do set "GIT_EXE=%%i"
# ... logic to find bash and run script
```



