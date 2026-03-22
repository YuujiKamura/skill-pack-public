---
description: WinUI3 unpackaged app の正常動作要件と勘所。(1) XamlControlsResources のPRI初期化要件、(2) TabView表示の必須条件、(3) IXamlMetadataProvider GetXamlType の正常応答パターン、(4) Application.LoadComponent と XBF の関係、(5) SwapChainPanel Loaded イベントの挙動。WinUI3、TabView、SwapChainPanel、XamlControlsResources、PRI、IXamlMetadataProvider、GetXamlType、BoxedType、ActivateInstance、COM vtable、com_aggregation、App.zig、surface_binding、tabview_runtime、ghostty-win、slot 6、slot 17、slot 19
---

# WinUI3 Unpackaged App 正常動作要件

## 核心知見: XamlControlsResources の初期化

### 問題
- `new XamlControlsResources()` をコードだけで呼ぶと `COMException: Cannot find a resource with the given key: AcrylicBackgroundFillColorDefaultBrush` で失敗する
- これは PRI (Package Resource Index) リソースシステムの初期化が不足しているため
- SelfContained モードでもこの問題は発生する（DLLは横にあっても PRI 検索パスが未登録）

### 解決策
- **App.xaml + XAML コンパイラが必須**: XAML コンパイラが生成する `App.g.i.cs` に PRI 初期化コードが含まれる
- App.xaml で `<XamlControlsResources>` を宣言すると、コンパイラが `ms-appx:///Microsoft.UI.Xaml/Themes/themeresources.xaml` を正しくロードするコードを生成
- XAMLコンパイラを使わない実装（Zig手書きCOM等）では、PRI初期化を自前で行う必要がある

### ghostty-win への影響
- ghostty-win は Zig + 手書き COM VTable で WinUI3 を実装しており、XAML コンパイラを使わない
- `IXamlMetadataProvider` の COM aggregation は実装済みだが、PRI リソース登録が不足している可能性
- MRT Core の `MrmCreateResourceManager` 等のネイティブ API で PRI を手動登録する必要があるかもしれない

## TabView 表示の必須条件

1. `XamlControlsResources` が `Application.Resources.MergedDictionaries` に追加されていること
2. Grid の Row 0 に TabView を配置（**Auto ではなく Pixel(40) を推奨** — 下記参照）
3. `ExtendsContentIntoTitleBar = true` + `SetTitleBar(tabView)` でタイトルバー統合
4. TabViewItem の Header と Content を設定
5. TabView に `HorizontalAlignment = Stretch`, `VerticalAlignment = Stretch` を設定

### Row 0 Auto vs Pixel 問題 (2026-03-07 確認)
- `GridUnitType.Auto` は TabView のコントロールテンプレートが完全適用されていれば動く（C# baseline で確認）
- **ghostty-win (Zig手書きCOM) では Auto だと高さ 0 に潰れる** — XamlControlsResources のテーマリソース（Acrylic ブラシ等）が完全には適用されないため
- **回避策**: `GridUnitType.Pixel` で固定 40px を指定してタブストリップの表示を保証
- 将来 PRI 初期化が完全になれば Auto に戻せる可能性あり

### Alignment enum の ABI (重要)
- `IFrameworkElement.put_HorizontalAlignment` / `put_VerticalAlignment` は **i32 (enum値)** を受け取る
- **誤り**: `?*anyopaque` (ポインタ) で定義すると、vtable 呼び出しは成功するが値が壊れる
- enum 定数: `HorizontalAlignment { Left=0, Center=1, Right=2, Stretch=3 }`
- enum 定数: `VerticalAlignment { Top=0, Center=1, Bottom=2, Stretch=3 }`

## ExtendsContentIntoTitleBar の影響

- `true`: タイトルバーが消え、コンテンツがタイトルバー領域まで拡張。SetTitleBar() でドラッグ領域指定
- `false`: 標準タイトルバーが表示される。TabView はタイトルバーの下に配置
- **注意**: ExtendsContentIntoTitleBar は **IWindow (v1)** のメンバー。IWindow2 ではない

