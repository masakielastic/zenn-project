---
title: "第5章：Rust の enum とパターンマッチは Haskell にそっくり？"
free: true
---

## Rust の enum ってどういうもの？

**ずんだもん**「Rust の `enum` って、C言語の `enum` と同じなのだ？」

**めたん**「見た目は似てるけど、中に値を持てるのがポイントだよ。実質、Haskell の `data` 型と同じ！」


```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Say(String),
}
```

**つるぎ**「これで `Quit` は定数、`Move` は構造体的なもの、`Say` は文字列を持つバリアントだね。」

## Haskell での定義

```haskell
data Message = Quit
             | Move Int Int
             | Say String
```

**めたん**「Haskell も構造はほぼ同じ！データコンストラクタで複数の形をまとめてるの。」

## 値の使い方とパターンマッチ

**Rust**
```rust
fn show(msg: Message) {
    match msg {
        Message::Quit => println!("終了！"),
        Message::Move { x, y } => println!("移動: {}, {}", x, y),
        Message::Say(text) => println!("言う: {}", text),
    }
}
```

**Haskell**
```haskell
showMsg :: Message -> String
showMsg Quit = "終了！"
showMsg (Move x y) = "移動: " ++ show x ++ ", " ++ show y
showMsg (Say text) = "言う: " ++ text
```

**ずんだもん**「Rust の match は、Haskell のパターンマッチと一緒なのだ！」

## よく出る enum：Option / Result

**Rust** の `Option<T>`
```rust
let some = Some(42);
let none: Option<i32> = None;

match some {
    Some(n) => println!("数値: {}", n),
    None => println!("なかったよ"),
}
```

**Haskell** の `Maybe`
```haskell
Just 42
Nothing

case Just 42 of
  Just n  -> putStrLn ("数値: " ++ show n)
  Nothing -> putStrLn "なかったよ"
````

Rust の `Result<T, E>`

```rust
fn divide(x: i32, y: i32) -> Result<i32, String> {
    if y == 0 {
        Err("ゼロで割れません".to_string())
    } else {
        Ok(x / y)
    }
}
```

Haskell の `Either`

```haskell
divide x y = if y == 0 then Left "ゼロで割れません" else Right (x `div` y)
```

## if let / while let：簡易マッチ

**めたん**「全部のパターン書かなくていいときは `if let` が便利！」

```rust
if let Some(n) = some_value {
    println!("値あり: {}", n);
}
```

**つるぎ**「繰り返し処理で使う `while let` も強力！」

```rust
let mut iter = vec![1, 2, 3].into_iter();
while let Some(x) = iter.next() {
    println!("{}", x);
}
```

## まとめ

| 概念      | Rust                          | Haskell                      |
| ------- | ----------------------------- | ---------------------------- |
| 列挙型定義   | `enum Message { ... }`        | `data Message = ...`         |
| パターンマッチ | `match` 式                     | 関数定義や `case of`              |
| Option型 | `Option<T>` / `Some` / `None` | `Maybe` / `Just` / `Nothing` |
| Result型 | `Result<T, E>`                | `Either e a`                 |
| 簡易マッチ   | `if let`, `while let`         | `maybe`, `either`, ガードなど     |


**ずんだもん**「Rust って、型を使った安全設計がしっかりしてるのだ！」

**めたん**「Haskell の良さを受け継ぎつつ、現実的な使いやすさもあるのが Rust だよ。」

**つるぎ**「次は、ライフタイム（lifetime）や所有権の延長戦を見てみる？」
