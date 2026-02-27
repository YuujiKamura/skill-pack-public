---
name: excel-export
description: Use when this skill's workflow applies
---

# Excel Export パターン

ExcelJSを使用したExcelファイル生成・ダウンロード。

## インストール

```bash
npm install exceljs
```

## 基本パターン

```typescript
import ExcelJS from 'exceljs';

// ワークブック作成
const workbook = new ExcelJS.Workbook();
workbook.creator = 'MyApp';
workbook.created = new Date();

// シート作成
const sheet = workbook.addWorksheet('Sheet1');

// 列幅設定
sheet.getColumn('A').width = 10;
sheet.getColumn('B').width = 20;

// ヘッダー行
sheet.getRow(1).values = ['No', '名前', '金額'];
sheet.getRow(1).font = { bold: true };

// データ行
data.forEach((item, idx) => {
  const row = sheet.getRow(idx + 2);
  row.values = [idx + 1, item.name, item.amount];
});

// ダウンロード
const buffer = await workbook.xlsx.writeBuffer();
const blob = new Blob([buffer], {
  type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'
});
const url = URL.createObjectURL(blob);
const a = document.createElement('a');
a.href = url;
a.download = 'export.xlsx';
a.click();
URL.revokeObjectURL(url);
```

## スタイル設定

```typescript
// フォント
cell.font = {
  name: 'ＭＳ ゴシック',
  size: 11,
  bold: true,
  color: { argb: 'FF000000' }
};

// 配置
cell.alignment = {
  horizontal: 'center',  // left, center, right
  vertical: 'middle',    // top, middle, bottom
};

// 罫線
const thinBorder = { style: 'thin', color: { argb: 'FF000000' } };
cell.border = {
  top: thinBorder,
  left: thinBorder,
  bottom: thinBorder,
  right: thinBorder
};

// 背景色
cell.fill = {
  type: 'pattern',
  pattern: 'solid',
  fgColor: { argb: 'FFCCCCCC' }
};

// 数値フォーマット
cell.numFmt = '#,##0.00';      // 数値
cell.numFmt = 'yyyy/mm/dd';    // 日付
```

## セル結合

```typescript
sheet.mergeCells('A1:D1');  // A1〜D1を結合
sheet.mergeCells('A1:A3');  // 縦に結合
sheet.mergeCells(1, 1, 1, 4);  // row1, col1 から row1, col4
```

## 行高さ設定

```typescript
sheet.getRow(1).height = 30;

// 複数行
for (let i = 1; i <= 20; i++) {
  sheet.getRow(i).height = 25;
}
```

## 日本語テキスト幅計算

```typescript
const getTextWidth = (text: string): number => {
  if (!text) return 0;
  let width = 0;
  for (const char of text) {
    // 全角は2、半角は1
    width += char.charCodeAt(0) > 255 ? 2 : 1;
  }
  return Math.ceil(width * 1.2);  // 余裕を持たせる
};

// 列幅の自動調整
const maxWidth = Math.max(...data.map(d => getTextWidth(d.name)));
sheet.getColumn('B').width = Math.max(10, maxWidth);
```

## 複数シート

```typescript
// 統括シート
const summarySheet = workbook.addWorksheet('統括表');

// 詳細シート（日付ごと）
dateGroups.forEach(([date, items]) => {
  const sheet = workbook.addWorksheet(date);
  // データ追加
});
```

## ダウンロード関数（汎用）

```typescript
export const downloadExcel = async (
  workbook: ExcelJS.Workbook,
  filename: string = 'export.xlsx'
): Promise<void> => {
  const buffer = await workbook.xlsx.writeBuffer();
  const blob = new Blob([buffer], {
    type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'
  });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
};
```

## ポイント

- `writeBuffer()` でブラウザ内生成
- Blob + createObjectURL でダウンロード
- 日本語フォントは `ＭＳ ゴシック` など全角で指定
- 列幅は文字数×係数で概算
