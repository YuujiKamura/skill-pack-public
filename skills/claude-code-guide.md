---
name: claude-code-guide
description: Claude Code初心者向けガイド。(1) CLAUDE.mdメモリ機能、(2) カスタムコマンド作成、(3) Agent Skills設定、(4) Hooks自動化、(5) MCPサーバー連携、(6) ワークフロー最適化（think/TDD/ビジュアル）。Claude Code、設定、CLAUDE.md、コマンド、スキル、フック、MCP、サブエージェント、think、ultrathink、初心者、使い方と言われた時に使用。
---

# Claude Code 初心者向けガイド

> 出典: Qiita記事「Claude Code初心者向けガイド」
> https://qiita.com/westtail/items/15767bbabf15a6db381d

---

## 1. メモリ機能（CLAUDE.md）

Claudeにプロジェクト固有のコンテキストを与えるファイル。**毎回のプロンプトに自動で含まれる**。

### 保存場所と優先度

| 種類 | パス | 用途 | 適用範囲 |
|------|------|------|----------|
| プロジェクトメモリ | `./CLAUDE.md` | プロジェクト固有のルール | そのプロジェクトのみ |
| ユーザーメモリ | `<CLAUDE_HOME>/CLAUDE.md` | 個人の作業スタイル | 全プロジェクト |

### 具体的な書き方例

```markdown
## 技術スタック
- バックエンド: Rails 8
- DB: MySQL
- フロントエンド: React + TypeScript

## コーディング規約
- エラーメッセージは日本語で統一
- 関数は20行以内に収める
- 型定義は必須（any禁止）

## 禁止事項
- console.logをコミットしない
- 環境変数を直接書かない
- テストなしでマージしない

## ビルド・テストコマンド
- ビルド: `npm run build`
- テスト: `npm test`
- リント: `npm run lint`
```

### 記載のコツ

| NG | OK | 理由 |
|----|-----|------|
| きれいなコードを書く | 関数は20行以内 | 具体的で検証可能 |
| 適切なエラーハンドリング | try-catchで必ずログ出力 | 曖昧さがない |
| 長い説明文... | 箇条書きで端的に | Claudeが効率的に処理 |

---

## 2. コマンド機能

カスタムコマンドでワークフローを自動化。`/コマンド名`で呼び出し。

### 保存場所

```
.claude/commands/           # プロジェクト固有
<CLAUDE_HOME>/commands/         # 全プロジェクト共通
├── commit.md              # /commit で呼び出し
├── review.md              # /review で呼び出し
└── fix-issue.md           # /fix-issue で呼び出し
```

### `!`記法（動的情報取得）

コマンド内で`!`を使うとシェルコマンドの出力を動的に取得:

```markdown
# commit.md
以下の変更内容を確認し、適切なコミットメッセージを提案してください:

## ステージングされた変更
!git diff --staged

## 現在のブランチ
!git branch --show-current

## 最近のコミット履歴（参考）
!git log --oneline -5
```

### 実用的なコマンド例

**fix-issue.md** - Issue番号から修正:
```markdown
# Issue修正

以下のIssueを修正してください:
!gh issue view $ARGUMENTS

## 手順
1. 関連コードを特定
2. 修正を実装
3. テストを追加
4. PRを作成
```

**review.md** - コードレビュー:
```markdown
# コードレビュー

以下の変更をレビューしてください:
!git diff main...HEAD

## チェック観点
- バグの可能性
- パフォーマンス問題
- セキュリティリスク
- 可読性
```

### 組み込みコマンド

| コマンド | 説明 |
|----------|------|
| `/clear` | 会話をクリア（コンテキストリセット） |
| `/help` | ヘルプ表示 |
| `/compact` | 会話を要約して圧縮 |
| `/cost` | トークン使用量を表示 |

---

## 3. Agent Skills

スキルはClaudeの専門知識を拡張するファイル。**キーワードに反応して自動的に読み込まれる**。

### 保存場所

```
<CLAUDE_HOME>/skills/
└── skill-name/
    └── SKILL.md
```

### YAML frontmatter（必須）

```yaml
---
name: skill-name
description: スキルの概要。(1) ユースケース1、(2) ユースケース2。キーワード1、キーワード2と言われた時に使用。
---

# スキル本文

ここに詳細な知識やルールを記載...
```

### トークン消費の段階設計

