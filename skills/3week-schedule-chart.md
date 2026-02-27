---
name: 3week-schedule-chart
description: 3週間工程表のグラフを3週間刻みで分割作成する
allowed-tools: Read, Bash, WebSearch
---

# 3週間工程表チャート作成

## 概要

工期全体を1つのグラフで表示する代わりに、3週間ごとに分割したグラフを作成する。

## シート構造

- **日付行**: 行4（1日2列構造）
- **計画%**: 行24
- **実施%**: 行25
- **週目**: 行26（「1週目」「2週目」...）
- **グラフ配置**: 行28〜34

### 列構造
- 1日 = 2列（日付列 + 空列）
- 1週間 = 14列
- 3週間 = 42列

### 週の日曜日位置（月初め表記あり）
```
12/14 → index 6  (列7)
12/21 → index 20 (列21)
12/28 → index 34 (列35)
01/04 → index 48 (列49)
...
```

## 3週間ブロック定義

```python
blocks = [
    {'name': '1-3週', 'start': 6, 'end': 48, 'width': 1260},   # 12/14〜1/3
    {'name': '4-6週', 'start': 48, 'end': 90, 'width': 1260},  # 1/4〜1/24
    {'name': '7-9週', 'start': 90, 'end': 132, 'width': 1260}, # 1/25〜2/14
    {'name': '10-12週', 'start': 132, 'end': 174, 'width': 1260},
    {'name': '13週', 'start': 174, 'end': 188, 'width': 420},  # 端数
]
```

## Google Sheets API設定

### チャート作成コード

```python
add_requests.append({
    'addChart': {
        'chart': {
            'spec': {
                'basicChart': {
                    'chartType': 'LINE',
                    'legendPosition': 'NO_LEGEND',
                    'axis': [
                        {'position': 'BOTTOM_AXIS'},
                        {'position': 'LEFT_AXIS'}
                    ],
                    'domains': [{
                        'domain': {
                            'sourceRange': {
                                'sources': [{
                                    'sheetId': sheet_id,
                                    'startRowIndex': 3,  # 行4（日付）
                                    'endRowIndex': 4,
                                    'startColumnIndex': block['start'],
                                    'endColumnIndex': block['end']
                                }]
                            }
                        }
                    }],
                    'series': [
                        {
                            'series': {
                                'sourceRange': {
                                    'sources': [{
                                        'sheetId': sheet_id,
                                        'startRowIndex': 23,  # 行24（計画%）
                                        'endRowIndex': 24,
                                        'startColumnIndex': block['start'],
                                        'endColumnIndex': block['end']
                                    }]
                                }
                            },
                            'targetAxis': 'LEFT_AXIS',
                            'color': {'red': 0, 'green': 0, 'blue': 0},  # 黒
                            'lineStyle': {'width': 2},
                            'pointStyle': {'shape': 'CIRCLE', 'size': 5}
                        },
                        {
                            'series': {
                                'sourceRange': {
                                    'sources': [{
                                        'sheetId': sheet_id,
                                        'startRowIndex': 24,  # 行25（実施%）
                                        'endRowIndex': 25,
                                        'startColumnIndex': block['start'],
                                        'endColumnIndex': block['end']
                                    }]
                                }
                            },
                            'targetAxis': 'LEFT_AXIS',
                            'color': {'red': 1, 'green': 0, 'blue': 0},  # 赤
                            'lineStyle': {'width': 2},
                            'pointStyle': {'shape': 'CIRCLE', 'size': 5}
                        }
                    ],
                    'headerCount': 1
                }
            },
            'position': {
                'overlayPosition': {
                    'anchorCell': {
                        'sheetId': sheet_id,
                        'rowIndex': 27,  # 行28
                        'columnIndex': block['anchor_col']
                    },
                    'widthPixels': block['width'],
                    'heightPixels': 180
                }
            }
        }
    }
})
```

## 設定項目

| 項目 | 設定値 |
|------|--------|
| 計画線の色 | 黒 (0,0,0) |
| 実施線の色 | 赤 (1,0,0) |
| 線の太さ | 2 |
| ポインタ形状 | CIRCLE |
| ポインタサイズ | 5 |
| グラフ高さ | 180px |
| 凡例 | なし (NO_LEGEND) |

## 幅の計算

```python
# 列幅を取得してチャート幅を計算
col_width = 30  # この工程表の場合
chart_width = (block['end'] - block['start']) * col_width
# 3週間: 42列 × 30px = 1260px
```

## API制限事項

**「ラベルをテキストとして使用」はAPIから設定不可**

- UI専用機能で、`treatLabelsAsText`プロパティは存在しない
- 手動で各チャートをダブルクリック → 設定 → チェックを入れる必要あり

## 既存チャートの削除

```python
# チャートIDを取得
spreadsheet = service.spreadsheets().get(spreadsheetId=spreadsheet_id).execute()
for sheet in spreadsheet.get('sheets', []):
    if sheet['properties']['sheetId'] == target_sheet_id:
        chart_ids = [chart['chartId'] for chart in sheet.get('charts', [])]

# 削除リクエスト
delete_requests = [{'deleteEmbeddedObject': {'objectId': cid}} for cid in chart_ids]
```

## 関連スプレッドシート

※スプレッドシートID・シートIDは別途管理
