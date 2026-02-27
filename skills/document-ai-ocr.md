---
name: document-ai-ocr
description: Google Document AI OCRで高精度な文字位置検出。(1) PDFから正確なバウンディングボックス取得、(2) normalizedVerticesで正規化座標、(3) 空欄位置の自動検出、(4) フォームフィールド解析。Document AI、OCR、バウンディングボックス、座標、空欄検出、PDF解析と言われた時に使用。
keywords: [Document AI, OCR, バウンディングボックス, 座標, 空欄, PDF, 正規化座標, Google Cloud, フォーム]
---

# Document AI OCR - 高精度PDF文字位置検出

## 概要

Google Cloud Document AI を使用してPDFから高精度な文字位置（バウンディングボックス）を取得する。Gemini等の汎用ビジョンAPIより正確な座標が得られる。

## セットアップ

### 1. Google Cloud プロジェクト設定

```bash
# Document AI API有効化
gcloud services enable documentai.googleapis.com

# プロセッサー作成（東京リージョン）
gcloud documentai processors create \
  --location=us \
  --display-name="OCR Processor" \
  --type=OCR_PROCESSOR
```

### 2. 認証設定

```bash
# サービスアカウント作成
gcloud iam service-accounts create document-ai-ocr

# 権限付与
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:document-ai-ocr@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/documentai.apiUser"

# キー生成
gcloud iam service-accounts keys create ~/document-ai-key.json \
  --iam-account=document-ai-ocr@PROJECT_ID.iam.gserviceaccount.com

# 環境変数設定
export GOOGLE_APPLICATION_CREDENTIALS=~/document-ai-key.json
```

## 使用方法

### TypeScript/Node.js

```typescript
import { DocumentProcessorServiceClient } from '@google-cloud/documentai';
import * as fs from 'fs';

interface BoundingBox {
  x: number;      // 左端 (0-1)
  y: number;      // 上端 (0-1)
  width: number;  // 幅 (0-1)
  height: number; // 高さ (0-1)
}

interface DetectedText {
  text: string;
  confidence: number;
  boundingBox: BoundingBox;
  pageIndex: number;
}

async function extractTextWithBoundingBoxes(
  pdfPath: string,
  projectId: string,
  location: string,  // 'us' or 'eu'
  processorId: string
): Promise<DetectedText[]> {
  const client = new DocumentProcessorServiceClient();

  const pdfData = fs.readFileSync(pdfPath);
  const encodedPdf = pdfData.toString('base64');

  const request = {
    name: `projects/${projectId}/locations/${location}/processors/${processorId}`,
    rawDocument: {
      content: encodedPdf,
      mimeType: 'application/pdf',
    },
  };

  const [result] = await client.processDocument(request);
  const { document } = result;

  if (!document || !document.pages) {
    throw new Error('No document or pages in response');
  }

  const detectedTexts: DetectedText[] = [];

  for (let pageIndex = 0; pageIndex < document.pages.length; pageIndex++) {
    const page = document.pages[pageIndex];

    // 各トークン（単語単位）を処理
    for (const token of page.tokens || []) {
      const layout = token.layout;
      if (!layout?.textAnchor?.textSegments?.[0] || !layout.boundingPoly?.normalizedVertices) {
        continue;
      }

      // テキスト取得
      const segment = layout.textAnchor.textSegments[0];
      const startIndex = parseInt(segment.startIndex || '0');
      const endIndex = parseInt(segment.endIndex || '0');
      const text = document.text?.substring(startIndex, endIndex) || '';

      // バウンディングボックス計算（normalizedVerticesは4点）
      const vertices = layout.boundingPoly.normalizedVertices;
      const xs = vertices.map(v => v.x || 0);
      const ys = vertices.map(v => v.y || 0);

      const boundingBox: BoundingBox = {
        x: Math.min(...xs),
        y: Math.min(...ys),
        width: Math.max(...xs) - Math.min(...xs),
        height: Math.max(...ys) - Math.min(...ys),
      };

      detectedTexts.push({
        text: text.trim(),
        confidence: layout.confidence || 0,
        boundingBox,
        pageIndex,
      });
    }
  }

  return detectedTexts;
}

// 使用例
async function main() {
  const texts = await extractTextWithBoundingBoxes(
    'input.pdf',
    'your-project-id',
    'us',
    'your-processor-id'
  );

  console.log(JSON.stringify(texts, null, 2));
}
```

