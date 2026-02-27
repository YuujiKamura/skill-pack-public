---
name: pdf-combine
description: PDF結合・ページ差し替えのバッチファイル化。(1) 複数PDFを結合するバッチ作成、(2) 特定ページの差し替え、(3) 番号順自動結合、(4) ページ回転。PDF結合、マージ、ページ差し替え、バッチファイル、再結合と言われた時に使用。
---

# PDF結合バッチファイル

## 基本パターン：指定ファイルを結合

フォルダにバッチファイルを置いて、ダブルクリックで再結合できるようにする。

```bat
@echo off
chcp 65001 > nul
python -c "
from pypdf import PdfReader, PdfWriter
import os

folder = r'%~dp0'
files = [
    os.path.join(folder, 'ファイル1.pdf'),
    os.path.join(folder, 'ファイル2.pdf')
]
output = os.path.join(folder, '結合済み.pdf')

writer = PdfWriter()
for f in files:
    reader = PdfReader(f)
    for page in reader.pages:
        writer.add_page(page)
    print(f'{os.path.basename(f)}: {len(reader.pages)}ページ')

with open(output, 'wb') as out:
    writer.write(out)

print(f'結合完了 → {os.path.basename(output)}')
"
pause
```

## 番号順自動結合

ファイル名の先頭に番号をつけて順番を制御：

```
01_表紙.pdf
02_目次.pdf
03_本文.pdf
```

```bat
@echo off
chcp 65001 > nul
python -c "
from pypdf import PdfReader, PdfWriter
import os
import glob

folder = r'%~dp0'
files = sorted(glob.glob(os.path.join(folder, '[0-9][0-9]_*.pdf')))
output = os.path.join(folder, '結合済み.pdf')

writer = PdfWriter()
for f in files:
    reader = PdfReader(f)
    for page in reader.pages:
        writer.add_page(page)
    print(f'{os.path.basename(f)}: {len(reader.pages)}ページ')

with open(output, 'wb') as out:
    writer.write(out)

print(f'結合完了 → {os.path.basename(output)}')
"
pause
```

## ページ差し替え

結合済みPDFの特定ページだけを差し替える：

```python
from pypdf import PdfReader, PdfWriter

def replace_page(combined_pdf, page_num, new_pdf, output_pdf=None):
    """結合済みPDFの特定ページを差し替え

    Args:
        combined_pdf: 結合済みPDFのパス
        page_num: 差し替えるページ番号（1始まり）
        new_pdf: 差し替え用PDFのパス
        output_pdf: 出力先（省略で上書き）
    """
    if output_pdf is None:
        output_pdf = combined_pdf

    reader = PdfReader(combined_pdf)
    new_reader = PdfReader(new_pdf)
    writer = PdfWriter()

    for i, page in enumerate(reader.pages):
        if i == page_num - 1:
            writer.add_page(new_reader.pages[0])
        else:
            writer.add_page(page)

    with open(output_pdf, 'wb') as f:
        writer.write(f)

# 例: 2ページ目を差し替え
replace_page('結合済み.pdf', 2, '修正版.pdf')
```

## ページ回転

A4縦→A4横など、全ページを回転：

```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader('input.pdf')
writer = PdfWriter()

for page in reader.pages:
    page.rotate(90)  # 右に90度
    writer.add_page(page)

with open('output.pdf', 'wb') as f:
    writer.write(f)
```

## 2-up変換（A4縦に2ページ上下配置）

マニフェスト伝票など、2ページを1ページにまとめて印刷コスト削減：

```python
from pypdf import PdfReader, PdfWriter, PageObject, Transformation

def convert_2up(input_path, output_path):
    """PDFを2-up変換（A4縦に2ページ上下配置）

    Args:
        input_path: 入力PDFパス
        output_path: 出力PDFパス
    """
    A4_WIDTH = 595.28
    A4_HEIGHT = 841.89

    reader = PdfReader(input_path)
    writer = PdfWriter()

    # 元ページのサイズ
    orig_width = float(reader.pages[0].mediabox.width)
    orig_height = float(reader.pages[0].mediabox.height)

    # スケール計算（A4の半分に収まるように）
    scale_x = A4_WIDTH / orig_width
    scale_y = (A4_HEIGHT / 2) / orig_height
    scale = min(scale_x, scale_y) * 0.95  # 5%マージン

    for i in range(0, len(reader.pages), 2):
        new_page = PageObject.create_blank_page(width=A4_WIDTH, height=A4_HEIGHT)

        # 上半分（奇数ページ）
        top_page = reader.pages[i]
        top_x = (A4_WIDTH - orig_width * scale) / 2
        top_y = A4_HEIGHT / 2 + (A4_HEIGHT / 2 - orig_height * scale) / 2
        new_page.merge_transformed_page(
            top_page,
            Transformation().scale(scale).translate(top_x, top_y)
        )

        # 下半分（偶数ページ）
        if i + 1 < len(reader.pages):
            bottom_page = reader.pages[i + 1]
            bottom_x = (A4_WIDTH - orig_width * scale) / 2
            bottom_y = (A4_HEIGHT / 2 - orig_height * scale) / 2
            new_page.merge_transformed_page(
                bottom_page,
                Transformation().scale(scale).translate(bottom_x, bottom_y)
            )

        writer.add_page(new_page)

    with open(output_path, 'wb') as f:
        writer.write(f)

    print(f"{len(reader.pages)}ページ → {len(writer.pages)}ページ")

# 例: マニフェスト伝票を2-up変換
convert_2up('マニデン.pdf', 'マニデン_2up.pdf')
```

### 用途
- マニフェスト伝票（A4横10枚 → A4縦5枚）
- 印刷コスト・紙の削減
- ファイリング効率化

## 変更履歴
- 2026-01-15: 2-up変換機能を追加
