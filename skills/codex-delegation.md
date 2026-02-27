---
name: codex-delegation
description: OpenAI Codex CLIへのMCP経由タスク委譲。(1) 大量リファクタリングの並列実行、(2) ボイラープレート生成、(3) コンテキスト節約のための外部エージェント活用、(4) Multi-Turn段階実装。Codex、委譲、並列、リファクタリング、ボイラープレートと言われた時に使用。
triggers:
  - codex
  - 委譲
  - 並列コーディング
  - リファクタリング大量
  - ボイラープレート
---

# Codex MCP 委譲スキル

## セットアップ

- MCP設定: `~/.claude.json` のグローバル `mcpServers` に `codex mcp-server` 登録済み
- エージェント定義: `<CLAUDE_HOME>/agents/codex.md`
- Codex CLI: `codex-cli 0.98.0` (npm global)

## ツール

| ツール | 用途 |
|--------|------|
| `mcp__codex__codex` | 新規セッション開始 |
| `mcp__codex__codex-reply` | 既存セッション続行 |

## 呼び出しパターン

### Single-Turn
```
mcp__codex__codex(
  prompt: "src/utils/date.tsにdate formatting utilityを作成。ISO8601パース、和暦変換、相対時間表示の3関数。",
  approval-policy: "never",
  sandbox: "workspace-write"
)
```

### Multi-Turn
```
# Turn 1
result = mcp__codex__codex(prompt: "型定義を作成...", ...)
# Turn 2
mcp__codex__codex-reply(session_id: result.session_id, prompt: "実装を追加...")
# Turn 3
mcp__codex__codex-reply(session_id: result.session_id, prompt: "テストを書け...")
```

## 委譲判断

### する（Codex向き）
- 10+ファイルのリネーム・リファクタリング
- CRUDボイラープレート大量生成
- フレームワーク移行（import書き換え等）
- コンテキストウィンドウ圧迫時

### しない（Claude向き）
- アーキテクチャ設計・判断
- テスト設計・TDD
- セキュリティレビュー
- 3ファイル以下の変更

## 注意

- Codexはコンテキスト非共有 → プロンプトに要件・パス・制約を全て書く
- 必ずfeature branchで作業、`git diff`で検証後コミット
- Codexの出力はClaude側で検証してからマージ
