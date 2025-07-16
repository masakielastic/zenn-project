---
title: "第8章：Rust の async / await と並行処理ってどうなってるの？"
free: true
---

## まずは Python の非同期処理から

**ずんだもん**「Python の `async` / `await` は使ったことあるのだ！」

```python
import asyncio

async def greet():
    await asyncio.sleep(1)
    print("Hello from async!")

asyncio.run(greet())
```

**めたん**「Python の `asyncio` はイベントループベース。`await` で中断・再開できる非同期関数を使えるよ。」


## Haskell は「非同期より並行」な設計

```haskell
import Control.Concurrent
import Control.Monad

main = do
  forkIO $ putStrLn "Hello from thread!"
  threadDelay 1000000
```

**つるぎ**「Haskell は**軽量スレッド**（forkIO）を使って並行処理を行うのが基本。非同期的ではあるけど、`await` は使わないね。」

## Rust の `async` / `.await` は型安全な非同期処理！

```rust
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    greet().await;
}

async fn greet() {
    sleep(Duration::from_secs(1)).await;
    println!("Hello from async!");
}
```

**めたん**「Rust の `async fn` は `Future` を返す関数で、`.await` を使ってその完了を待つよ。」

## 非同期 vs スレッド：どっちを使うの？

**Rust のスレッド例**

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        println!("Hello from thread!");
    });

    thread::sleep(Duration::from_secs(1));
}
```

**ずんだもん**「非同期とスレッドって何が違うのだ？」

**つるぎ**「非同期は1スレッドで複数タスクを切り替える軽量な手法。スレッドはOSレベルの並列で、より重たいけど CPU バウンド向き。」

## Rust の executor（Tokio / async-std）

**めたん**「Rust の非同期は `Future` ベースだけど、それを動かすために**executor**（実行器）が必要だよ。」

| ランタイム       | 特徴                     |
| ----------- | ---------------------- |
| `tokio`     | 高機能、高速、業界標準            |
| `async-std` | async/await に慣れた書き方、簡単 |
| `smol`      | 小規模用途、軽量               |

## Python / Haskell / Rust の並行モデル比較

| 言語      | 非同期API                 | スレッドAPI                         | 特徴                   |
| ------- | ---------------------- | ------------------------------- | -------------------- |
| Python  | `asyncio` / `await`    | `threading` / `multiprocessing` | シンプルだがGIL制限あり        |
| Haskell | `STM`, `forkIO`        | `Control.Concurrent`            | 軽量スレッド、純粋関数と親和性高     |
| Rust    | `async/.await` + Tokio | `std::thread::spawn`            | 静的型安全、ゼロコスト抽象、柔軟かつ高速 |

## ちょっと応用：並列で2つの処理を同時に待つ（Rust）

```rust
use tokio::join;

async fn task1() {
    println!("task1 start");
    sleep(Duration::from_millis(500)).await;
    println!("task1 end");
}

async fn task2() {
    println!("task2 start");
    sleep(Duration::from_millis(700)).await;
    println!("task2 end");
}

#[tokio::main]
async fn main() {
    join!(task1(), task2());  // 並行実行！
}
```

## まとめ

| モデル     | Python                 | Haskell       | Rust                     |
| ------- | ---------------------- | ------------- | ------------------------ |
| 非同期     | `async/await`          | 明示的APIなし      | `async fn` + `.await`    |
| 並行処理    | `threading`, `asyncio` | `forkIO`, STM | `std::thread`, `tokio`など |
| タスク実行   | 単純ループ                  | スケジューラ任せ      | executor 明示（Tokio等）      |
| エラーと型安全 | 例外ベース                  | 型安全           | コンパイル時にエラー検出             |

ずんだもん「Rust の `async`、ちょっと難しいけど強そうなのだ！」

めたん「スレッドと `async` をちゃんと使い分けできるのが Rust の魅力だよ。」

つるぎ「次はエラーハンドリングと `?` 演算子、`try` 式、パターンマッチでの `Result` 分解に進もうか？」
