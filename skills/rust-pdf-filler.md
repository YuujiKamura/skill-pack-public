---
name: rust-pdf-filler
description: RustでPDFにテキスト書き込み。PDF、Rust、lopdf、書き込み、テキスト追加、アノテーションと言われた時に使用。
---

# Rust PDF書き込みスキル

## 概要

RustのlopdfライブラリでPDFにテキストを追加する。FreeTextアノテーション方式を使用し、元のPDFコンテンツを破損しない安全な実装。

## プロジェクト場所

```
<USER_HOME>\yuuji\Sanyuu2Kouku\SekouTaiseiMaker\rust_pdf_filler\
```

## 依存関係（Cargo.toml）

```toml
[package]
name = "pdf_filler"
version = "0.1.0"
edition = "2021"

[dependencies]
lopdf = "0.36"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
anyhow = "1.0"
```

## 核心技術

### 1. PDF読み込みとページ情報取得

```rust
use lopdf::{Document, Object, Dictionary, Stream, dictionary};

let mut doc = Document::load(input_pdf)?;
let pages = doc.get_pages();
let page_id = *pages.get(&1)?;

// ページサイズ取得（MediaBox: [x0, y0, x1, y1]）
if let Ok(Object::Dictionary(dict)) = doc.get_object(page_id) {
    if let Ok(Object::Array(media_box)) = dict.get(b"MediaBox") {
        let width = media_box[2];  // x1
        let height = media_box[3]; // y1
    }
}
```

### 2. FreeTextアノテーション方式（推奨）

コンテンツストリームを直接変更すると既存のPDFが壊れる場合がある。代わりにアノテーションを使用：

```rust
// 標準フォント（Times-Roman：セリフ体で明朝に近い）
let font_dict = dictionary! {
    "Type" => Object::Name(b"Font".to_vec()),
    "Subtype" => Object::Name(b"Type1".to_vec()),
    "BaseFont" => Object::Name(b"Times-Roman".to_vec())
};
let font_id = doc.add_object(font_dict);

// Appearance Stream（実際の描画内容）
let ap_content = format!(
    "q BT /TimesRoman {:.1} Tf 0 g 0 0 Td ({}) Tj ET Q",
    font_size, text_value
);
let ap_stream = Stream::new(
    dictionary! {
        "Type" => Object::Name(b"XObject".to_vec()),
        "Subtype" => Object::Name(b"Form".to_vec()),
        "BBox" => Object::Array(vec![
            Object::Real(0.0),
            Object::Real(0.0),
            Object::Real(text_width),
            Object::Real(font_size),
        ]),
        "Resources" => dictionary! {
            "Font" => dictionary! {
                "TimesRoman" => Object::Reference(font_id)
            }
        }
    },
    ap_content.into_bytes(),
);
let ap_id = doc.add_object(ap_stream);

// FreeTextアノテーション
let annot_dict = dictionary! {
    "Type" => Object::Name(b"Annot".to_vec()),
    "Subtype" => Object::Name(b"FreeText".to_vec()),
    "Rect" => Object::Array(vec![
        Object::Real(x1),
        Object::Real(y1),
        Object::Real(x2),
        Object::Real(y2),
    ]),
    "Contents" => Object::String(text_value.as_bytes().to_vec(), lopdf::StringFormat::Literal),
    "DA" => Object::String(
        format!("/TimesRoman {} Tf 0 g", font_size).into_bytes(),
        lopdf::StringFormat::Literal
    ),
    "AP" => dictionary! {
        "N" => Object::Reference(ap_id)
    },
    "F" => Object::Integer(4),  // Print flag
    "BS" => dictionary! {
        "W" => Object::Integer(0)  // ボーダー幅0（非表示）
    }
};
```

### 3. 座標変換（Document AI OCR → PDF）

Document AI OCRはパーセント座標（上が0%）を返す。PDF座標系は左下が原点。

