---
name: grid-layout-params
description: グリッドレイアウトのセル高さ計算パターン。(1) v_scale_ratioとrow_gapの分離計算、(2) セクション高さの一元管理、(3) 重複コードの共通化リファクタリング、(4) 固定マージンとスケール依存部分の分離、(5) DXF単位変換の統一。GridLayoutParams、セル高さ、行間隔、v_scale、縦倍率、グリッド、レイアウト、間隔、row_gap、cell_height、重なる、重なりと言われた時に使用。
---

# GridLayoutParams パターン

## 変更履歴
- 2026-01-29: TOP_MARGIN_DXF/BOTTOM_MARGIN_DXFを追加、ラベル領域を正しく計算

## 問題

グリッドレイアウトで複数のセクションを配置する際、以下の問題が発生：

1. **row_gapがv_scale_ratioでスケールされる問題**
   - 旧: `(max_height + row_gap) * scale * v_scale_ratio`
   - v_scale_ratio=1.0のとき、row_gapも縮小されて間隔が詰まる

2. **ラベル領域が計算に含まれていない問題**
   - 旗揚げテキスト（No.xxx, GH=, FH=）は上方向に伸びる
   - 切削厚ラベルはDLより下に伸びる
   - これらがセル高さに含まれず、セクション同士が重なる

3. **重複コード**
   - 同じ計算ロジックが4箇所以上に存在

## セル高さの正しい構造

```
セル高さ = セクション本体(スケール適用) + 上マージン(固定) + 下マージン(固定) + 行間隔(固定)

具体的には:
cell_height = body_height * scale * v_scale_ratio  // 本体（地盤〜最高点）
            + TOP_MARGIN_DXF                       // 旗揚げラベル領域
            + BOTTOM_MARGIN_DXF                    // 切削厚ラベル領域
            + row_gap * scale                      // 行間隔
```

## 固定マージン定数（DXF単位）

SectionStyleのデフォルト値から計算：

```rust
// 上方向: flag_base_offset + label_title_offset + text_height * title_scale
const TOP_MARGIN_DXF: f64 = 1600.0 + 1300.0 + 300.0 * 1.5;  // = 3350.0

// 下方向: cutting_label_offset + text_height + cutting_label_gap
const BOTTOM_MARGIN_DXF: f64 = 300.0 + 300.0 + 100.0;  // = 700.0
```

## GridLayoutParams構造体

```rust
#[derive(Debug, Clone, Copy)]
pub struct GridLayoutParams {
    pub scale: f64,           // 基本スケール（通常1000.0 = 1m）
    pub v_scale_ratio: f64,   // 縦方向スケール倍率（1.0〜5.0）
    pub column_gap: f64,      // 列間隔（メートル）
    pub row_gap: f64,         // 行間隔（メートル）
}

impl GridLayoutParams {
    pub fn new(column_gap: f64, row_gap: f64) -> Self { ... }
    pub fn with_v_scale(mut self, v_scale_ratio: f64) -> Self { ... }

    /// セクションの本体高さ（メートル単位、max_elev - dl）
    pub fn section_body_height(&self, section: &CrossSectionData) -> f64 { ... }

    /// 複数セクションの最大本体高さ
    pub fn max_section_body_height(&self, sections: &[CrossSectionData]) -> f64 { ... }

    /// セクションの全描画高さ（DXF単位）
    /// = body * scale * v_scale + TOP_MARGIN + BOTTOM_MARGIN
    pub fn section_height_dxf(&self, body_height_m: f64) -> f64 {
        body_height_m * self.scale * self.v_scale_ratio + TOP_MARGIN_DXF + BOTTOM_MARGIN_DXF
    }

    /// セル高さ（DXF単位）= section_height + row_gap
    pub fn cell_height_dxf(&self, max_body_height_m: f64) -> f64 {
        self.section_height_dxf(max_body_height_m) + self.row_gap * self.scale
    }

    /// 複数セクションからセル高さを直接計算
    pub fn cell_height_for_sections(&self, sections: &[CrossSectionData]) -> f64 { ... }

    /// Y方向スケール（DXF単位変換用）
    pub fn y_scale(&self) -> f64 { ... }

    /// 下マージン（配置オフセット用）
    pub fn bottom_margin_dxf(&self) -> f64 { BOTTOM_MARGIN_DXF }
}
```

## offset_yの計算

描画は`offset_y`を基準に行われ、切削厚ラベルは`offset_y`より下に配置される。
セル内に収めるには`BOTTOM_MARGIN_DXF`分上にシフトする必要がある：

```rust
// NG: 切削厚ラベルがセル下端からはみ出す
let offset_y = row_in_col as f64 * cell_height;

// OK: 下マージン分上にシフト
let offset_y = row_in_col as f64 * cell_height + BOTTOM_MARGIN_DXF;
```

## 使用例

```rust
let params = GridLayoutParams::new(column_gap, row_gap)
    .with_v_scale(v_scale_ratio);

// セル高さ
let cell_height = params.cell_height_for_sections(sections);

// セクション配置位置
let offset_y = row_in_col as f64 * cell_height + BOTTOM_MARGIN_DXF;
```

## 適用箇所（cross-section-ui）

1. `calc_grid_bounds_with_v_scale` - バウンディングボックス計算
2. `generate_multi_drawing_internal` - DXF生成
3. `draw_sections_in_frame` - 図枠内描画
4. `calc_section_bounds_in_grid_with_v_scale` - 個別セクション位置計算

## 一般化のポイント

1. **スケール依存部分と固定部分を明確に分離**
   - コンテンツ本体 → v_scale_ratio適用
   - ラベルマージン → 固定DXF単位（スケール非依存）
   - 行間隔 → 固定（scale適用、v_scale非依存）

2. **マージンはスタイル定数から算出**
   - SectionStyleの値を元に計算
   - スタイル変更時は定数も更新が必要

3. **配置時のオフセット考慮**
   - 下方向にはみ出す要素がある場合、offset_yをシフト

## 関連Issue

- GitHub Issue #60: 縦倍率変更時の行間隔スケーリング問題