```
┌─────────────────────────────────────────┐
│ description（常に読み込み）             │  ← 軽量に！
├─────────────────────────────────────────┤
│ SKILL.md本文（スキル発火時のみ）        │  ← 詳細はここ
├─────────────────────────────────────────┤
│ 関連ファイル（明示的な参照時のみ）      │  ← 必要な時だけ
└─────────────────────────────────────────┘
```

### descriptionの書き方

```yaml
# NG: 短すぎる
description: Gmail APIでメール作成。

# OK: ユースケース + キーワード
description: Gmail API操作。(1) メール下書き作成・送信、(2) 受信メール検索・フィルタリング、(3) 添付ファイル付きメール作成。Gmail、メール、下書き、送信、検索、返信と言われた時に使用。
```

---

## 4. サブエージェント

Claudeは複雑なタスクをサブエージェントに委任できる。**独立したコンテキスト**で動作。

### 種類と特徴

| エージェント | 権限 | 用途 | 起動方法 |
|--------------|------|------|----------|
| **Explore** | 読取のみ | コードベース探索、ファイル検索 | 自動 |
| **Plan** | 読取のみ | 実装計画の策定 | `EnterPlanMode` |

### Exploreエージェント

コードベースを探索する専門エージェント:

```
ユーザー: 「認証機能はどこに実装されている？」
     ↓
Claude: Exploreエージェントを起動
     ↓
Explore: src/auth/, middleware/auth.ts を特定
     ↓
Claude: 結果を要約して返答
```

### Planエージェント

実装計画を策定するエージェント:

```
ユーザー: 「ログイン機能を追加して」
     ↓
Claude: Planモードに入る
     ↓
Plan: コードベース調査 → 計画作成
     ↓
ユーザー: 計画を承認
     ↓
Claude: 実装開始
```

### 起動のコツ

- **明示的に依頼**: 「コードベースを探索して」「計画を立てて」
- **複雑なタスク**: 自動的にサブエージェントが起動される
- **独立コンテキスト**: メイン会話の文脈は引き継がない

---

## 5. Hooks

ツール実行前後に自動処理を挿入。`.claude/settings.json`で設定。

### Hookタイプ

| Hook | タイミング | 用途 |
|------|------------|------|
| `PreToolUse` | ツール実行前 | 危険コマンドの検証 |
| `PostToolUse` | ツール実行後 | 自動フォーマット |
| `UserPromptSubmit` | ユーザー入力送信時 | 入力の前処理 |
| `Stop` | 会話終了時 | クリーンアップ |
| `SessionStart` | セッション開始時 | 初期化処理 |

### 設定例

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "echo 'Bash command executing...'"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit",
        "command": "npx prettier --write $FILE_PATH"
      },
      {
        "matcher": "Write",
        "command": "npx prettier --write $FILE_PATH"
      }
    ],
    "Stop": [
      {
        "command": "echo 'Session ended' >> <CLAUDE_HOME>/session.log"
      }
    ]
  }
}
```

### 実用的なHook例

**自動リント（PostToolUse）**:
```json
{
  "matcher": "Edit",
  "command": "eslint --fix $FILE_PATH"
}
```

**危険コマンド警告（PreToolUse）**:
```json
{
  "matcher": "Bash",
  "command": "if echo '$COMMAND' | grep -q 'rm -rf'; then echo 'DANGER: rm -rf detected' >&2; exit 1; fi"
}
```

---

## 6. MCPサーバー

外部ツールをClaudeに接続するプロトコル。

### スコープ別設定

| スコープ | ファイル | 共有 | 用途 |
|----------|----------|------|------|
| **user** | `~/.claude.json` | なし | 個人の全プロジェクト |
| **project** | `.mcp.json` | チームで共有可 | プロジェクト固有 |
| **local** | なし | なし | 一時的なテスト |

### 設定例（~/.claude.json）

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-filesystem", "/path/to/allowed/dir"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_xxxx"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost/db"
      }
    }
  }
}
```

### プロジェクト固有（.mcp.json）

```json
{
  "mcpServers": {
    "project-api": {
      "command": "node",
      "args": ["./tools/mcp-server.js"],
      "env": {
        "API_KEY": "${PROJECT_API_KEY}"
      }
    }
  }
}
```

### よく使うMCPサーバー

| サーバー | 用途 |
|----------|------|
| `@anthropic/mcp-server-filesystem` | ファイル操作 |
| `@anthropic/mcp-server-github` | GitHub操作 |
| `@anthropic/mcp-server-postgres` | PostgreSQL操作 |
| `@anthropic/mcp-server-memory` | 永続メモリ |

