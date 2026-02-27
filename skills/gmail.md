---
name: gmail
description: Gmail API操作。(1) メール下書き作成・送信、(2) 受信メール検索・フィルタリング、(3) 添付ファイル付きメール作成、(4) ラベル管理・スレッド取得、(5) 返信・転送の自動化。Gmail、メール、下書き、送信、検索、添付、返信、転送、受信トレイ、ラベル、連絡、通知と言われた時に使用。
allowed-tools: Read, Grep, Glob, Bash(python:*)
---

# Gmail API スキル集

## 必要なスコープ

```python
SCOPES = [
    'https://www.googleapis.com/auth/gmail.readonly',    # メール読み取り
    'https://www.googleapis.com/auth/gmail.compose',     # メール作成・下書き
    'https://www.googleapis.com/auth/gmail.send',        # メール送信
    'https://www.googleapis.com/auth/drive',             # Drive（添付ファイル用）
]
```

## トークンファイルの場所

※トークンファイルのパスは別途管理

---

## Gmail APIサービス取得

```python
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build

def get_gmail_service():
    """Gmail APIサービスを取得"""
    token_path = "path/to/gmail_token.json"  # 実際のパスに置き換え
    creds = Credentials.from_authorized_user_file(token_path, SCOPES)
    return build('gmail', 'v1', credentials=creds)
```

---

## 下書きメール作成（テキストのみ）

```python
import base64
from email.mime.text import MIMEText

def create_draft_simple(gmail_service, to, subject, body, cc=None):
    """シンプルなテキストメールの下書きを作成"""
    message = MIMEText(body, 'plain', 'utf-8')
    message['to'] = to
    message['subject'] = subject
    if cc:
        message['cc'] = cc

    raw_message = base64.urlsafe_b64encode(
        message.as_bytes()
    ).decode('utf-8')

    draft = gmail_service.users().drafts().create(
        userId='me',
        body={'message': {'raw': raw_message}}
    ).execute()

    return draft

# 使用例
gmail_service = get_gmail_service()
draft = create_draft_simple(
    gmail_service,
    to='example@city.kumamoto.lg.jp',
    subject='【案件名】施工前資料の提出',
    body='お世話になっております。\n\n資料を提出します。'
)
print(f"下書きID: {draft['id']}")
```

---

## 下書きメール作成（添付ファイル付き）

```python
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
from email import encoders

def create_draft_with_attachments(
    gmail_service,
    drive_service,
    to,
    subject,
    body,
    attachments,
    cc=None
):
    """
    添付ファイル付きメールの下書きを作成

    attachments: [{'file_id': 'xxx', 'filename': 'yyy.pdf'}, ...]
    """
    message = MIMEMultipart()
    message['to'] = to
    message['subject'] = subject
    if cc:
        message['cc'] = cc

    message.attach(MIMEText(body, 'plain', 'utf-8'))

    for attachment in attachments:
        file_id = attachment['file_id']
        filename = attachment['filename']

        # Drive APIでファイルをダウンロード
        request = drive_service.files().get_media(fileId=file_id)
        file_content = request.execute()

        # MIMEタイプを設定
        if filename.endswith('.pdf'):
            part = MIMEBase('application', 'pdf')
        elif filename.endswith('.xlsx'):
            part = MIMEBase('application', 'vnd.openxmlformats-officedocument.spreadsheetml.sheet')
        else:
            part = MIMEBase('application', 'octet-stream')

        part.set_payload(file_content)
        encoders.encode_base64(part)

        # 日本語ファイル名対応（RFC 2231）
        encoded_filename = ('utf-8', '', filename)
        part.add_header(
            'Content-Disposition',
            'attachment',
            filename=encoded_filename
        )
        message.attach(part)

    raw_message = base64.urlsafe_b64encode(
        message.as_bytes()
    ).decode('utf-8')

    draft = gmail_service.users().drafts().create(
        userId='me',
        body={'message': {'raw': raw_message}}
    ).execute()

    return draft
```

---

## 既存の下書きを削除

```python
def delete_draft(gmail_service, draft_id):
    """下書きを削除"""
    gmail_service.users().drafts().delete(
        userId='me',
        id=draft_id
    ).execute()
```

---

## 下書き一覧を取得

```python
def list_drafts(gmail_service, max_results=10):
    """下書き一覧を取得"""
    results = gmail_service.users().drafts().list(
        userId='me',
        maxResults=max_results
    ).execute()

    return results.get('drafts', [])
```

---

## 下書きの内容を取得

```python
def get_draft_content(gmail_service, draft_id):
    """下書きの詳細を取得"""
    draft = gmail_service.users().drafts().get(
        userId='me',
        id=draft_id,
        format='full'
    ).execute()

    return draft
```

---

## 下書きを送信

```python
def send_draft(gmail_service, draft_id):
    """下書きを送信"""
    result = gmail_service.users().drafts().send(
        userId='me',
        body={'id': draft_id}
    ).execute()

    return result
```

---

## メール検索

```python
def search_emails(gmail_service, query, max_results=10):
    """
    メールを検索

    query例:
        "in:sent to:example@gmail.com"  # 送信済み
        "from:example@gmail.com"         # 差出人で検索
        "subject:報告書"                  # 件名で検索
        "has:attachment filename:pdf"    # 添付ファイル
        "after:2024/01/01"               # 日付指定
    """
    results = gmail_service.users().messages().list(
        userId='me',
        q=query,
        maxResults=max_results
    ).execute()

    return results.get('messages', [])
```

---

## メール本文を取得（マルチパート対応）

```python
def get_email_body(gmail_service, message_id):
    """メール本文を取得（再帰的マルチパート対応）"""
    message = gmail_service.users().messages().get(
        userId='me',
        id=message_id,
        format='full'
    ).execute()

    def extract_body(payload):
        if 'body' in payload and payload['body'].get('data'):
            return base64.urlsafe_b64decode(payload['body']['data']).decode('utf-8')
        if 'parts' in payload:
            for part in payload['parts']:
                if part['mimeType'] == 'text/plain':
                    if part['body'].get('data'):
                        return base64.urlsafe_b64decode(part['body']['data']).decode('utf-8')
                result = extract_body(part)
                if result:
                    return result
        return None

    return extract_body(message['payload'])
```

---

## 注意事項

1. **認証トークンの更新**: トークンが期限切れの場合は `gmail_oauth2_setup_extended.py` を実行して再認証
2. **添付ファイルサイズ**: Gmail APIの制限は25MB（Base64エンコード後は約35MB）
3. **日本語ファイル名**: RFC 2231形式でエンコードすることで文字化けを防止
4. **下書きIDの保存**: 更新・削除する場合は下書きIDを控えておく

---

## 参考モジュール

- `scripts/utils/create_report_email_draft.py` - 報告メール下書き作成
- `scripts/utils/gmail_oauth2_setup_extended.py` - OAuth認証セットアップ
- `scripts/utils/check_gmail.py` - Gmail接続確認
- `scripts/utils/search_reference_email.py` - メール検索