### IWindow vs IWindow2 IID (2026-03-07 判明)
- **IWindow (v1)**: `{61F0EC79-5D52-56B5-86FB-40FA4AF288B0}` — ExtendsContentIntoTitleBar, SetTitleBar, Content, Activate, Close 等
- **IWindow2**: `{42FEBAA5-1C32-522A-A591-57618C6F665D}` — get_SystemBackdrop, put_SystemBackdrop, get_AppWindow のみ（3メソッド）
- ghostty-win の `native_interop.zig` で IWindow2 にIWindow v1 のIIDが設定されていた → QI成功するが実質IWindow v1 として使用していた（結果オーライだが混乱の元）
- `window` が既に `*com.IWindow` なら、`window.putExtendsContentIntoTitleBar(true)` で直接呼べる（QI不要）

## デプロイメントモードの違い

| モード | DLL位置 | PRI初期化 | Bootstrap必要 |
|--------|---------|-----------|---------------|
| SelfContained | exe横 | XAMLコンパイラ必須 | 不要 |
| Non-SelfContained + Bootstrap | システム | Bootstrap が初期化 | 必要 |
| Packaged (MSIX) | パッケージ内 | 自動 | 不要 |

## XAMLコンパイラが生成するコードの核心

```csharp
// App.g.i.cs — InitializeComponent() 内
global::System.Uri resourceLocator = new global::System.Uri("ms-appx:///App.xaml");
global::Microsoft.UI.Xaml.Application.LoadComponent(this, resourceLocator);
```

- `Application.LoadComponent(this, "ms-appx:///App.xaml")` が XBF (XAML Binary Format) をPRIからロード
- PRI に `ms-resource://WinUI3Baseline/Files/App.xbf` として登録されている
- XBF の中に `<XamlControlsResources>` 参照が含まれ、そこから themeresources.xaml がロードされる
- 生成された Main には `WinRT.ComWrappersSupport.InitializeComWrappers()` も含まれる

### ghostty-win での対応案
1. **案A**: PRI + XBF を自前生成して `IApplication::LoadComponent` COM呼び出し（最も正統）
2. **案B**: `ResourceDictionary` の `Source` に直接 `ms-appx:///Microsoft.UI.Xaml/Themes/themeresources.xaml` を設定
3. **案C**: MRT Core API (`MrmCreateResourceManager`) で PRI 検索パスを登録してから XamlControlsResources 相当を呼ぶ

## IXamlMetadataProvider 正常動作パターン (Level 5)

### GetXamlType 結果

| 型名 | 結果 | IsConstructible | 備考 |
|------|------|-----------------|------|
| TabView | OK | **True** | ActivateInstance で生成可能 |
| TabViewItem | OK | **True** | ActivateInstance で生成可能 |
| Grid | OK | **False** | 別ルート (RoActivateInstance) で生成 |
| Border | **null** | - | Provider が知らない基本XAML型 |
| SwapChainPanel | **null** | - | Provider が知らない基本XAML型 |
| TextBlock | **null** | - | Provider が知らない基本XAML型 |

### IXamlType プロパティ (TabView)
- `BoxedType = null` → **slot 17 が null は正常** (UWP との差異で slot ずれ注意)
- `ContentProperty = "TabItems"` → TabView のデフォルト content property
- `ActivateInstance` → TabView インスタンス生成成功 (slot 19)
- `GetXmlnsDefinitions: count=0` → **空配列が正常**

### ghostty-win への検証ポイント
- slot 6 (TypeName overload): TabView/TabViewItem/Grid を返す、それ以外は null → **正しい**
- slot 7 (HSTRING overload): 同じ結果 → **正しい**
- slot 17 (BoxedType): null → **正しい**
- slot 19 (ActivateInstance): TabView/TabViewItem を正常生成 → **正しい**
- IsConstructible=False の型 (Grid等) は ActivateInstance ではなく RoActivateInstance で生成