---

## 7. ワークフロー最適化

### thinkモード（深い思考）

複雑な問題で深い思考を促す。**予算が増える = より長く考える**。

| コマンド | 強度 | 使い所 |
|----------|------|--------|
| `think` | 基本 | 軽い検討 |
| `think hard` | 中 | 設計判断 |
| `think harder` | 高 | 複雑な問題 |
| `ultrathink` | 最大 | アーキテクチャ設計 |

**使用例**:
```
「このアーキテクチャについてultrathinkして」
「認証フローをthink hardして設計して」
「このバグの原因をthink harderして特定して」
```

### TDD（テスト駆動開発）

```
1. テスト作成を依頼
   └→ 「ログイン機能のテストを先に書いて」

2. テストが失敗することを確認
   └→ npm test → RED

3. コミット
   └→ git commit -m "Add login tests"

4. 実装を依頼
   └→ 「テストが通るように実装して」

5. 反復
   └→ GREEN になるまで繰り返し
```

### ビジュアル反復

UIの実装で効果的:

```
1. スクリーンショットをClaudeに提供
   └→ 「このデザインを実装して」

2. 実装結果を確認
   └→ ブラウザでプレビュー → スクショ

3. 差分をフィードバック
   └→ 「ボタンの色が違う。青 #3B82F6 に」

4. 完成まで反復
```

### マルチClaude（並列作業）

```
ターミナル1: メイン作業
├── 機能実装中...

ターミナル2: レビュー用（別セッション）
├── /clear
├── 「ターミナル1の実装をレビューして」
└── 独立した視点でレビュー

ターミナル3: テスト用（別セッション）
├── /clear
├── 「この機能のテストを書いて」
└── 並行してテスト作成
```

**Git worktree活用**:
```bash
# 独立した作業ディレクトリを作成
git worktree add ../feature-branch feature-branch

# 別ターミナルで作業
cd ../feature-branch
claude
```

---

## 8. セキュリティ注意事項

### 信頼できるソースのみ使用

- **MCPサーバー**: 公式または信頼できるソースから
- **スキル**: 内容を確認してから追加
- **コマンド**: 実行前にコードを確認

### ファイル確認

```bash
# 自動生成されたコードは実行前に確認
git diff --staged

# 危険なパターンをチェック
grep -r "rm -rf" .
grep -r "eval(" .
```

### ネットワーク監視

MCPサーバーがどこに通信しているか把握:

```bash
# macOS
sudo lsof -i -P | grep node

# Linux
netstat -tulpn | grep node
```

### 環境変数の管理

```bash
# .envは.gitignoreに追加
echo ".env" >> .gitignore

# 機密情報はCLAUDE.mdに書かない
# NG: API_KEY=sk-xxxx
# OK: 環境変数 $API_KEY を使用
```

---

## クイックリファレンス

| やりたいこと | 方法 |
|--------------|------|
| プロジェクトルールを設定 | `./CLAUDE.md`に記載 |
| 個人設定を全プロジェクトに適用 | `<CLAUDE_HOME>/CLAUDE.md`に記載 |
| よく使う操作を自動化 | `.claude/commands/`にコマンド作成 |
| 専門知識を追加 | `<CLAUDE_HOME>/skills/`にスキル作成 |
| ツール実行を自動化 | `.claude/settings.json`にHook設定 |
| 外部サービス連携 | MCPサーバーを設定 |
| 深く考えさせる | `think` → `think hard` → `ultrathink` |
| 計画を立てさせる | 「計画を立てて」または`EnterPlanMode` |
| コンテキストをリセット | `/clear` |
| 並列作業 | 別ターミナルで新しいClaude起動 |

---

## 関連スキル

| スキル | 内容 | 関係 |
|--------|------|------|
| `agent-best-practices` | エージェント活用のベストプラクティス | 補完（より高度な使い方） |
| `skill-management` | スキル作成・管理の詳細 | 補完（スキル作成に特化） |

---

## 参考資料

- [Claude Code公式ドキュメント](https://docs.anthropic.com/en/docs/claude-code)
- [Qiita: Claude Code初心者向けガイド](https://qiita.com/westtail/items/15767bbabf15a6db381d)
- [Cursor Agent Best Practices](https://cursor.com/ja/blog/agent-best-practices)
