---
name: drive-file-auto-update
description: Use when this skill's workflow applies
---

# Google Drive ファイルID自動更新スキル

Google DriveでPDFファイルを再アップロードするとfileIdが変わる問題に対応するためのスキル。
「ファイルID」「古いファイル」「最新のファイル」「ドライブ」「PDF更新」と言われた時に使用。

## 問題の背景

- Google Driveでファイルを再アップロードすると、新しいfileIdが発行される
- スプレッドシートに保存されているURLは古いfileIdを指し続ける
- GASのWeb App URLはスナップショットなので、ファイル情報も古いまま

## 解決策

### 1. GASに `getLatestFile` アクションを追加

古いfileIdから親フォルダを特定し、同名ファイルまたは最新のPDFを自動検出する。

```javascript
// doGet に追加
if (action === 'getLatestFile') {
  const fileId = e.parameter.fileId;
  if (!fileId) {
    return jsonResponse({ error: 'fileId is required' });
  }
  const result = getLatestFileInFolder(fileId);
  return jsonResponse(result);
}

// 関数を追加
function getLatestFileInFolder(oldFileId) {
  try {
    let oldFile;
    let oldFileName = null;
    let folderId = null;

    try {
      oldFile = DriveApp.getFileById(oldFileId);
      oldFileName = oldFile.getName();
      const parents = oldFile.getParents();
      if (parents.hasNext()) {
        folderId = parents.next().getId();
      }
    } catch (e) {
      return { error: 'Original file not found: ' + oldFileId };
    }

    if (!folderId) {
      return {
        success: true,
        fileId: oldFileId,
        fileName: oldFileName,
        folderId: null,
        modifiedTime: oldFile.getLastUpdated().toISOString(),
        isLatest: true
      };
    }

    const folder = DriveApp.getFolderById(folderId);
    const files = folder.getFilesByType(MimeType.PDF);

    let latestFile = null;
    let latestTime = null;
    let sameNameFile = null;

    while (files.hasNext()) {
      const file = files.next();
      const fileTime = file.getLastUpdated();

      // 同名ファイルを探す
      if (file.getName() === oldFileName) {
        sameNameFile = file;
      }

      // 最新のファイルを追跡
      if (!latestTime || fileTime > latestTime) {
        latestTime = fileTime;
        latestFile = file;
      }
    }

    // 同名ファイルがあればそれを優先、なければ最新のファイル
    const targetFile = sameNameFile || latestFile;

    if (!targetFile) {
      return { error: 'No PDF files found in folder' };
    }

    const isUpdated = targetFile.getId() !== oldFileId;

    return {
      success: true,
      fileId: targetFile.getId(),
      fileName: targetFile.getName(),
      folderId: folderId,
      modifiedTime: targetFile.getLastUpdated().toISOString(),
      isLatest: true,
      wasUpdated: isUpdated,
      oldFileId: isUpdated ? oldFileId : undefined
    };
  } catch (err) {
    return { error: 'Failed to get latest file: ' + err.message };
  }
}
```

### 2. GASに `updateDocUrl` アクションを追加（GETリクエスト）

新しいfileIdが見つかった時、スプレッドシートのProjectDataを自動更新する。

**重要**: POSTではなくGETを使う（CORSプリフライト回避のため）

