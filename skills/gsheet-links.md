---
name: gsheet-links
description: Google DriveからスプレッドシートのリンクをLINKS.mdとして収集。.gsheetファイルはDrive APIで取得。
allowed-tools: Read, Glob, Bash(python:*), Write
---

# Google スプレッドシートリンク収集

## 概要

Google Drive上の`.gsheet`ファイル（スプレッドシートへのリンク）はローカルで直接読めない。
Drive APIを使ってリンクを取得し、`LINKS.md`としてまとめる。

## スクリプト

`scripts/get_gsheet_links.py`

## 使用方法

```bash
# 基本
python scripts/get_gsheet_links.py "フォルダ名"

# ファイル出力
python scripts/get_gsheet_links.py "フォルダ名" -o LINKS.md
```

## 認証設定

### ローカル環境

clasp認証を使用（推奨）:
```bash
npm install -g @google/clasp
clasp login
# ~/.clasprc.json が作成される
```

### クラウド/CI環境

サービスアカウントを使用:
```bash
export GOOGLE_SERVICE_ACCOUNT_FILE=/path/to/service-account.json
```

### サービスアカウント作成手順

1. [Google Cloud Console](https://console.cloud.google.com/) でプロジェクト作成
2. Drive API を有効化
3. サービスアカウント作成 → JSONキーをダウンロード
4. 対象フォルダをサービスアカウントのメールアドレスと共有

## gitignore

認証情報をコミットしないよう`.gitignore`に追加:
```
credentials.json
token.json
*-service-account.json
.clasprc.json
```

## 依存パッケージ

```bash
pip install google-api-python-client google-auth-oauthlib
```
