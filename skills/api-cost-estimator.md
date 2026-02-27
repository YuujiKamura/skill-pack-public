---
name: api-cost-estimator
description: AI APIの呼び出しコストを事前に見積もり、処理中に追跡し、処理後に報告するフレームワーク。
allowed-tools: Read, Write, Edit, Bash(node:*), Bash(npx:*)
---

# AI API コスト見積もりフレームワーク

## 概要

AI APIを使うツールで、処理前にコストを見積もり、ユーザー確認後に実行する仕組み。

## 3フェーズ

### 1. 処理前（見積もり）

```typescript
interface CostEstimate {
  inputTokens: number;      // 推定入力トークン
  outputTokens: number;     // 推定出力トークン
  apiCalls: number;         // API呼び出し回数
  estimatedCostUSD: number; // 推定コスト（ドル）
  estimatedCostJPY: number; // 推定コスト（円）
  model: string;            // 使用モデル
  breakdown: {              // 内訳
    images?: number;
    textChars?: number;
    tokensPerImage?: number;
  };
}

// 見積もり表示例
📊 処理前見積もり
├─ 画像: 100枚
├─ 推定トークン: 入力258,000 / 出力5,000
├─ API呼び出し: 3回（MAGI方式）
├─ 推定コスト: $0.02 (約3円)
└─ [Y] 実行 / [N] キャンセル / [D] 詳細
```

### 2. 処理中（追跡）

```typescript
interface CostTracker {
  startTime: number;
  currentApiCalls: number;
  actualInputTokens: number;
  actualOutputTokens: number;
  currentCostUSD: number;

  onProgress?: (progress: number, cost: number) => void;
  onApiCall?: (callNumber: number, tokens: { input: number; output: number }) => void;
}

// 進捗表示例
⏳ 処理中 [████████░░] 80%
├─ 経過: 12.3s
├─ API呼び出し: 2/3回
├─ 現在コスト: $0.015
└─ [Ctrl+C] 中断
```

### 3. 処理後（レポート）

```typescript
interface CostReport {
  estimate: CostEstimate;   // 見積もり
  actual: {                 // 実績
    inputTokens: number;
    outputTokens: number;
    apiCalls: number;
    costUSD: number;
    costJPY: number;
    duration: number;       // 処理時間(ms)
  };
  accuracy: number;         // 見積もり精度(%)
  sessionTotal: number;     // セッション累計
}

// レポート表示例
✅ 処理完了
├─ 実際のトークン: 入力245,000 / 出力4,800
├─ 実際のコスト: $0.019 (約2.8円)
├─ 見積もり精度: 95%
├─ 処理時間: 15.2秒
└─ セッション累計: $0.05 (約7.5円)
```

## モデル別料金テーブル

```typescript
const PRICING = {
  'gemini-2.5-flash': {
    input: 0.075,   // $/1M tokens
    output: 0.30,
    imageTokens: 258,  // 768px換算
  },
  'gemini-2.5-pro': {
    input: 1.25,
    output: 10.0,
    imageTokens: 258,
  },
  'gpt-4o': {
    input: 2.50,
    output: 10.0,
    imageTokens: 170,  // low detail
  },
  'gpt-4o-mini': {
    input: 0.15,
    output: 0.60,
    imageTokens: 170,
  },
  'claude-sonnet-4': {
    input: 3.0,
    output: 15.0,
    imageTokens: 1600, // 概算
  },
};

const USD_TO_JPY = 150; // 適宜更新
```

## 画像トークン推定

```typescript
function estimateImageTokens(
  imageSize: number,      // bytes
  model: string
): number {
  const pricing = PRICING[model];

  // 画像サイズに応じた補正
  // 小さい画像: 基準値
  // 大きい画像: 最大4倍程度
  const sizeMultiplier = Math.min(4, imageSize / 500000 + 1);

  return Math.ceil(pricing.imageTokens * sizeMultiplier);
}
```

## ドライラン機能

```typescript
async function dryRun(
  images: Buffer[],
  model: string,
  apiCallMultiplier: number = 1  // MAGI方式なら3
): Promise<CostEstimate> {
  const imageTokens = images.reduce(
    (sum, img) => sum + estimateImageTokens(img.length, model),
    0
  );

  const pricing = PRICING[model];
  const inputTokens = imageTokens * apiCallMultiplier;
  const outputTokens = images.length * 50 * apiCallMultiplier; // 推定

  const costUSD =
    (inputTokens / 1_000_000) * pricing.input +
    (outputTokens / 1_000_000) * pricing.output;

  return {
    inputTokens,
    outputTokens,
    apiCalls: apiCallMultiplier,
    estimatedCostUSD: costUSD,
    estimatedCostJPY: costUSD * USD_TO_JPY,
    model,
    breakdown: {
      images: images.length,
      tokensPerImage: pricing.imageTokens,
    }
  };
}
```

## CLI表示ユーティリティ

```typescript
function showEstimate(estimate: CostEstimate): void {
  console.log(`
📊 処理前見積もり
├─ 画像: ${estimate.breakdown.images}枚
├─ 推定トークン: 入力${estimate.inputTokens.toLocaleString()} / 出力${estimate.outputTokens.toLocaleString()}
├─ API呼び出し: ${estimate.apiCalls}回
├─ モデル: ${estimate.model}
└─ 推定コスト: $${estimate.estimatedCostUSD.toFixed(4)} (約${Math.ceil(estimate.estimatedCostJPY)}円)
`);
}

function askConfirmation(): Promise<boolean> {
  return new Promise(resolve => {
    process.stdout.write('[Y] 実行 / [N] キャンセル: ');
    process.stdin.once('data', data => {
      resolve(data.toString().trim().toLowerCase() === 'y');
    });
  });
}
```

## 使用例

```typescript
import { dryRun, showEstimate, askConfirmation, CostTracker } from './cost-estimator';

async function processPhotos(images: Buffer[]) {
  // 1. 見積もり
  const estimate = await dryRun(images, 'gemini-2.5-flash', 3);
  showEstimate(estimate);

  // 2. 確認
  if (!await askConfirmation()) {
    console.log('キャンセルしました');
    return;
  }

  // 3. 実行（トラッキング付き）
  const tracker = new CostTracker(estimate);
  tracker.onProgress = (p, cost) => {
    process.stdout.write(`\r⏳ [${progressBar(p)}] ${p}% - $${cost.toFixed(4)}`);
  };

  const result = await runWithTracking(images, tracker);

  // 4. レポート
  tracker.showReport();
}
```
