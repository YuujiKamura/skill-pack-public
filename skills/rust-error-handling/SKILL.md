---
description: RustのResult型によるエラーハンドリング統一パターン。(1) プロジェクト共通のResult型定義、(2) ErrorKindによる分類、(3) thiserrorでのエラー定義、(4) ?演算子での伝播、(5) unwrap/panic禁止。Rust、エラーハンドリング、Result、Error、thiserrorと言われた時に使用。
keywords: [Rust, エラーハンドリング, Result, Error, thiserror, anyhow, panic]
---

# Rust エラーハンドリング統一パターン

プロジェクト全体でResult型を統一し、エラー処理を一貫させる。

## 基本構造

```rust
// src/error.rs

use thiserror::Error;

/// プロジェクト共通のエラー型
#[derive(Error, Debug)]
pub enum Error {
    #[error("解析エラー: {0}")]
    Parse(String),

    #[error("検証エラー: {0}")]
    Validation(String),

    #[error("IO エラー: {0}")]
    Io(#[from] std::io::Error),

    #[error("ネットワークエラー: {0}")]
    Network(String),

    #[error("予期しないエラー: {0}")]
    Other(String),
}

/// プロジェクト共通のResult型
pub type Result<T> = std::result::Result<T, Error>;
```

## 使用例

```rust
use crate::error::{Error, Result};

pub fn parse_config(path: &str) -> Result<Config> {
    let content = std::fs::read_to_string(path)?;  // Io エラーに自動変換

    let config: Config = serde_json::from_str(&content)
        .map_err(|e| Error::Parse(e.to_string()))?;

    if config.name.is_empty() {
        return Err(Error::Validation("name は必須です".into()));
    }

    Ok(config)
}
```

## エラー分類の指針

```rust
pub enum ErrorKind {
    /// 入力データの解析に失敗
    Parse,

    /// ビジネスルール違反
    Validation,

    /// ファイル/DB操作の失敗
    Io,

    /// 外部API/ネットワーク通信の失敗
    Network,

    /// 上記に該当しない
    Other,
}
```

| 種類 | 例 |
|------|-----|
| Parse | JSON/XML/CSVのパース失敗、フォーマット不正 |
| Validation | 必須項目なし、範囲外の値、状態不正 |
| Io | ファイル読み書き、DB接続、権限エラー |
| Network | タイムアウト、接続失敗、HTTPエラー |
| Other | 予期しない状態、外部ライブラリのエラー |

## Non-negotiables

### 禁止: unwrap() / expect() / panic!()

```rust
// ❌ 禁止（本番コード）
let value = some_option.unwrap();
let result = some_result.expect("should not fail");
panic!("unexpected state");

// ✅ 許可（テストコード）
#[cfg(test)]
mod tests {
    #[test]
    fn test_something() {
        let value = some_option.unwrap();  // テストでは OK
    }
}
```

### 必須: ? 演算子でのエラー伝播

```rust
// ❌ 冗長
fn process() -> Result<Data> {
    let file = match std::fs::read_to_string("data.txt") {
        Ok(f) => f,
        Err(e) => return Err(Error::Io(e)),
    };
    // ...
}

// ✅ 簡潔
fn process() -> Result<Data> {
    let file = std::fs::read_to_string("data.txt")?;  // 自動変換
    // ...
}
```

## エラーコンテキストの追加

```rust
use anyhow::{Context, Result};

fn load_user(id: u64) -> Result<User> {
    let path = format!("users/{}.json", id);
    let content = std::fs::read_to_string(&path)
        .with_context(|| format!("ユーザー {} の読み込みに失敗", id))?;

    let user: User = serde_json::from_str(&content)
        .with_context(|| format!("ユーザー {} のJSON解析に失敗", id))?;

    Ok(user)
}

// エラーメッセージ例:
// Error: ユーザー 123 の読み込みに失敗
// Caused by: No such file or directory (os error 2)
```

## thiserror vs anyhow

| ライブラリ | 用途 |
|-----------|------|
| thiserror | ライブラリ用。型付きエラーを定義 |
| anyhow | アプリケーション用。エラーチェーンを簡単に |

```toml
# Cargo.toml
[dependencies]
thiserror = "1.0"  # エラー型定義
anyhow = "1.0"     # アプリケーションレベル
```

## エラーログの統一

```rust
impl Error {
    pub fn log(&self) {
        match self {
            Error::Parse(msg) => tracing::warn!("Parse error: {}", msg),
            Error::Validation(msg) => tracing::info!("Validation error: {}", msg),
            Error::Io(e) => tracing::error!("IO error: {}", e),
            Error::Network(msg) => tracing::error!("Network error: {}", msg),
            Error::Other(msg) => tracing::error!("Unexpected error: {}", msg),
        }
    }
}
```

## ディレクトリ構成

```
src/
├── error.rs        # エラー型定義
├── lib.rs          # pub mod error;
└── main.rs         # use crate::error::Result;
```

## 参考

- 元記事: https://qiita.com/hisaho/items/5bd0096ca33d5ea41acb
- thiserror: https://docs.rs/thiserror
- anyhow: https://docs.rs/anyhow
