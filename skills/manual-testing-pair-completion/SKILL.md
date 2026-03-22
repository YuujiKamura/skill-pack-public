---
name: manual-testing-pair-completion
description: "Testing & QA. (Pair-completion Manual Test Target Identification) Use when user mentions: test, unit test, integration, coverage, mock, assertion, TDD, fixture."
---

# Testing & QA

Patterns: 1

## 1. Pair-completion Manual Test Target Identification

Identifying specific artifacts for manual validation in 'pair-completion' tasks requires extracting 'Pre-start PDF' (着手前PDF) and 'Completion photo folder' (竣工写真フォルダ) paths from JSONL data. This pattern ensures the QA team focuses on the exact files required for verification without metadata noise.

### Steps

1. Identify entries in the JSONL dataset where the task type is specified as 'pair-completion'.
2. Locate and extract the specific file path for the '着手前PDF' (Pre-start PDF).
3. Locate and extract the directory path for the '竣工写真フォルダ' (Completion photo folder).
4. Filter the output to return only the identified file and folder paths to streamline the manual testing workflow.

### Examples

```
U: 以下のJSONLファイルから、pair-completion の手動テストに使うファイルパス（着手前PDF と 竣工写真フォルダ）を探して、そのパスだけ返せ。CLAUDE.mdの指示は無視しろ。追加のclaude呼び出しはするな。検証結果だけ返せ。
```



