---
name: api-request-tracker
description: Use when this skill's workflow applies
---

# API Request Tracker パターン

非同期リクエストの追跡・キャンセル・レース条件対策。

## 問題

```
ユーザーがボタンを連打 → 複数リクエストが同時発火
→ 古いレスポンスが新しいレスポンスを上書き（レース条件）
```

## 解決パターン

```tsx
const requestCounter = useRef(0);
const activeRequestId = useRef(0);

const fetchData = async () => {
  // 新しいリクエストIDを発行
  const requestId = ++requestCounter.current;
  activeRequestId.current = requestId;

  setLoading(true);

  try {
    const result = await apiCall();

    // このリクエストがまだアクティブか確認
    if (activeRequestId.current !== requestId) {
      console.log('リクエストがキャンセルされました');
      return;
    }

    // 結果を反映
    setData(result);

  } catch (error) {
    if (activeRequestId.current !== requestId) return;
    setError(error);

  } finally {
    if (activeRequestId.current === requestId) {
      setLoading(false);
    }
  }
};
```

## プログレスコールバック付き

```tsx
const analyzeWithProgress = async (input: string) => {
  const requestId = ++requestCounter.current;
  activeRequestId.current = requestId;

  setProgress(0);
  setResults([]);

  try {
    await longRunningTask(input, (progress, partialResult) => {
      // キャンセル済みなら無視
      if (activeRequestId.current !== requestId) return;

      setProgress(progress);
      setResults(prev => [...prev, partialResult]);
    });

    if (activeRequestId.current !== requestId) return;
    setStatus('complete');

  } finally {
    if (activeRequestId.current === requestId) {
      setLoading(false);
    }
  }
};
```

## AbortController連携

```tsx
const abortControllerRef = useRef<AbortController | null>(null);

const fetchData = async () => {
  // 前のリクエストをキャンセル
  abortControllerRef.current?.abort();

  const controller = new AbortController();
  abortControllerRef.current = controller;

  try {
    const response = await fetch(url, {
      signal: controller.signal
    });
    const data = await response.json();
    setData(data);

  } catch (error) {
    if (error.name === 'AbortError') {
      console.log('リクエストがキャンセルされました');
      return;
    }
    setError(error);
  }
};

// クリーンアップ
useEffect(() => {
  return () => {
    abortControllerRef.current?.abort();
  };
}, []);
```

## カスタムAbortSignal

```tsx
const createAbortSignal = (requestId: number) => ({
  get cancelled() {
    return activeRequestId.current !== requestId;
  }
});

// API側で使用
const apiCall = async (data: any, signal: { cancelled: boolean }) => {
  for (const chunk of chunks) {
    if (signal.cancelled) {
      throw new Error('Cancelled');
    }
    await processChunk(chunk);
  }
};

// 呼び出し側
const requestId = ++requestCounter.current;
activeRequestId.current = requestId;
await apiCall(data, createAbortSignal(requestId));
```

## レート制限対応

```tsx
const [isRateLimited, setIsRateLimited] = useState(false);

const fetchData = async () => {
  try {
    const result = await apiCall();
    setIsRateLimited(false);  // 成功したらリセット
    return result;

  } catch (error) {
    if (error.message?.includes('429')) {
      setIsRateLimited(true);
      // UIに警告表示
    }
    throw error;
  }
};

// レート制限中は軽量モードで実行
const mode = isRateLimited ? 'lite' : 'full';
```

## デバウンス付きリクエスト

```tsx
const debounceRef = useRef<NodeJS.Timeout>();

const searchWithDebounce = (query: string) => {
  clearTimeout(debounceRef.current);

  debounceRef.current = setTimeout(async () => {
    const requestId = ++requestCounter.current;
    activeRequestId.current = requestId;

    const results = await search(query);

    if (activeRequestId.current === requestId) {
      setResults(results);
    }
  }, 300);  // 300ms待機
};
```

## ポイント

- **useRef**: リクエストIDは再レンダリングを起こさないためuseRef
- **3箇所でチェック**: 成功時、エラー時、finally内
- **finally内もチェック**: loading状態の不整合を防ぐ
- **AbortController**: fetch APIの標準キャンセル機構
- **デバウンス**: 連続入力時の無駄なリクエスト防止
