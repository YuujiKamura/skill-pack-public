---
name: localstorage-cache
description: Use when this skill's workflow applies
---

# LocalStorage Cache パターン

設定やデータをlocalStorageに永続化するパターン集。

## 基本パターン

```typescript
const STORAGE_KEY = 'myapp_settings';

// デフォルト値
const defaultConfig = {
  theme: 'light',
  language: 'ja',
  pageSize: 20,
};

// 読み込み（エラーハンドリング付き）
export const loadConfig = () => {
  try {
    const cached = localStorage.getItem(STORAGE_KEY);
    if (cached) {
      // デフォルト値とマージ（新しいフィールドに対応）
      return { ...defaultConfig, ...JSON.parse(cached) };
    }
  } catch (e) {
    console.error('設定読み込みエラー:', e);
  }
  return defaultConfig;
};

// 保存
export const saveConfig = (config: typeof defaultConfig) => {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(config));
  } catch (e) {
    console.error('設定保存エラー:', e);
  }
};
```

## Reactでの使用

```tsx
const [config, setConfig] = useState(loadConfig);

const updateConfig = (updates: Partial<Config>) => {
  const newConfig = { ...config, ...updates };
  setConfig(newConfig);
  saveConfig(newConfig);
};
```

## 配列データの管理（件数制限付き）

```typescript
const MAX_ITEMS = 50;
const STORAGE_KEY = 'myapp_history';

export const getItems = (): Item[] => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    return saved ? JSON.parse(saved) : [];
  } catch {
    return [];
  }
};

export const saveItem = (item: Item) => {
  const items = getItems();
  const updated = [item, ...items.filter(i => i.id !== item.id)];
  localStorage.setItem(
    STORAGE_KEY,
    JSON.stringify(updated.slice(0, MAX_ITEMS))
  );
};

export const deleteItem = (id: string) => {
  const items = getItems().filter(i => i.id !== id);
  localStorage.setItem(STORAGE_KEY, JSON.stringify(items));
};
```

## バージョン管理付きマイグレーション

```typescript
const STORAGE_KEY = 'myapp_data_v2';  // バージョン付きキー
const LEGACY_KEY = 'myapp_data';       // 旧キー

export const migrateData = () => {
  const legacy = localStorage.getItem(LEGACY_KEY);
  if (!legacy) return;

  try {
    const oldData = JSON.parse(legacy);

    // 新形式に変換
    const newData = oldData.map((item: any) => ({
      id: item.id,
      name: item.name,
      // 新フィールド追加
      createdAt: item.timestamp || Date.now(),
      version: 2,
    }));

    localStorage.setItem(STORAGE_KEY, JSON.stringify(newData));
    localStorage.removeItem(LEGACY_KEY);  // 旧データ削除
  } catch (e) {
    console.error('マイグレーションエラー:', e);
  }
};
```

## カスタムフック

```typescript
function useLocalStorage<T>(key: string, defaultValue: T) {
  const [value, setValue] = useState<T>(() => {
    try {
      const saved = localStorage.getItem(key);
      return saved ? JSON.parse(saved) : defaultValue;
    } catch {
      return defaultValue;
    }
  });

  const setStoredValue = (newValue: T | ((prev: T) => T)) => {
    setValue(prev => {
      const resolved = typeof newValue === 'function'
        ? (newValue as (prev: T) => T)(prev)
        : newValue;
      localStorage.setItem(key, JSON.stringify(resolved));
      return resolved;
    });
  };

  return [value, setStoredValue] as const;
}

// 使用
const [settings, setSettings] = useLocalStorage('settings', { theme: 'light' });
```

## 容量チェック

```typescript
const getStorageSize = () => {
  let total = 0;
  for (const key in localStorage) {
    if (localStorage.hasOwnProperty(key)) {
      total += localStorage[key].length * 2;  // UTF-16
    }
  }
  return total;  // bytes
};

const isStorageNearFull = () => {
  return getStorageSize() > 4 * 1024 * 1024;  // 4MB警告
};
```

## ポイント

- **try-catch必須**: パースエラー、容量オーバーに対応
- **デフォルト値マージ**: `{ ...default, ...parsed }` で新フィールド対応
- **件数制限**: `slice(0, MAX)` でストレージ肥大化防止
- **バージョン管理**: キー名にバージョン番号を含める
- **容量**: localStorageは通常5MB制限
