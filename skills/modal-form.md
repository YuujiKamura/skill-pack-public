---
name: modal-form
description: Use when this skill's workflow applies
---

# Modal Form パターン

Controlled Componentパターンによる編集モーダルフォーム。

## 使用場面

- アイテム詳細の編集
- 設定画面
- 確認ダイアログ

## 基本構造

```tsx
interface EditFormProps<T> {
  item: T | null;           // 編集対象（nullで非表示）
  isOpen: boolean;          // 表示状態
  onSave: (id: string, updates: Partial<T>) => void;
  onClose: () => void;
  // オプショナルなアクション
  onDelete?: (id: string) => void;
  onAction?: (item: T) => void;
}

const EditForm = <T extends { id: string }>({
  item,
  isOpen,
  onSave,
  onClose,
  onDelete,
}: EditFormProps<T>) => {
  // ローカル状態（フォーム値）
  const [formData, setFormData] = useState<Partial<T>>({});

  // アイテム変更時にフォームをリセット
  useEffect(() => {
    if (item && isOpen) {
      setFormData({ ...item });
    }
  }, [item, isOpen]);

  const handleSave = () => {
    if (!item) return;
    onSave(item.id, formData);
    onClose();
  };

  if (!isOpen || !item) return null;

  return (
    <div className="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4">
      <div className="bg-white rounded-xl w-full max-w-md max-h-[90vh] overflow-y-auto">
        {/* ヘッダー */}
        <div className="sticky top-0 bg-white border-b p-3 flex justify-between">
          <h3 className="font-bold">編集</h3>
          <button onClick={onClose}>✕</button>
        </div>

        {/* フォーム本体 */}
        <div className="p-4 space-y-4">
          {/* 入力フィールド */}
        </div>

        {/* フッター */}
        <div className="sticky bottom-0 bg-white border-t p-3 flex gap-2">
          <button onClick={onClose}>キャンセル</button>
          <button onClick={handleSave}>保存</button>
        </div>
      </div>
    </div>
  );
};
```

## 親コンポーネントでの使用

```tsx
const ParentComponent = () => {
  const [editingItem, setEditingItem] = useState<Item | null>(null);

  return (
    <>
      {/* リスト表示 */}
      {items.map(item => (
        <div key={item.id} onClick={() => setEditingItem(item)}>
          {item.name}
        </div>
      ))}

      {/* 編集モーダル */}
      <EditForm
        item={editingItem}
        isOpen={!!editingItem}
        onSave={(id, updates) => {
          updateItem(id, updates);
          setEditingItem(null);
        }}
        onClose={() => setEditingItem(null)}
      />
    </>
  );
};
```

## スクロール可能なモーダル

```tsx
// オーバーレイ
<div className="fixed inset-0 bg-black/80 z-50 flex items-center justify-center p-4 overflow-y-auto">

// モーダル本体
<div className="bg-white rounded-xl w-full max-w-md max-h-[90vh] overflow-y-auto">

// ヘッダー・フッターを固定
<div className="sticky top-0 bg-white border-b ...">
<div className="sticky bottom-0 bg-white border-t ...">
```

## ポイント

- **Controlled Component**: 親が`isOpen`と`item`を管理
- **useEffect同期**: アイテム変更時にフォーム値をリセット
- **nullチェック**: `if (!isOpen || !item) return null`
- **即座クローズ**: onSave後に親がsetEditingItem(null)
- **z-index管理**: モーダルは高いz-indexを使用
