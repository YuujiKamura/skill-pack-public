---
name: zig-winrt-com-binding-refactoring
description: "Zig COM/WinRTバインディングのシンボル削除・リファクタリング時の影響分析。(1) pub const再エクスポートの削除はグローバル破壊リスク、(2) Zigのlazy evaluationがビルドエラーを隠す、(3) critical-onlyレビュー戦略。Zig、COM、WinRT、binding、refactoring、ABI、symbol、vtableと言われた時に使用。"
---

# Zig COM/WinRT バインディング リファクタリング知見

**ドメイン**: Zig & COM

---

## 1. シンボル削除の影響分析

### pub const再エクスポートは高リスク
`com.zig` 等のブリッジファイルで `pub const IApplicationAbi = native.IApplicationAbi;` を削除すると、プロジェクト全体で `@ptrCast` やインターフェース初期化サイトが壊れる。ローカルでは不要に見えても、外部モジュールから唯一のエントリポイントとして使われている場合がある。

### Zigのlazy evaluationの罠
Zigはlazy compilationのため、ブリッジファイルの変更がそのファイル単体ではコンパイル通過する。しかし、実際にそのシンボルを使うパスが特殊化された時に初めて `undeclared identifier` エラーが出る。**シンボル削除前にgrep必須**。

```bash
grep -r "IApplicationAbi" src/
```

---

## 2. critical-onlyレビュー戦略

バインディングやブリッジコードのdiffレビューでは、命名規則やdocstring変更は無視し、以下の2点だけに集中する:

1. **シンボル連鎖の断絶**: public constantの削除がリンクやコンパイルに必要な「シンボルチェーン」を壊していないか
2. **ABI署名の変更**: extern structのフィールド順序やcallconv変更がないか

```
// レビュー制約:
// "Provide a concise, critical-only review (maximum 2 lines) of this git diff."
```

---

## 3. Connectivity Breakerの特定

ブリッジファイルのクリーンアップでは、高レベルwrapperと低レベルWin32/COMエントリポイントを接続するシンボルを「Connectivity Breaker」として重点チェックする。ローカルで冗長に見えても、インターフェース定義ライブラリとして外部モジュールが依存している可能性がある。
