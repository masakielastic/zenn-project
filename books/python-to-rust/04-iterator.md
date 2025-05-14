---
title: "第4章：イテレータって関数型？Rust で map・filter・collect！"
free: true
---

## 【1】Python の map/filter のおさらい

**ずんだもん**「Python で `map` や `filter` はよく使ったのだ！」

```python
nums = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x * x, nums))
evens = list(filter(lambda x: x % 2 == 0, nums))
```

**めたん**「おなじみのコードだね。でも `list()` で囲まないと結果が見えない点は注意だよ。」

## Haskell の map/filter は関数型の基本！


```haskell
nums = [1, 2, 3, 4, 5]
squares = map (^2) nums
evens = filter even nums
```

**つるぎ**「Haskell ではこれが自然なスタイル。リスト内包表記でも書けるよ。」

```haskell
squares = [x^2 | x <- nums]
```

## Rust のイテレータは“変換チェーン”が強力！

**めたん**「Rust は `iter()` → `.map()` → `.filter()` → `.collect()` の流れが基本だよ。」

```rust
fn main() {
    let nums = vec![1, 2, 3, 4, 5];

    let squares: Vec<i32> = nums.iter().map(|x| x * x).collect();
    let evens: Vec<i32> = nums.iter().filter(|x| *x % 2 == 0).cloned().collect();

    println!("squares: {:?}", squares); // [1, 4, 9, 16, 25]
    println!("evens: {:?}", evens);     // [2, 4]
}
```

**ずんだもん**「`collect()` をつけると `Vec` に戻るのだね！」

## iter, into_iter, iter_mut の違い（Rust 特有）

**つるぎ**「Rust にはイテレータが3種類あるのもポイントだよ。」

| メソッド           | 値の種類     | 使用用途            |
| -------------- | -------- | --------------- |
| `.iter()`      | **参照**   | 読み取り専用          |
| `.iter_mut()`  | **可変参照** | 値を変更したいとき       |
| `.into_iter()` | **値ごと**  | 所有権を消費して処理したいとき |


```rust
let mut v = vec![1, 2, 3];
for x in v.iter_mut() {
    *x *= 10;
}
println!("{:?}", v); // [10, 20, 30]
```

## クロージャと map の比較

**Python**
```python
map(lambda x: x * 2, [1, 2, 3])
```

**Haskell**
```haskell
[1, 2, 3].iter().map(|x| x * 2)
```

**Rust**
```rust
map (*2) [1,2,3]
```

**めたん**「Rust の `|x| x * 2` はクロージャ。Haskell の `(*2)` に似てるよ。」

## collect の型注釈は必須？

**ずんだもん**「`.collect()` のあとに型を書くの、忘れやすいのだ…」

**つるぎ**「型推論が効かない場合、Rust では明示が必要だよ。」

```rust
let doubled: Vec<i32> = nums.iter().map(|x| x * 2).collect();
```

**めたん**「型を書かずに済ませたいときは `let doubled = nums.iter().map(|x| x * 2).collect::<Vec<_>>();` でもOK！」

## まとめ

| 処理      | Python                        | Haskell     | Rust                          |
| ------- | ----------------------------- | ----------- | ----------------------------- |
| map     | `map(lambda x: ..., list)`    | `map` 関数    | `.iter().map(...)`            |
| filter  | `filter(lambda x: ..., list)` | `filter` 関数 | `.iter().filter(...)`         |
| collect | `list(...)`                   | 組込み         | `.collect::<Vec<_>>()`        |
| 可変操作    | ミュータブル                        | 不変          | `.iter_mut()` で変更             |
| 所有権管理   | 暗黙                            | 不変          | `.iter()`, `.into_iter()` で制御 |


## 次回予告

**ずんだもん**「Rust の `map` とか、Haskell に似てて安心したのだ！」

**めたん**「次は、`match` 式と `enum` のパターンマッチに進んでみようか。Haskell の `data` 型と近いんだよ。」

**つるぎ**「`Option<T>` や `Result<T, E>` も `enum` の応用だしね！」
