---
name: ghostty-winui3-apprt
description: "Ghostty WinUI3アプリランタイム開発の知見集。(1) IME入力の強制活性化、(2) マルチタブIPC設計、(3) クラッシュ調査手順、(4) WT準拠の初期化契約、(5) キャプションボタン・スクロール。ghostty、WinUI3、winui3、Zig、COM、vtable、IME、タブ、IPC、クラッシュと言われた時に使用。"
---

# Ghostty WinUI3 App Runtime 開発知見

**文脈**: GhosttyターミナルエミュレータをC++/WinRTもXAMLコンパイラも使わずに、ZigからWinUI3 APIを手書きCOM vtableで直接叩いて動かすプロジェクト。

リポジトリ: `~/ghostty-win` (branch: winui3-apprt)

---

## 1. IME日本語入力

### 問題の判定
- `WM_KEYDOWN vk=0x41` + `WM_CHAR` が来る = IMEバイパス状態（raw input）
- `WM_KEYDOWN vk=0xE5`（VK_PROCESSKEY）が来る = IME正常動作
- `VK_IME_ON`(0xF3) がサブクラス化された親/兄弟HWNDに食われていないかトレース

### 修正パターン
フォーカス遷移時に `ImmSetOpenStatus` を明示呼出して強制Open:
```zig
const himc = ImmGetContext(input_hwnd);
if (himc != null) {
    _ = ImmSetOpenStatus(himc, 1);
    _ = ImmReleaseContext(input_hwnd, himc);
}
```

### 注意
- WinUI3のサブクラス化コンテナが `VK_IME_ON` を消費してinput overlayに届かない問題がある
- フレームワーク内部のIME状態マシンとの競合（race condition）に注意
- 手動 `ImmSetOpenStatus` がフレームワークの即時リセットと衝突しないか検証が必要

---

## 2. マルチタブIPC (Control Plane)

Win32版の単一ウィンドウControl PlaneをWinUI3のマルチタブ環境に移植する設計。

### 設計原則
- IPC入力はアクティブタブ（`app.activeSurface()`）に動的ルーティング。メインウィンドウ送りにしない
- Named Pipeにセッション+PIDを含める: `\\.\pipe\ghostty-winui3-{session}-{pid}`
- Win32版からの安易な共通化より、コード複製を許容して挙動の独立性を確保

### WM定数
```zig
const WM_APP_CONTROL_INPUT = WM_USER + 4;
```

---

## 3. クラッシュ調査手順

1. 「クラッシュした」だけの報告 → **推測でコードを触るな。直近ログを最優先で確認**
2. 起動直後クラッシュ → 最新ログが唯一の事実源。「どの初期化段階で落ちたか」を特定
3. fileLogが空 → デバッグビルドで確認。リリースビルドでは出ないログがある
4. `OutputDebugStringW` でfileLogに依存しない経路でも確認可能

### Wrapper置き換えコミットの危険性
- raw vtable call (`self.lpVtbl.Release(self)`) をwrapper (`self.release()`) に置き換える「hygiene」コミットは、ref-countingバグの主犯になりやすい
- クラッシュ発生時は、直近のhygieneコミットをrevert対象として最優先で疑え
- 投機的修正の前に、シンボル解決とフォレンジクスツール（ghostty-win-crash-triage等）のデプロイを先にやれ

### COM Delegateの寿命問題
- 起動後5-20秒で発生するクラッシュ → 非同期イベントハンドラ（TypedEventHandlerImpl等）のref-counting異常を疑え
- `Release` は必ずatomic操作を使い、ref_count == 0 の時だけ破棄すること
- 自動生成されたdelegateの`this`ポインタ所有権モデルが、手書き版の期待と一致しているか検証必須
```zig
// 正しいCOM Releaseパターン
fn Release(self: *Self) callconv(WINAPI) u32 {
    const count = @atomicRmw(u32, &self.ref_count, .Sub, 1, .seq_cst) - 1;
    if (count == 0) {
        self.allocator.destroy(self);
    }
    return count;
}
```

---

## 4. Windows Terminal準拠の原則

- **「準拠」= 見た目の模倣ではなく、初期化挙動の一致**
  - タブ初期化・アクティブ化・起動直後の状態遷移を合わせる
  - UI模倣だけだと起動時クラッシュに直結する
- **起動経路に触る変更は、機能完成前でも実行確認を優先**
  - 静的確認だけでは不十分。「起動できるか」を先に確認
  - 起動失敗したら機能改修を続行せず、即クラッシュ原因の切り分けにピボット
- キャプションボタン（最小化/最大化/閉じる）が消える問題 → WT同等の構造を忠実に再現

---

## 5. ビルド・実行

- WinUI3: `./build-winui3.sh`（ラッパーが `--prefix zig-out-winui3` を強制）
- Win32: `zig build -Dapp-runtime=win32 --prefix zig-out-win32`
- **`zig build` を直接叩くな。必ずラッパーか `--prefix` を使え**
- デバッグビルドでClaude Codeなど重い表示をするとフリーズする（既知）
