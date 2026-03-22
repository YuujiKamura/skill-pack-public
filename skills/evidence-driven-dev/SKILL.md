---
description: 証拠駆動開発。「改善しました」ではなくテストで検証可能な成果物を要求する開発フロー。(1) 修正前に失敗するテストを追加、(2) 修正後にテストがパス、(3) ベースライン比較、(4) 計測可能性の確保。証拠、検証、テスト駆動、TDD、エビデンス、計測と言われた時に使用。
keywords: [証拠, 検証, TDD, テスト駆動, エビデンス, 計測, ベースライン]
---

# 証拠駆動開発（Evidence-Driven Development）

「改善しました」「直しました」は証拠にならない。

## 原則

```
❌ 信頼できない報告
「パフォーマンスを改善しました」
「バグを修正しました」
「コードをきれいにしました」

✅ 検証可能な報告
「レスポンス時間が 500ms → 120ms になりました（テスト結果添付）」
「このテストが通るようになりました（before: FAIL, after: PASS）」
「循環的複雑度が 15 → 8 になりました（計測結果添付）」
```

## 開発フロー

### 1. 失敗するテストを先に書く

```typescript
// バグ修正の場合
it('reasoningがない場合、推測と明示する', () => {
  const result = buildExplainPrompt({ reasoning: '' });
  expect(result.prompt).toContain('推測');  // ← まだ失敗する
});
```

### 2. 修正を実施

```typescript
// 実装を修正
const prompt = hasReasoning
  ? '根拠に基づいて...'
  : '推測になるが...';  // ← これで通る
```

### 3. テストがパスすることを確認

```bash
npm test -- tests/explainService.test.ts
# ✓ reasoningがない場合、推測と明示する
```

### 4. 証拠を残す

```markdown
## 修正内容
- reasoningがない場合に「推測」と明示するよう修正

## 証拠
- テスト追加: tests/explainService.test.ts (14件パス)
- 振る舞いテスト: tests/explainService.behavior.test.ts (3件パス)
```

## 計測可能性の確保（5ステップ）

```
Inject(計測) → Trace(追跡) → Collect(収集) → Understand(理解) → Systematize(体系化)
```

### 1. Inject - 計測ポイントを埋め込む

```typescript
const start = performance.now();
const result = await heavyProcess();
const duration = performance.now() - start;
console.log(`[PERF] heavyProcess: ${duration}ms`);
```

### 2. Trace - 実行パスを追跡

```typescript
onLog?.(`[STEP] 1. データ取得開始`, 'info');
onLog?.(`[STEP] 2. 解析実行`, 'info');
onLog?.(`[STEP] 3. 結果保存`, 'info');
```

### 3. Collect - データを収集

```typescript
trackUsage(model, prompt, response, count, 'functionName');
```

### 4. Understand - 傾向を把握

```
過去7日間のレスポンス時間:
- 平均: 250ms
- P95: 800ms
- 最大: 2500ms
```

### 5. Systematize - 体系化してアクション

```
P95が500msを超えたらアラート
→ 原因調査
→ 改善実施
→ 効果測定
```

## テストの種類と証拠レベル

| テスト種類 | 証拠レベル | 用途 |
|-----------|-----------|------|
| 単体テスト | 高 | ロジックの正しさ |
| 統合テスト | 高 | コンポーネント連携 |
| 振る舞いテスト | 中 | AI/LLMの意図通りの動作 |
| 手動テスト | 低 | 最終確認（証拠残しにくい） |

## 振る舞いテストの例（Claude CLI使用）

```typescript
// APIコストなしでLLMの振る舞いを検証
it('reasoningありなら推測と言わない', async () => {
  const prompt = buildExplainPrompt(analysisWithReasoning).prompt;
  const response = await askClaude(prompt);

  expect(response).not.toContain('推測');
});
```

## コミットメッセージでの証拠

```
fix: reasoningを参照するよう修正

- テスト追加: 14件
- 振る舞いテスト: 3件
- ビルド: OK
```

## 参考

- 元記事: https://qiita.com/hisaho/items/5bd0096ca33d5ea41acb
- 関連Issue: #172
