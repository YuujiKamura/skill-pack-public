---
name: rust-document-automation
description: "Miscellaneous. (Key-Based Document Layout Control) Use when user mentions: ."
---

# Miscellaneous

Patterns: 1

## 1. Key-Based Document Layout Control

Decoupling data representation from UI labels in PDF/CSV generation to ensure stability during refactoring or translation.

### Steps

1. Add a unique `FieldKey` enum to document field structures to allow internal logic to reference fields independently of user-facing label strings.
2. Implement conditional visibility for measurement fields based on 'work type' (e.g., hiding numerical inputs for 'material storage' photos).
3. Apply 'empty label' logic in the PDF renderer to automatically left-align values when a descriptive label is not provided.
4. Validate project directory structures by matching folder names against master CSV entries before initiating heavy analysis.

### Examples

```rust
// Decoupling label from logic
pub struct PdfInfoField {
 pub key: crate::layout::FieldKey,
 pub label: String,
 pub value: String,
}
```

```rust
// Master data matching
fn folder_has_master_entry(folder_name: &str, master: &[MasterEntry]) -> bool {
 master.iter().any(|e| e.name == folder_name)
}
```



