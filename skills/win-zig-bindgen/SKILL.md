---
name: win-zig-bindgen
description: "win-zig-bindgen (WinMD→Zig COM bindingジェネレータ) の設計知見。(1) 統合マップによる型解決、(2) vtable形状検証、(3) 本家windows-rsパリティ、(4) テスト設計。win-zig-bindgen、ジェネレータ、WinMD、bindgen、コード生成、型解決、パリティと言われた時に使用。"
---

# win-zig-bindgen 開発知見

**文脈**: WinMDメタデータからZig用COM vtableバインディングを自動生成するツール。本家 windows-rs と同品質の生成物を目標とする。

リポジトリ: `~/win-zig-bindgen` (private)

---

## 1. 型解決: 統合マップ方式

### 問題
main/companion二段階探索は、複数WinMDにまたがる深い依存チェーンで破綻する。

### 解決
全WinMDを起動時に読み込み、`Namespace.TypeName → WinmdSource` の統合HashMapを構築。ファイル境界に依存しない型解決を実現。

```zig
// unified_index.zig
const TypeMap = AutoHashMap(TypeName, WinmdSource);
// IAsyncAction がどのWinMDにあっても解決可能
```

### 原則
- windows-rs の構造を参考にする。独自アプローチを増やさない
- 規模を数値化してから設計判断する（「本家は巨大」で済ませない）

---

## 2. vtable形状検証

### 粗い検証では不十分
`?*anyopaque` のカウントだけでは微妙なslotズレを検出できない。

### 正しい検証方法
1. WinMDの **MethodDefテーブル** から期待されるvtable形状を導出
2. 生成されたZigコードと **slot単位** で比較（hybrid comparator）

```zig
fn extractWinmdShape(method_def_table: Table) VTableLayout { ... }
fn runExactShapeProbe(generated_type: type, expected: VTableLayout) void { ... }
```

### vtable slotズレの診断
- `ACCESS_VIOLATION at 0x0` → vtable index shiftを疑え。missing/extra/reorderedなslotが1つあるだけで後続全メソッドがズレる
- 手書きABI定義で `put_Resources` 等のメソッドが抜けていないか、WinMDメタデータと突合せろ
- 特定メソッドのslot番号をアンカーにして、手書きと自動生成の差分を検証

### 基底スロットのフィルタリング
- IUnknown (QueryInterface/AddRef/Release) = +3スロット
- IInspectable = +3スロット（計+6 or +7）
- これを除外しないと全インターフェースで誤検出が大量発生
- windows-rsもusizeプレースホルダを多用（IUIElementで125個）。差分（~147個）が本当の改善対象

---

## 3. 本家パリティテスト

### mock vs parity
- **mock**: 型が存在するか確認するだけ → 不十分
- **parity**: 本家windows-rsと同一出力か厳密比較 → 必要

### Golden File方式
```zig
for (cases) |case| {
    const generated = try bindgen.generate(case);
    try expectEqualStrings(golden_files.get(case), generated);
}
```

### テストディレクトリ構造（upstream準拠）
```
tests/metadata/    (WinMDリーダーのパリティ)
tests/bindgen/runtime/  (ランタイムサポート)
tests/bindgen/winui/    (exact-shape probes)
```

- ローカルのissue番号やファイル境界でテストを分割するな
- upstreamの責務分割を唯一の正とする

---

## 4. テストインフラの完成基準

- **「テストが通る」ではなく「正しく失敗する」が完成**
- 具体的なエラーログ（missing vtable slot等）を出せる状態 = インフラ完成
- バグ修正はインフラ完成後に別タスクとして着手
- 手動COM定義が残っている場合は徹底削除。残骸がテストを汚染する

---

## 5. 注意事項

- 本家と同じアプローチでも統合マップの品質が違えば意味がない → 数値比較必須
- 「細かいことでも本家がどうやってるか確認しろ」（ユーザー指示）
- フェーズごとにissueを立てて管理する