### 空欄検出ロジック

```typescript
interface BlankField {
  type: 'destination' | 'year' | 'month' | 'day';
  marker: string;
  x: number;
  y: number;
  width: number;
  height: number;
}

function detectBlankFields(texts: DetectedText[]): BlankField[] {
  const fields: BlankField[] = [];

  for (const text of texts) {
    const { boundingBox } = text;

    // 宛名欄: 「殿」「御中」「様」の左側
    if (/^(殿|御中|様)$/.test(text.text)) {
      fields.push({
        type: 'destination',
        marker: text.text,
        x: Math.max(0, boundingBox.x - 0.15),  // マーカーの左側
        y: boundingBox.y,
        width: 0.15,
        height: boundingBox.height,
      });
    }

    // 年欄: 「令和」の右側
    if (text.text.includes('令和')) {
      fields.push({
        type: 'year',
        marker: '令和',
        x: boundingBox.x + boundingBox.width,  // マーカーの右側
        y: boundingBox.y,
        width: 0.03,
        height: boundingBox.height,
      });
    }

    // 月欄: 「年」の右側（日付文脈）
    if (/^年$/.test(text.text)) {
      fields.push({
        type: 'month',
        marker: '年',
        x: boundingBox.x + boundingBox.width,
        y: boundingBox.y,
        width: 0.025,
        height: boundingBox.height,
      });
    }

    // 日欄: 「月」の右側（日付文脈）
    if (/^月$/.test(text.text)) {
      fields.push({
        type: 'day',
        marker: '月',
        x: boundingBox.x + boundingBox.width,
        y: boundingBox.y,
        width: 0.025,
        height: boundingBox.height,
      });
    }
  }

  return fields;
}
```

### Dart/Flutter版

```dart
import 'dart:convert';
import 'dart:io';
import 'package:http/http.dart' as http;
import 'package:googleapis_auth/auth_io.dart';

class DocumentAIService {
  final String projectId;
  final String location;
  final String processorId;
  final String credentialsPath;

  DocumentAIService({
    required this.projectId,
    required this.location,
    required this.processorId,
    required this.credentialsPath,
  });

  Future<List<DetectedText>> extractText(String pdfPath) async {
    // 認証
    final credentials = ServiceAccountCredentials.fromJson(
      jsonDecode(File(credentialsPath).readAsStringSync()),
    );
    final client = await clientViaServiceAccount(
      credentials,
      ['https://www.googleapis.com/auth/cloud-platform'],
    );

    // PDF読み込み
    final pdfBytes = File(pdfPath).readAsBytesSync();
    final encodedPdf = base64Encode(pdfBytes);

    // APIリクエスト
    final url = 'https://$location-documentai.googleapis.com/v1/'
        'projects/$projectId/locations/$location/processors/$processorId:process';

    final response = await client.post(
      Uri.parse(url),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({
        'rawDocument': {
          'content': encodedPdf,
          'mimeType': 'application/pdf',
        },
      }),
    );

    if (response.statusCode != 200) {
      throw Exception('API error: ${response.statusCode} - ${response.body}');
    }

    final result = jsonDecode(response.body);
    return _parseDocument(result['document']);
  }

  List<DetectedText> _parseDocument(Map<String, dynamic> document) {
    final texts = <DetectedText>[];
    final fullText = document['text'] as String? ?? '';

    for (final page in document['pages'] ?? []) {
      final pageIndex = page['pageNumber'] as int? ?? 1;

      for (final token in page['tokens'] ?? []) {
        final layout = token['layout'];
        if (layout == null) continue;

        final textAnchor = layout['textAnchor'];
        final segments = textAnchor?['textSegments'] as List? ?? [];
        if (segments.isEmpty) continue;

        final startIndex = int.tryParse(segments[0]['startIndex']?.toString() ?? '0') ?? 0;
        final endIndex = int.tryParse(segments[0]['endIndex']?.toString() ?? '0') ?? 0;
        final text = fullText.substring(startIndex, endIndex).trim();

        final vertices = layout['boundingPoly']?['normalizedVertices'] as List? ?? [];
        if (vertices.length < 4) continue;

        final xs = vertices.map((v) => (v['x'] as num?)?.toDouble() ?? 0.0).toList();
        final ys = vertices.map((v) => (v['y'] as num?)?.toDouble() ?? 0.0).toList();

        texts.add(DetectedText(
          text: text,
          confidence: (layout['confidence'] as num?)?.toDouble() ?? 0.0,
          boundingBox: BoundingBox(
            x: xs.reduce((a, b) => a < b ? a : b),
            y: ys.reduce((a, b) => a < b ? a : b),
            width: xs.reduce((a, b) => a > b ? a : b) - xs.reduce((a, b) => a < b ? a : b),
            height: ys.reduce((a, b) => a > b ? a : b) - ys.reduce((a, b) => a < b ? a : b),
          ),
          pageIndex: pageIndex - 1,
        ));
      }
    }

    return texts;
  }
}

class BoundingBox {
  final double x, y, width, height;
  BoundingBox({required this.x, required this.y, required this.width, required this.height});
}

class DetectedText {
  final String text;
  final double confidence;
  final BoundingBox boundingBox;
  final int pageIndex;
  DetectedText({required this.text, required this.confidence, required this.boundingBox, required this.pageIndex});
}
```

