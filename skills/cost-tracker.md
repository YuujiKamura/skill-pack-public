---
name: cost-tracker
description: Use when this skill's workflow applies
---

# Cost Tracker パターン

API利用コストをlocalStorageで追跡・可視化するパターン。

## 使用場面

- 外部API（Gemini, OpenAI等）の利用コスト追跡
- 日別・モデル別のコスト集計
- 多通貨対応のコスト表示

## 型定義

```typescript
interface CostEntry {
  id: string;
  timestamp: number;
  model: string;
  callCount: number;
  estimatedCost: number;  // USD基準
  imageCount: number;     // 画像枚数など
}

interface DailyCost {
  date: string;
  totalCost: number;
  callCount: number;
}
```

## モデル別コスト設定

```typescript
const COST_PER_CALL: Record<string, number> = {
  'gemini-flash': 0.0005,
  'gemini-pro': 0.005,
  'gpt-4': 0.03,
  'default': 0.001
};
```

## 通貨変換

```typescript
const EXCHANGE_RATES: Record<string, number> = {
  'USD': 1,
  'JPY': 150,
  'EUR': 0.92,
};

// ブラウザ言語から通貨を判定
const getCurrency = () => {
  const lang = navigator.language || 'en-US';

  if (lang.startsWith('ja')) {
    return { code: 'JPY', symbol: '¥', rate: 150 };
  }
  return { code: 'USD', symbol: '$', rate: 1 };
};

// コストをローカル通貨でフォーマット
const formatCost = (usdAmount: number): string => {
  const currency = getCurrency();
  const localAmount = usdAmount * currency.rate;

  if (currency.code === 'JPY') {
    return localAmount < 1
      ? currency.symbol + localAmount.toFixed(2)
      : currency.symbol + Math.round(localAmount).toLocaleString();
  }

  return currency.symbol + localAmount.toFixed(4);
};
```

## コスト記録

```typescript
const STORAGE_KEY = 'api_cost_history';
const MAX_ENTRIES = 1000;

const getCostHistory = (): CostEntry[] => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    return saved ? JSON.parse(saved) : [];
  } catch {
    return [];
  }
};

const saveCostEntry = (
  model: string,
  imageCount: number = 1,
  isFreeTier: boolean = false
): CostEntry => {
  const history = getCostHistory();
  const costPerCall = isFreeTier ? 0 : (COST_PER_CALL[model] || COST_PER_CALL['default']);

  const entry: CostEntry = {
    id: crypto.randomUUID(),
    timestamp: Date.now(),
    model,
    callCount: 1,
    estimatedCost: costPerCall * imageCount,
    imageCount
  };

  history.push(entry);

  // 件数制限
  const trimmed = history.slice(-MAX_ENTRIES);
  localStorage.setItem(STORAGE_KEY, JSON.stringify(trimmed));

  return entry;
};
```

## 集計関数

```typescript
// 合計コスト
const getTotalCost = (): number => {
  return getCostHistory().reduce((sum, entry) => sum + entry.estimatedCost, 0);
};

// 今日のコスト
const getTodayCost = (): number => {
  const today = new Date().toDateString();
  return getCostHistory()
    .filter(entry => new Date(entry.timestamp).toDateString() === today)
    .reduce((sum, entry) => sum + entry.estimatedCost, 0);
};

// 日別集計
const getDailyCosts = (days: number = 7): DailyCost[] => {
  const history = getCostHistory();
  const result: Map<string, DailyCost> = new Map();

  // 過去N日分の日付を生成
  for (let i = days - 1; i >= 0; i--) {
    const date = new Date();
    date.setDate(date.getDate() - i);
    const dateStr = date.toISOString().split('T')[0];
    result.set(dateStr, { date: dateStr, totalCost: 0, callCount: 0 });
  }

  // 履歴からコストを集計
  history.forEach(entry => {
    const dateStr = new Date(entry.timestamp).toISOString().split('T')[0];
    if (result.has(dateStr)) {
      const daily = result.get(dateStr)!;
      daily.totalCost += entry.estimatedCost;
      daily.callCount += entry.callCount;
    }
  });

  return Array.from(result.values());
};

// モデル別集計
const getModelBreakdown = () => {
  const history = getCostHistory();
  const breakdown: Map<string, { cost: number; count: number }> = new Map();

  history.forEach(entry => {
    const existing = breakdown.get(entry.model) || { cost: 0, count: 0 };
    existing.cost += entry.estimatedCost;
    existing.count += entry.callCount;
    breakdown.set(entry.model, existing);
  });

  return Array.from(breakdown.entries()).map(([model, data]) => ({
    model,
    ...data
  }));
};
```

## API呼び出し時の使用例

```typescript
const analyzeImage = async (imageData: string) => {
  const result = await geminiApi.analyze(imageData);

  // コストを記録
  saveCostEntry('gemini-flash', 1, false);

  return result;
};
```

## ダッシュボード表示例

```tsx
const CostDashboard = () => {
  const todayCost = getTodayCost();
  const totalCost = getTotalCost();
  const dailyCosts = getDailyCosts(7);

  return (
    <div>
      <p>今日: {formatCost(todayCost)}</p>
      <p>累計: {formatCost(totalCost)}</p>

      {/* 日別グラフ */}
      <div className="flex gap-1 h-20 items-end">
        {dailyCosts.map(day => (
          <div
            key={day.date}
            style={{ height: `${(day.totalCost / maxCost) * 100}%` }}
            className="bg-blue-500 flex-1"
          />
        ))}
      </div>
    </div>
  );
};
```

## ポイント

- **USD基準**: 内部はUSDで管理、表示時に変換
- **件数制限**: `slice(-MAX)` でストレージ肥大化防止
- **無料枠対応**: `isFreeTier` フラグでコスト0記録
- **日付集計**: `toDateString()` で日単位のグループ化
- **モデル別**: 異なる単価のモデルを区別して集計
