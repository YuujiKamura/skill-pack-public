# windows-rs First Principle

## When to use
win-zig-bindgen のバグ修正・機能追加・リファクタリング時、実装を書く前に必ず使う。

## Principle
**win-zig-bindgen で何か修正・実装する前に、必ず windows-rs (本家) がどう実装しているか確認しろ。**

1. まず windows-rs の該当コード (crates/libs/bindgen/) を調べる
2. 本家の設計意図・データフロー・エッジケース処理を理解する
3. 本家と同じ方式で実装する。独自方式を発明するな
4. 実装後、本家との差異を列挙して自己検証する

## Why
- win-zig-bindgen は windows-rs の Zig 移植。本家が正解
- 独自判断で修正すると、本家が既に解決済みのエッジケースを踏む
- "動けばいい" 修正は技術的負債。本家と同じにすれば将来も互換

## Reference locations
- windows-rs repo: `crates/libs/bindgen/src/` (Rust)
  - `types.rs` — 型解決、required interfaces
  - `winmd/mod.rs` — WinMD reader
  - `winmd/index.rs` — TypeIndex (二段HashMap)
  - `structs.rs`, `interfaces.rs`, `delegates.rs`, `enums.rs` — emit
- win-zig-bindgen repo: `C:\Users\yuuji\win-zig-bindgen\`
  - `unified_index.zig` — TypeIndex の Zig 移植
  - `emit.zig` — emit ロジック
  - `sig_decode.zig` — 型シグネチャ解析
  - `dependency.zig` — 依存追跡

## Checklist (every fix)
- [ ] windows-rs で同等の処理を特定したか？
- [ ] 本家のロジックを理解したか？
- [ ] 本家と同じ方式で実装したか？
- [ ] 実装後、本家との差異がないか確認したか？