```rust
/// パーセント座標をPDFポイント座標に変換
pub fn percent_to_pdf_coords(
    marker_x_percent: f64,
    marker_y_percent: f64,
    marker_height_percent: f64,
    page_width: f64,
    page_height: f64,
    font_size: f64,
) -> (f64, f64) {
    // X座標: パーセント → ポイント
    let x_pt = (marker_x_percent / 100.0) * page_width;

    // マーカーの中央Y座標（上からの距離）
    let marker_top = (marker_y_percent / 100.0) * page_height;
    let marker_height = (marker_height_percent / 100.0) * page_height;
    let marker_center_from_top = marker_top + marker_height / 2.0;

    // PDF座標系に変換（下からの距離）
    let marker_center_from_bottom = page_height - marker_center_from_top;

    // テキストの視覚的中央をマーカー中央に合わせる
    let baseline_y = marker_center_from_bottom - (font_size * 0.3);

    (x_pt, baseline_y)
}
```

### 4. フォントサイズ計算

```rust
// マーカーの高さの1.2倍
let marker_height_pt = (marker_height_percent / 100.0) * page_height;
let font_size = marker_height_pt * 1.2;
```

## 使用可能な標準Type1フォント

- `Helvetica` - サンセリフ（ゴシック系）
- `Helvetica-Bold`
- `Times-Roman` - セリフ（明朝系）★推奨
- `Times-Bold`
- `Courier` - 等幅
- `Courier-Bold`

## 注意事項

### MS明朝フォント埋め込みの課題

CIDフォント（日本語フォント）の埋め込みは複雑：
- TTCファイル（TrueType Collection）の処理
- CIDToGIDMapの設定
- エンコーディング（Identity-H、90ms-RKSJ-H）

ASCII数字のみの場合は標準Type1フォント（Times-Roman）で十分。

### コンテンツストリーム直接変更は危険

```rust
// NG: コンテンツストリームを直接追加すると既存PDFが壊れる可能性
dict.set(b"Contents", Object::Array(vec![
    Object::Reference(existing_ref),
    Object::Reference(new_stream_id),  // 危険！
]));
```

代わりにアノテーション方式を使用すること。

## アプリ統合の課題（保留）

Document AI OCRとの連携にはGoogle Cloud認証が必要：
- サービスアカウント認証ファイル（JSON）の管理
- 環境変数またはファイルパスでの指定
- Webアプリでの認証フロー

現状はスタンドアロンのRustツールとして使用。

## 関連ファイル

- `rust_pdf_filler/src/main.rs` - メイン実装
- `scripts/test_documentai_ocr.py` - Document AI OCRテスト（Python）
- `scripts/documentai_config.json` - OCR設定

## 対応書類と座標（アイエスティー）

### 書類パターン一覧

| 書類 | パターン | 令和印刷 | ページ数 | y座標 |
|------|----------|----------|----------|-------|
| 暴対法 | 令+和+年月+日 | あり | 1 | 19.5% |
| 建退共 | 令+和+年月+日 | あり | 1 | 19.5% |
| 作業員名簿 | 提出日+年+月+日 | なし | 2 | 12.14% |

### パターン1: 暴対法・建退共（令和が印刷済み）

Document AI OCR座標:
```
令: x=62.3%, y=19.5%, w=1.9%, h=1.3%
和: x=64.5%, y=19.5%, w=1.78%, h=1.3%
年月: x=70.4%, y=19.5%, w=7.7%, h=1.3%
日: x=82.9%, y=19.5%, w=1.2%, h=1.3%
```

空欄位置の計算:
```rust
// 年: 和と年月の間
year.x = (和.x + 和.width + 年月.x) / 2.0
// 月: 年月の中央右寄り
month.x = 年月.x + 年月.width / 2.0 + 0.5
// 日: 年月と日の間
day.x = (年月.x + 年月.width + 日.x) / 2.0
```

### パターン2: 作業員名簿（令和が印刷されていない）

Document AI OCR座標:
```
提出日: x=75.36%, y=12.14%, w=0.42%, h=0.89%
年: x=79.81%, y=12.14%, w=0.76%, h=0.89%
月: x=82.80%, y=12.14%, w=0.67%, h=0.95%
日: x=86.04%, y=12.20%, w=0.55%, h=0.83%
```

