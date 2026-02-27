---
name: theme-switcher
description: Use when this skill's workflow applies
---

# Theme Switcher パターン

ライト/ダークテーマ切替をlocalStorageで永続化するパターン。

## 使用場面

- アプリ全体のテーマ切替
- コンポーネント単位のテーマ管理
- ユーザー設定の永続化

## 実装パターン

```tsx
import { useState } from 'react';
import { Sun, Moon } from 'lucide-react';

// ストレージキー
const THEME_KEY = 'app_theme';

// ユーティリティ関数
const getStoredTheme = () => localStorage.getItem(THEME_KEY) || 'light';
const setStoredTheme = (theme: string) => localStorage.setItem(THEME_KEY, theme);

// コンポーネント内で使用
const MyComponent = () => {
  const [theme, setTheme] = useState(getStoredTheme);

  const toggleTheme = () => {
    const newTheme = theme === 'light' ? 'dark' : 'light';
    setTheme(newTheme);
    setStoredTheme(newTheme);
  };

  const isDark = theme === 'dark';

  return (
    <div className={isDark ? 'bg-slate-950 text-white' : 'bg-white text-gray-900'}>
      <button onClick={toggleTheme}>
        {isDark ? <Sun size={18} /> : <Moon size={18} />}
      </button>
      {/* コンテンツ */}
    </div>
  );
};
```

## Tailwindでの条件付きクラス

```tsx
// 背景色
className={isDark ? 'bg-slate-900' : 'bg-white'}

// テキスト色
className={isDark ? 'text-gray-200' : 'text-gray-800'}

// ボーダー
className={isDark ? 'border-slate-700' : 'border-gray-300'}

// ホバー
className={isDark ? 'hover:bg-slate-800' : 'hover:bg-gray-50'}

// 複合
className={`p-4 rounded ${isDark ? 'bg-slate-800 text-white' : 'bg-gray-100 text-gray-900'}`}
```

## カスタムフック版

```tsx
const useTheme = (storageKey = 'app_theme') => {
  const [theme, setThemeState] = useState(() =>
    localStorage.getItem(storageKey) || 'light'
  );

  const setTheme = (newTheme: 'light' | 'dark') => {
    setThemeState(newTheme);
    localStorage.setItem(storageKey, newTheme);
  };

  const toggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };

  return {
    theme,
    isDark: theme === 'dark',
    isLight: theme === 'light',
    setTheme,
    toggleTheme,
  };
};

// 使用
const { isDark, toggleTheme } = useTheme();
```

## システム設定連動

```tsx
const getSystemTheme = () =>
  window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';

const getStoredTheme = () =>
  localStorage.getItem(THEME_KEY) || getSystemTheme();
```

## ポイント

- デフォルトは `'light'`（または `getSystemTheme()`）
- `useState(getStoredTheme)` で初期値をストレージから取得
- 変更時は即座にストレージに保存
- Tailwindの条件付きクラスでスタイル適用
