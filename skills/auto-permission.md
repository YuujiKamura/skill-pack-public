---
name: auto-permission
description: 実装作業開始前に必要な許可設定を先回りで自動追加。(1) 実装計画からBashコマンドを予測、(2) CLAUDE.mdの許可設定を自動更新、(3) npm/node/git等の一括許可、(4) 作業中断を防止、(5) プロジェクト固有のコマンド追加。許可、承認、permission、自動化、中断防止、設定、npm install、ビルド、コマンドと言われた時に使用。
---

# Auto Permission - 許可設定の先回り自動追加

## 概要

実装作業を開始する前に、必要になるBashコマンドを予測してCLAUDE.mdの許可設定に自動追加する。これにより作業中の承認待ちによる中断を防止する。

## 使用タイミング

1. **実装計画が確定した直後**（EnterPlanMode → ExitPlanMode の後）
2. **新しいプロジェクトで作業を始める前**
3. **npm install等で中断が発生した時**

## 手順

### Step 1: 計画から必要コマンドを予測

実装計画の内容から、必要になるBashコマンドを列挙：

| 作業内容 | 必要なコマンド |
|---------|---------------|
| パッケージ追加 | `npm install`, `npm run` |
| TypeScriptビルド | `tsc`, `npx tsc` |
| Git操作 | `git add`, `git commit`, `git push` |
| ディレクトリ作成 | `mkdir` |
| ファイル操作 | `rm`, `mv`, `cp` |
| テスト実行 | `npm test`, `vitest` |
| Node実行 | `node` |

### Step 2: CLAUDE.mdの許可設定を確認・更新

CLAUDE.mdの `## 許可されたコマンド` セクションを読み取り、不足しているコマンドを追加：

```markdown
## 許可されたコマンド

\`\`\`
Bash(mkdir:*)
Bash(rm:*)
Bash(mv:*)
Bash(cp:*)
Bash(ls:*)
Bash(cat:*)
Bash(git:*)
Bash(git add:*)
Bash(git commit:*)
Bash(git push:*)
Bash(git checkout:*)
Bash(git merge:*)
Bash(git stash:*)
Bash(git rm:*)
Bash(npm:*)
Bash(npm install:*)
Bash(npm run:*)
Bash(node:*)
Bash(npx:*)
Bash(tsc:*)
\`\`\`
```

### Step 3: 編集実行

```
1. Read: CLAUDE.md
2. 許可設定セクションを特定
3. 不足コマンドを追加
4. Edit: CLAUDE.md を更新
```

## 標準許可セット（コピペ用）

### 最小セット（基本操作）
```
Bash(mkdir:*)
Bash(git add:*)
Bash(git commit:*)
Bash(git push:*)
Bash(npm:*)
Bash(node:*)
Bash(npx:*)
```

### フルセット（開発作業全般）
```
Bash(mkdir:*)
Bash(rm:*)
Bash(mv:*)
Bash(cp:*)
Bash(ls:*)
Bash(cat:*)
Bash(head:*)
Bash(tail:*)
Bash(which:*)
Bash(echo:*)
Bash(git:*)
Bash(git add:*)
Bash(git commit:*)
Bash(git push:*)
Bash(git checkout:*)
Bash(git branch:*)
Bash(git merge:*)
Bash(git stash:*)
Bash(git rm:*)
Bash(git log:*)
Bash(git status:*)
Bash(git diff:*)
Bash(npm:*)
Bash(npm install:*)
Bash(npm run:*)
Bash(npm test:*)
Bash(node:*)
Bash(npx:*)
Bash(tsc:*)
```

## 注意事項

- `Bash(rm:*)` は破壊的操作なので、必要な場合のみ追加
- プロジェクト固有のコマンド（`cargo`, `python` 等）は必要に応じて追加
- セキュリティ上重要なコマンド（`sudo`, `curl` 等）は原則追加しない

## 変更履歴
- 2026-01-17: 初版作成