**重要**: 令和が印刷されていないので「R」を追加書き込みする。

空欄位置の計算:
```rust
// R と 年数字 を詰めて配置（年マーカーを基準）
year.x = 年.x - 1.0   // 年の直前
reiwa.x = 年.x - 1.8  // 年数字の直前（R8を詰めて表示）
// 月: 年と月の間の中央
month.x = (年.x + 年.width + 月.x) / 2.0 - 0.3
// 日: 月と日の間の中央
day.x = (月.x + 月.width + 日.x) / 2.0 - 0.3
```

### 全ページ対応

作業員名簿は2ページあるため、全ページに書き込む:

```rust
let pages = doc.get_pages();
for (page_num, page_id) in &pages {
    // 各ページに同じアノテーションを追加
    let (page_width, page_height) = get_page_size(&doc, *page_id)?;
    // ... アノテーション追加処理
}
```

## Document AI OCR座標取得の正しいフロー

### 重要: Gemini APIではなくDocument AI OCRを使う

Gemini APIの座標は不正確。必ずDocument AI OCRを使用すること。

```bash
cd scripts
python test_documentai_ocr.py "対象PDF.pdf"
```

### 座標取得時の確認項目

1. **y座標で対象箇所を特定**: 提出日欄は通常 y < 20% の上部にある
2. **x座標でマーカーを特定**: 「令」「和」「年」「月」「日」などの文字
3. **複数ページの場合は両方確認**: レイアウトが同じか確認

### Document AI OCR実行例

```python
# 右上のトークン（提出日欄）を抽出
for token in page.tokens:
    v = token.layout.bounding_poly.normalized_vertices
    x = v[0].x * 100
    y = v[0].y * 100
    if x > 70 and y < 15:  # 右上のみ
        print(f"'{text}': x={x:.2f}%, y={y:.2f}%")
```

## 教訓: やってはいけないこと

### 1. Gemini APIで座標を取得してはいけない

Gemini APIは視覚的な認識はできるが、正確なバウンディングボックス座標は返さない。Document AI OCRのnormalized_verticesを使うこと。

### 2. 座標を目視確認せずに実装してはいけない

OCR結果を数値で確認し、計算した空欄位置が妥当か検証する:
```rust
println!("Document AI OCR座標:");
for (name, m) in &markers {
    println!("  {}: x={:.2}%, y={:.2}%", name, m.x, m.y);
}
println!("計算した空欄位置:");
for (name, b) in &blanks {
    println!("  {}: x={:.2}%, y={:.2}%", name, b.x, b.y);
}
```

### 3. 間違った書類のOCR結果を使ってはいけない

`test_documentai_ocr.py`にはTEST_PDFがハードコードされている。別の書類をOCRする場合は直接Pythonコードを書くか、スクリプトを修正すること。

### 4. 後付けの調整をしてはいけない

座標が間違っていたら、元のDocument AI OCR座標を再確認して根本的に修正する。「+10px」のような後付け調整は問題を複雑化させる。

## 位置調整の考え方

### マーカーを基準に計算

空欄位置は必ずOCRで検出したマーカーを基準に計算する:

```rust
// 良い例: マーカーを基準に相対位置を計算
year.x = 年マーカー.x - 1.0

// 悪い例: 固定座標を直接指定
year.x = 78.5  // マーカーとの関係が不明
```

### R と 数字 の間隔

作業員名簿の「R8」のように近接配置する場合、間隔を詰める:

```rust
// R と 8 を詰めて配置
year.x = nen.x - 1.0   // 8 の位置
reiwa.x = nen.x - 1.8  // R の位置（8の0.8%左）
```

## 変更履歴

- 2026-01-05: 初版作成。lopdf + FreeTextアノテーション方式で実装。Times-Romanフォント使用。
- 2026-01-05: 建退共・作業員名簿対応追加。Document AI OCR座標取得フロー、全ページ対応、教訓を追記。
