---
name: rust-wasm-pages
description: Rust + WebAssembly + Pagesによる横断図・切削計算WebUIの構築。(1) Rustでのwasm-bindgen/Trunkビルド、(2) DXF生成ライブラリ（dxf-rs等）、(3) GitHub/Cloudflare Pagesへのデプロイ、(4) 横断図データの補間ロジック。Rust、WASM、WebAssembly、Pages、横断図UI、切削計算UIと言われた時に使用。
---

# Rust + WASM + Pages WebUI構築

## 概要

舗装工事の切削計算書・横断図生成をブラウザ上で行うWebアプリケーションをRust + WebAssemblyで構築する。

## 技術スタック

### フロントエンド

| 選択肢 | 特徴 |
|--------|------|
| **Yew** | React風、成熟、ドキュメント豊富 |
| **Leptos** | React風、SSR対応、高速 |
| **Dioxus** | React風、マルチプラットフォーム |

### ビルドツール

- **Trunk**: Rustフロントエンドのビルド・バンドル
- **wasm-pack**: ライブラリとしてWASMを生成

### DXF生成

| クレート | 用途 |
|----------|------|
| `dxf-rs` | DXFファイル読み書き |
| `lyon` | ベクターグラフィックス（SVGプレビュー用） |

## 実装済みプロジェクト

`csv_to_dxf/cross-section-ui/` に実装済み。

```
cross-section-ui/
├── .github/workflows/deploy.yml  # GitHub Pagesデプロイ
├── .gitignore
├── Cargo.toml
├── Cargo.lock
├── index.html
├── src/
│   └── main.rs    # Yewアプリ（Canvas描画、切削計算表）
└── static/
    └── styles.css
```

### 実装済み機能

- サンプルデータ読込
- 測点選択UI
- Canvas描画（3本線: 表層天端/現地盤/切削底面）
- 切削計算表（距離、現地盤高、切削厚）
- レジェンド表示

## 初期セットアップ

### 1. Rustツールチェーンの準備

```bash
rustup target add wasm32-unknown-unknown
cargo install trunk
```

### 2. プロジェクト作成（Yewの場合）

```bash
cargo new cross-section-ui
cd cross-section-ui
```

### 3. Cargo.toml

```toml
[package]
name = "cross-section-ui"
version = "0.1.0"
edition = "2021"

[dependencies]
yew = { version = "0.21", features = ["csr"] }
wasm-bindgen = "0.2"
web-sys = { version = "0.3", features = ["File", "FileReader", "Blob"] }
js-sys = "0.3"
gloo = "0.11"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
dxf = "0.5"  # DXF生成用
calamine = "0.24"  # Excel読み込み

[profile.release]
lto = true
opt-level = "z"
```

### 4. index.html

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1"/>
    <title>横断図・切削計算システム</title>
    <link data-trunk rel="css" href="static/styles.css"/>
</head>
<body>
</body>
</html>
```

### 5. ビルド・起動

```bash
trunk serve  # 開発サーバー
trunk build --release  # 本番ビルド
```

## 横断図の3本線描画

### データ構造

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct CrossSectionPoint {
    pub distance: f64,           // CLからの距離（左がマイナス）
    pub existing_elevation: f64, // 現地盤高
    pub surface_plan: f64,       // 表層天端計画高
    pub cutting_bottom: f64,     // 切削底面計画高
}

#[derive(Debug, Clone)]
pub struct CrossSection {
    pub station_name: String,
    pub points: Vec<CrossSectionPoint>,
}

impl CrossSectionPoint {
    /// 切削厚を計算
    pub fn cutting_depth(&self) -> f64 {
        self.existing_elevation - self.cutting_bottom
    }

    /// 舗装厚を計算
    pub fn pavement_thickness(&self) -> f64 {
        self.surface_plan - self.cutting_bottom
    }
}
```

### 補間ロジック

```rust
/// CLの計画高と横断勾配から、任意の距離の計画高を補間
pub fn interpolate_plan_height(
    cl_height: f64,
    distance: f64,
    slope: f64,  // 横断勾配（正: 下り、負: 上り）
) -> f64 {
    cl_height - (distance.abs() * slope)
}

/// 2点間の線形補間
pub fn linear_interpolate(
    x: f64,
    x1: f64, y1: f64,
    x2: f64, y2: f64,
) -> f64 {
    y1 + (y2 - y1) * (x - x1) / (x2 - x1)
}
```

## Pagesデプロイ

### GitHub Pages

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Rust
        uses: dtolnay/rust-action@stable
        with:
          targets: wasm32-unknown-unknown

      - name: Install Trunk
        run: cargo install trunk

      - name: Build
        run: trunk build --release --public-url /${{ github.event.repository.name }}/

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

### Cloudflare Pages

1. GitHubリポジトリを接続
2. ビルド設定:
   - Build command: `trunk build --release`
   - Build output directory: `dist`

## 入力データ形式

### Excel（着工前測量結果.xlsx）

```
| 測点 | 距離 | L現地盤 | CL現地盤 | R現地盤 |
|------|------|---------|----------|---------|
| No.0 | 0    | 10.45   | 10.42    | 10.48   |
```

### Excel（計画.xlsx）

```
| 測点 | CL計画高 | 横断勾配L | 横断勾配R | 舗装厚 |
|------|----------|-----------|-----------|--------|
| No.0 | 10.50    | 0.02      | 0.02      | 0.10   |
```

## 重要ルール

### デプロイ前の動作確認（必須）

**コード変更後は必ずブラウザで実際の画面を確認すること。**

1. `trunk serve` でローカル確認、または
2. GitHub Pagesにデプロイ後、実際のURLで確認
3. 期待通りの動作をしているか目視で検証
4. 確認せずに「完了」と報告してはならない

### trianglelistからのコード再利用

DXF描画関連の実装は `<USER_HOME>\yuuji\StudioProjects\trianglelist` に既存実装がある。
新規実装する前に必ず参照し、同じロジックを移植すること。

| 機能 | 参照ファイル |
|------|--------------|
| テキスト回転 | `desktop/src/main/kotlin/.../TextRenderer.kt` |
| テキストアライメント | `desktop/src/main/kotlin/.../TextRenderer.kt` calculateAlignedPosition |
| DXFパース | `common/src/commonMain/kotlin/.../dxf/` |

## 関連スキル

- `cutting-calculation`: 切削計算書の構造
- `cross-section`: 横断図DXF生成（Python版）
- `google-api`: スプレッドシートとの連携
- `photo-ai-rust`: 工事写真AI解析CLIツール