```javascript
// doGet に追加
if (action === 'updateDocUrl') {
  const contractorId = e.parameter.contractorId;
  const docKey = e.parameter.docKey;
  const newFileId = e.parameter.newFileId;
  if (!contractorId || !docKey || !newFileId) {
    return jsonResponse({ error: 'contractorId, docKey, newFileId are required' });
  }
  const result = updateDocUrl(contractorId, docKey, newFileId);
  return jsonResponse(result);
}

// 関数を追加
function updateDocUrl(contractorId, docKey, newFileId) {
  try {
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    let dataSheet = ss.getSheetByName(CONFIG.DATA_SHEET);
    if (!dataSheet) {
      return { error: 'ProjectData sheet not found' };
    }

    const data = dataSheet.getRange('A1').getValue();
    if (!data) {
      return { error: 'No project data found' };
    }

    const project = JSON.parse(data);

    let updated = false;
    for (const contractor of (project.contractors || [])) {
      if (contractor.id === contractorId && contractor.docs && contractor.docs[docKey]) {
        const newUrl = `https://drive.google.com/file/d/${newFileId}/view?usp=drivesdk`;
        contractor.docs[docKey].url = newUrl;
        updated = true;
        break;
      }
    }

    if (!updated) {
      return { error: `Document not found: ${contractorId}/${docKey}` };
    }

    dataSheet.getRange('A1').setValue(JSON.stringify(project, null, 2));

    return {
      success: true,
      message: `URL updated for ${contractorId}/${docKey}`,
      newFileId: newFileId
    };
  } catch (err) {
    return { error: 'Failed to update doc URL: ' + err.message };
  }
}
```

### 3. React側での呼び出し（PdfViewer.tsx, AiChecker.tsx）

```typescript
// GASから最新ファイル情報を取得
let actualFileId = fileId;
try {
  const infoRes = await fetch(`${gasUrl}?action=getLatestFile&fileId=${fileId}`, { cache: 'no-store' });
  const info = await infoRes.json();
  console.log('[PdfViewer] GAS getLatestFile response:', info);
  if (!info.error) {
    modifiedTime = info.modifiedTime;
    setFileModifiedTime(modifiedTime || null);
    // ファイルIDが更新された場合は新しいIDを使用
    if (info.wasUpdated && info.fileId) {
      console.log('[PdfViewer] File updated:', fileId, '->', info.fileId);
      actualFileId = info.fileId;
      // スプレッドシートのURLを更新（GETリクエスト）
      if (contractorId && docKey) {
        try {
          const updateUrl = `${gasUrl}?action=updateDocUrl&contractorId=${encodeURIComponent(contractorId)}&docKey=${encodeURIComponent(docKey)}&newFileId=${encodeURIComponent(info.fileId)}`;
          await fetch(updateUrl, { cache: 'no-store' });
          console.log('[PdfViewer] Spreadsheet URL updated');
        } catch (e) {
          console.error('[PdfViewer] Failed to update spreadsheet URL:', e);
        }
      }
    }
  }
} catch (e) {
  console.error('[PdfViewer] getLatestFile failed:', e);
}

// 以降、actualFileId を使ってキャッシュチェックやPDF取得を行う
```

### 4. Rust側でiframe URLにパラメータ追加（pdf_viewer.rs）

```rust
let iframe_url = format!(
    "editor/index.html?mode=view&fileId={}&docType={}&contractor={}&gasUrl={}&contractorId={}&docKey={}",
    js_sys::encode_uri_component(&file_id),
    js_sys::encode_uri_component(&doc_type),
    js_sys::encode_uri_component(&contractor),
    js_sys::encode_uri_component(&gas_url),
    js_sys::encode_uri_component(&contractor_id),
    js_sys::encode_uri_component(&doc_key)
);
```

## CORSエラー回避

### POSTリクエストの場合

GASへのPOSTで `Content-Type: application/json` を使うと、CORSプリフライトが発生してブロックされる。

**解決策**: `Content-Type: text/plain` に変更

```rust
// gas.rs の save_to_gas 関数
request.headers()
    .set("Content-Type", "text/plain")
    .map_err(|e| format!("ヘッダー設定失敗: {:?}", e))?;
```

GAS側は `e.postData.contents` をJSONとしてパースするので、Content-Typeが何でも動作する。

### 書き込み操作の場合

書き込み操作（updateDocUrl等）は可能ならGETリクエストに変更する。
クエリパラメータでデータを渡せば、CORSプリフライトは発生しない。

## 注意事項

1. GASを再デプロイする必要がある（新しいアクションを追加した場合）
2. `wasUpdated: true` の時のみスプレッドシート更新を行う（無駄なAPI呼び出しを避ける）
3. キャッシュは `actualFileId` で管理する（古いfileIdではなく）