## 料金

| 項目 | 料金 |
|------|------|
| OCR処理 | $1.50 / 1000ページ |
| Form Parser | $30 / 1000ページ |
| 最初の1000ページ/月 | 無料 |

## 依存パッケージ

### Node.js
```bash
npm install @google-cloud/documentai
```

### Dart/Flutter
```yaml
dependencies:
  googleapis_auth: ^1.4.0
  http: ^1.2.0
```

## テスト用CLIスクリプト

```typescript
// test-document-ai.ts
import { DocumentProcessorServiceClient } from '@google-cloud/documentai';
import * as fs from 'fs';

const PROJECT_ID = process.env.DOCUMENT_AI_PROJECT_ID!;
const LOCATION = process.env.DOCUMENT_AI_LOCATION || 'us';
const PROCESSOR_ID = process.env.DOCUMENT_AI_PROCESSOR_ID!;

async function main() {
  const pdfPath = process.argv[2];
  if (!pdfPath) {
    console.error('Usage: npx tsx test-document-ai.ts <pdf_path>');
    process.exit(1);
  }

  console.log('=== Document AI OCR Test ===');
  console.log(`PDF: ${pdfPath}`);
  console.log(`Project: ${PROJECT_ID}`);
  console.log(`Processor: ${PROCESSOR_ID}`);

  const client = new DocumentProcessorServiceClient();
  const pdfData = fs.readFileSync(pdfPath).toString('base64');

  const [result] = await client.processDocument({
    name: `projects/${PROJECT_ID}/locations/${LOCATION}/processors/${PROCESSOR_ID}`,
    rawDocument: { content: pdfData, mimeType: 'application/pdf' },
  });

  const doc = result.document!;
  console.log(`\nPages: ${doc.pages?.length}`);
  console.log(`Full text length: ${doc.text?.length}`);

  // マーカー検出
  const markers = ['殿', '御中', '様', '令和', '年', '月', '日'];
  console.log('\n=== Detected Markers ===');

  for (const page of doc.pages || []) {
    for (const token of page.tokens || []) {
      const layout = token.layout;
      const segment = layout?.textAnchor?.textSegments?.[0];
      if (!segment) continue;

      const start = parseInt(segment.startIndex || '0');
      const end = parseInt(segment.endIndex || '0');
      const text = doc.text?.substring(start, end).trim() || '';

      if (markers.some(m => text.includes(m))) {
        const v = layout!.boundingPoly!.normalizedVertices!;
        const x = Math.min(...v.map(p => p.x || 0));
        const y = Math.min(...v.map(p => p.y || 0));
        console.log(`  "${text}" at (${x.toFixed(4)}, ${y.toFixed(4)})`);
      }
    }
  }

  console.log('\n✓ Test completed');
}

main().catch(console.error);
```

## 環境変数

```bash
export DOCUMENT_AI_PROJECT_ID="your-project-id"
export DOCUMENT_AI_LOCATION="us"
export DOCUMENT_AI_PROCESSOR_ID="your-processor-id"
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"
```
