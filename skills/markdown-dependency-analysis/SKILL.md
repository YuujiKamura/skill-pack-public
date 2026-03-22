---
name: markdown-dependency-analysis
description: "CLI & Tooling. (Inter-Knowledge Dependency Mapping) Use when user mentions: CLI, script, automation, tool, plugin, extension, config."
---

# CLI & Tooling

Patterns: 1

## 1. Inter-Knowledge Dependency Mapping

Tracking and bundling cross-references between skills, memory files, and project rules to ensure knowledge integrity across systems.

### Steps

1. Identify three primary dependency types: `MarkdownLink` (standard `[name](path)`), `SkillRef` (backticked skill names), and `ProjectPath` (absolute or relative filesystem paths).
2. Construct a directed graph of knowledge nodes to detect 'parent' skills that require specific memory files or sub-skills to be functional.
3. Extract references using regex patterns that look for context-specific phrases like '詳細はスキル `skill-name`' or 'See [memory](path)'.
4. When exporting a bundle, recursively include all dependent files to prevent 'broken link' errors when the knowledge is imported into a new environment.

### Examples

```rust
pub enum DepType {
 MarkdownLink,
 SkillRef,
 ProjectPath,
}

pub struct SkillDependency {
 pub from: String,
 pub to: String,
 pub dep_type: DepType,
 pub line: usize,
}
```



