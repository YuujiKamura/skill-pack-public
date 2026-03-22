---
name: toolchain-inventory
description: "ユーザー独自ツールチェーンの棚卸し・スキル化。(1) ~/配下の自作プロジェクトをスキャン、(2) スキル未作成のツールを検出、(3) 各ツールのスキルを自動生成、(4) 廃止・重複ツールの整理。ツールチェーン、棚卸し、ツール一覧、自作ツール、インベントリと言われた時に使用。"
---

# ツールチェーン棚卸し

## 目的

ユーザーが自作したCLI・Webアプリ・スクリプト群を把握し、各ツールの使い方をスキル化する。
スキル化されていないツールは、ユーザーが名前で呼んでもAIがピンとこない。

## 手順

### 1. プロジェクト検出

```bash
# Rust / Node / Python プロジェクトを検出
ls ~/*/Cargo.toml ~/*/package.json ~/*/pyproject.toml 2>/dev/null
# スタンドアロンスクリプト群
ls ~/weather_map_updater/*.py ~/scripts/*.sh 2>/dev/null
```

### 2. スキルカバレッジ確認

```bash
# 各プロジェクトに対応するスキルがあるか確認
for dir in ~/*/Cargo.toml; do
  name=$(basename $(dirname "$dir"))
  if ls ~/.claude/skills/ | grep -qi "$name"; then
    echo "OK: $name"
  else
    echo "MISSING: $name"
  fi
done
```

### 3. ツール情報の収集（MISSINGのもの）

各ツールについて以下を読む:
- `Cargo.toml` / `package.json`: name, description
- `README.md`: 概要
- `src/main.rs` の clap 定義: サブコマンド・オプション
- 最近のgitログ: 現在アクティブか

### 4. スキル生成

以下のテンプレートで `~/.claude/skills/{tool-name}.md` を作成:

```markdown
---
name: {tool-name}
description: "{ツール名}の使い方。{一文説明}。{キーワード1}、{キーワード2}と言われた時に使用。"
---

# {ツール名}

## 場所・実行
- リポジトリ: `~/{dir}/`
- 実行: `{実行コマンド}`

## 主な機能
- {機能1}
- {機能2}

## よく使うコマンド
| コマンド | 用途 |
|----------|------|
| `{cmd1}` | {説明} |
```

### 5. 整理判断

以下に該当するプロジェクトはスキル化不要。ユーザーに確認:
- **廃止**: 長期間更新なし、後継ツールあり（例: google-photos-fetch → API廃止済み）
- **重複**: 同一目的の複数バリアント（例: tonsuu-checker系が5つ）
- **実験**: 一時的なPoC、本番利用なし

## 既知のツール分類

| カテゴリ | ツール |
|----------|--------|
| 写真AI | photo-ai-rust, photo-tagger, GASPhotoAIManager |
| AI基盤 | cli-ai-analyzer |
| 積載量推定 | tonsuu-checker, tonsuu-core, TonSuuChecker |
| 書類生成 | SekouTaiseiMaker, ShoruiChecker |
| ユーティリティ | dropbox-fetch, sshot, folder-watcher, skill-miner |
| 天気 | weather_map_updater (Python scripts) |
| GUI | claudecodeui, desktop-viewer, tauri-gui-shell |

## 注意

- スキルのdescriptionの**キーワード**が重要。ユーザーはツール名の日本語読み（「スキルマイナー」「フォトタガー」）で呼ぶことが多い
- MEMORYにツールの場所を書くな。スキルに書け。MEMORYは常時ロードで行数が貴重
- 廃止判定は `git log -1 --format=%ci` で最終更新日を見る
