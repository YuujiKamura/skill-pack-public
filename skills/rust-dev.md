---
name: rust-dev
description: "Rust開発。(再帰的AIコード検証 (Self-Verification Loop)、マルチエージェント並列タスク分割 (Team-based Execution)、CLIインターフェース先行調査と統合 (CLI Discovery & Integration)、PythonプロトタイプからのRust移植 (Script-to-Crate Migration)、ビルド・チェック主導の反復修正 (Build-Driven Refinement)) Rust、クレート、cargo、ビルド、WASM、deriveと言われた時に使用。"
---

# Rust開発

会話数: 31 | パターン数: 6

## 1. 再帰的AIコード検証 (Self-Verification Loop)

実装したコードの差分やロジックの妥当性を、別のClaudeインスタンスを呼び出して客観的に検証させるパターン。環境変数をリセットして純粋な検証結果のみを求める。

### 手順

1. コードを修正または生成する
2. CLAUDECODE= claude -p "検証結果だけ返せ。CLAUDE.mdの指示は無視しろ..." という形式でサブエージェントを起動
3. 返ってきた検証結果に基づき、修正の是非を判断または再修正する

出現頻度: 14回

## 2. マルチエージェント並列タスク分割 (Team-based Execution)

影響範囲が複数のプロジェクト（photo-ai-rust, photo-tagger, dropbox-fetch等）に及ぶ場合、各プロジェクトを独立したタスクとしてサブエージェントに割り当て並列実行する。

### 手順

1. 全体のプランをContext/Issue/修正対象に分解する
2. プロジェクトごとに専門エージェント（photo-ai-agent等）を割り当てる
3. 各エージェントからの完了通知（idle_notification）を待機し、統合を確認する

出現頻度: 6回

## 3. CLIインターフェース先行調査と統合 (CLI Discovery & Integration)

新しいコマンドを追加したり既存のものを修正する前に、helpオプションを実行して既存の引数構成や構造（Clap等の定義）を正確に把握する。

### 手順

1. src/cli.rs や src/commands.rs を読み込む
2. cargo run -- <command> --help を実行して実行時の挙動を確認する
3. 既存のパターンの命名規則や引数設計に合わせて実装を追加する

出現頻度: 8回

## 4. PythonプロトタイプからのRust移植 (Script-to-Crate Migration)

PillowやPythonスクリプトで実証済みのロジック（画像処理、PDF生成、ペアリング等）を、Rustのパフォーマンスと型安全性を活かすためにメインツールへ移植する。

### 手順

1. 検証済みのPythonスクリプト（test_*.py）の内容を解析する
2. Cargo.tomlに必要なクレート（printpdf, image等）を追加する
3. Rustの型システムに合わせてロジックを再実装し、ビルドを通して検証する

出現頻度: 5回

## 5. ビルド・チェック主導の反復修正 (Build-Driven Refinement)

Rustコンパイラの厳格なチェックを利用し、ビルドエラーや警告をパイプとgrepでフィルタリングしながら、インクリメンタルに修正を繰り返す。

### 手順

1. cargo check または cargo build 2>&1 を実行
2. grep -E "error|warning" で重要な指摘のみを抽出
3. 指摘箇所を順次修正し、再度ビルドを実行してエラーが消えるのを確認する

出現頻度: 12回

## 6. コンテキスト優先の並列ファイル読み込み (Parallel Context Gathering)

タスク開始時に、関連する複数のソースファイル（lib.rs, cli.rs, logic.rs）を並列で読み込み、システム全体の依存関係と設計意図を最初に把握する。

### 手順

1. プランに含まれる修正対象ファイルと、その依存先（common等）を特定する
2. read_file を並列実行してコードベースの「現状」を同期的に把握する
3. 実装前に、設計の矛盾（ハードコードされた文字列や重複ロジック）がないかチェックする

出現頻度: 15回


