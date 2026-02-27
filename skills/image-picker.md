---
name: image-picker
description: Use when this skill's workflow applies
---

# ImagePicker コンポーネント

カメラ撮影とファイル選択を統合した画像入力コンポーネント。

## 使用場面

- フォーム内での画像アップロード
- モバイル対応アプリでのカメラ/ギャラリー選択
- 画像編集機能付きフォーム

## 実装パターン

```tsx
import React, { useRef } from 'react';
import { Camera, FolderOpen } from 'lucide-react';

interface ImagePickerProps {
  onImageSelect: (base64: string, dataUrl: string) => void;
  compact?: boolean;  // コンパクトモード（アイコンのみ）
  disabled?: boolean;
}

const ImagePicker: React.FC<ImagePickerProps> = ({
  onImageSelect,
  compact = false,
  disabled = false
}) => {
  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onloadend = () => {
      const dataUrl = reader.result as string;
      const base64 = dataUrl.split(',')[1];
      onImageSelect(base64, dataUrl);
    };
    reader.readAsDataURL(file);
    e.target.value = '';  // リセットして同じファイル再選択可能に
  };

  if (compact) {
    return (
      <div className="flex gap-1.5">
        <label className="p-2 rounded cursor-pointer hover:bg-gray-100">
          <Camera size={16} />
          <input
            type="file"
            accept="image/*"
            capture="environment"
            onChange={handleFileChange}
            disabled={disabled}
            className="hidden"
          />
        </label>
        <label className="p-2 rounded cursor-pointer hover:bg-gray-100">
          <FolderOpen size={16} />
          <input
            type="file"
            accept="image/*"
            onChange={handleFileChange}
            disabled={disabled}
            className="hidden"
          />
        </label>
      </div>
    );
  }

  return (
    <div className="flex items-center justify-center gap-6">
      <label className="flex flex-col items-center gap-2 p-4 rounded-xl cursor-pointer hover:bg-gray-100">
        <Camera size={24} />
        <span className="text-xs">カメラ</span>
        <input
          type="file"
          accept="image/*"
          capture="environment"
          onChange={handleFileChange}
          disabled={disabled}
          className="hidden"
        />
      </label>
      <label className="flex flex-col items-center gap-2 p-4 rounded-xl cursor-pointer hover:bg-gray-100">
        <FolderOpen size={24} />
        <span className="text-xs">ギャラリー</span>
        <input
          type="file"
          accept="image/*"
          onChange={handleFileChange}
          disabled={disabled}
          className="hidden"
        />
      </label>
    </div>
  );
};

export default ImagePicker;
```

## 使用例

```tsx
// 画像未選択時
<ImagePicker onImageSelect={(base64, url) => {
  setImageBase64(base64);
  setImageUrl(url);
}} />

// 画像選択済み（差し替え用コンパクト表示）
<div className="relative">
  <img src={imageUrl} />
  <div className="absolute bottom-2 right-2">
    <ImagePicker onImageSelect={handleReplace} compact />
  </div>
</div>
```

## ポイント

- `capture="environment"`: モバイルでリアカメラを使用
- `e.target.value = ''`: 同じファイルの再選択を許可
- Base64とDataURL両方を返す（API送信用/表示用）
