---
name: tool-design
description: "ツール設計。(メタデータ駆動の現状分析 (Metadata-Driven Status Analysis)、環境依存問題のシェルレベル解決 (Shell-Level Environment Troubleshooting)、スキーマ先行の永続化設計 (Schema-First Persistence Design)、外部知見の「スキル」カプセル化 (External Knowledge Skill-ification)、検証駆動型インクリメンタル実装 (Validation-Driven Incremental Implementation)) CLI、スキル、自動化、ツール設計と言われた時に使用。"
---

# ツール設計

会話数: 9 | パターン数: 5

## 1. メタデータ駆動の現状分析 (Metadata-Driven Status Analysis)

ツール自身の履歴ファイル（history.jsonl）やプロジェクト固有のメタデータを走査し、設計やトラブルシューティングの前提条件を特定するパターン。

### 手順

1. ~/.claude/history.jsonl や projects/ フォルダ内のJSONLを確認し、過去のコンテキストを特定する
2. grep や python ワンライナーを用いて、ログから特定の役割（role）やメッセージを抽出する
3. Cargo.toml や package.json を読み込み、依存関係とプロジェクト構造を把握する

出現頻度: 9回

## 2. 環境依存問題のシェルレベル解決 (Shell-Level Environment Troubleshooting)

ツール実行時のエラー（パスの不通、エイリアス競合、フラグの誤認）を、シェル設定ファイル（.bashrc等）の修正によって根本解決するパターン。

### 手順

1. which, where, alias コマンドで実行バイナリの実体と競合状況を調査する
2. env | grep で関連する環境変数（CLAUDE_PRINT_MODE等）の状態を確認する
3. .bashrc や .profile に修正コードを追記し、source コマンドで即時反映・検証する

出現頻度: 5回

## 3. スキーマ先行の永続化設計 (Schema-First Persistence Design)

データの整合性を保ちつつ解析ログや中間状態を保存するため、実装前に型定義（TypeScript/Rust）とストア（IndexedDB等）を設計するパターン。

### 手順

1. types.ts や types.rs で永続化データのインターフェースを厳密に定義する
2. メインのデータストア（StockItem等）からログ用ストア（analysisLogs）を分離し、独立性を確保する
3. 既存データとの整合性を保つためのアップグレード関数やマイグレーション手順を定義する

出現頻度: 4回

## 4. 外部知見の「スキル」カプセル化 (External Knowledge Skill-ification)

外部ドキュメント（Qiita等）やチャット履歴から抽出した知見を、ツールの拡張機能である「スキル」として構造化・再利用可能にするパターン。

### 手順

1. ~/.claude/skills/ 配下に専用ディレクトリを作成する
2. SKILL.md を作成し、YAMLフロントマターで名前や目的を定義する
3. 具体的な手順やベストプラクティスをMarkdown形式で記述し、エージェントが参照可能な状態にする

出現頻度: 3回

## 5. 検証駆動型インクリメンタル実装 (Validation-Driven Incremental Implementation)

計画に基づき実装を進める際、各ステップで静的解析やビルドツールを頻繁に実行し、デグレードを防ぐパターン。

### 手順

1. npx tsc --noEmit による型チェックを各ファイル変更後に行う
2. npm run build や cargo build でビルド可能性を継続的に検証する
3. git add と git commit --no-verify を組み合わせ、論理的な単位で変更を記録する

出現頻度: 6回