## SwapChainPanel の挙動 (Level 2-4)

### Background プロパティは使えない
- `SwapChainPanel.Background = ...` は `COMException: Setting 'Background' property is not supported on SwapChainPanel` で失敗
- D3D11/D3D12 SwapChain で直接描画する必要がある

### イベント発火順序
1. **SizeChanged が先、Loaded が後**
2. 初回: `SizeChanged(W x H)` → `Loaded(W x H)`
3. Clear+Append 後: `Loaded(ActualSize=0x0!)` → `SizeChanged(W x H)` — **再発火時の ActualSize は 0x0**

### Loaded 無限ループ (Level 4 で再現)
- **Loaded handler 内で `Children.Clear() + Children.Append()` を呼ぶと Loaded が即座に再発火する**
- C# でも同じ挙動。安全リミットがなければ**無限ループ**になる
- ghostty-win の `surface_binding.zig` `onLoaded` → `ensureVisibleSurfaceAttached` → `attachSurfaceToTabItem` → `children.clear() + children.append()` → Loaded 再発火 — **これが根本原因**
- **対策**: Loaded handler 内で reparent (Clear+Append) しない。または再入ガード (`isInLoadedHandler` フラグ) で防ぐ

### Tab切替時の SwapChainPanel 差し替え (Level 3)
- `SelectionChanged` で `contentGrid.Children.Clear() + Append(newPanel)` → 正常動作
- 差し替え後、新パネルの `SizeChanged` → `Loaded` が1回発火（ループなし）
- Loaded handler 内で再 Clear+Append しなければ安全

## ベースラインテストプロジェクト

- 場所: `C:\Users\yuuji\winui3-baseline`
- ビルド: VS 2022 MSBuild (`dotnet build` は MrtCore DLL 不足で失敗)
- レベル 0-5 の段階的テスト
- `capture2.ps1` でスクリーンショット自動取得

## PRI ファイル構成 (SelfContained)

- `Microsoft.UI.pri` — XAML フレームワークのリソース
- `Microsoft.UI.Xaml.Controls.pri` — コントロールテーマリソース
- `WinUI3Baseline.pri` — アプリ固有リソース
- これらが exe と同じディレクトリにあっても、MRT Core が検索パスを知らないとロードされない

## Resource Dictionary Composition Strategy

`put_Resources` で直接置き換えるのではなく、`MergedDictionaries` パターンを使う。

- 直接 `put_Resources` を呼ぶとスタイルの伝播失敗や初期化タイミング問題が起きる
- 正しい手順: (1) 新しい ResourceDictionary を作成 (2) ターゲットの `MergedDictionaries` IVector を取得 (3) `append` で追加
- 既存のフレームワークレベルリソースを維持しつつカスタムスタイルを重ねられる

```zig
// Instead of: control.put_Resources(new_res);
var merged = try control.get_MergedDictionaries();
try merged.append(new_res);
```

## XAML Metadata Resolution Delegation

`STATUS_STOWED_EXCEPTION (0xc000027b)` を防ぐための型解決委譲パターン。

- WinUI3 フレームワークは起動後 3-5 秒でXAML型 (TabView等) のテンプレートロード中に型解決に失敗すると stowed exception を投げる
- カスタム `IXamlMetadataProvider` 実装は、未知の型に対して null を返してはならない（内部/親プロバイダが存在する場合）
- Slot 6 (`GetXamlType`) を内部プロバイダに委譲しないと、組み込みコントロールの解決が壊れ、起動時致命的クラッシュになる

```zig
// In com_aggregation.zig: Ensure Slot 6 (GetXamlType) delegates
if (self.provider) |p| {
 return p.lpVtbl.GetXamlType(p, type_name_ptr, result_ptr);
}
```
