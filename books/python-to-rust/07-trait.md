---
title: "第7章：Rust のトレイトってなに？型クラスや duck typing とどう違うの？"
free: true
---

## まずは Python の「なんでもアリ」な世界から

**ずんだもん**「Python では `+` を使えば、数字も文字列も連結できるのだ！」

```python
def double(x):
    return x + x

print(double(3))       # 6
print(double("Hi "))   # Hi Hi 
```

**めたん**「これは ダックタイピング（Duck Typing）って呼ばれる考え方。『`x` が `+` を持ってるなら使える』という動的型付けだね。」

## Haskell の型クラスは安全なインタフェース

```haskell
class Describable a where
    describe :: a -> String

instance Describable Int where
    describe x = "整数: " ++ show x

instance Describable Bool where
    describe True  = "真"
    describe False = "偽"
```

**つるぎ**「Haskell では `Describable` っていう**型クラス**（typeclass）で、その型が使える操作を制限してるの。」

## Rust の trait：型クラスにかなり近い！

```rust
trait Describable {
    fn describe(&self) -> String;
}

impl Describable for i32 {
    fn describe(&self) -> String {
        format!("整数: {}", self)
    }
}

impl Describable for bool {
    fn describe(&self) -> String {
        match self {
            true => "真".to_string(),
            false => "偽".to_string(),
        }
    }
}
```

**めたん**「Rust の `trait` は、Haskell の型クラスに構文も概念もかなり似てるんだよ！」

## ジェネリクスとトレイト境界

**Haskell**
```haskell
describeTwice :: Describable a => a -> String
describeTwice x = describe x ++ ", " ++ describe x
```

**Rust**
```rust
fn describe_twice<T: Describable>(x: T) -> String {
    format!("{}, {}", x.describe(), x.describe())
}
```

つるぎ「Haskell の `=>` は Rust の `<T: Trait>` に対応するね。」

## ジェネリクスの型制約なしとありの違い（Rust編）

```rust
fn double<T>(x: T) -> T {
    x + x  // コンパイルエラー！T に + が定義されていない
}
```

```rust
use std::ops::Add;

fn double<T: Add<Output = T>>(x: T) -> T {
    x + x  // OK！
}
```

**めたん**「`+` を使いたいなら `Add` トレイトが実装されてるって明示しなきゃダメなんだ。」

**ずんだもん**「Python だと自由だけど、Rust は慎重なのだ！」

## トレイトオブジェクト：動的ディスパッチも可能！

```rust
fn describe_all(things: Vec<Box<dyn Describable>>) {
    for thing in things {
        println!("{}", thing.describe());
    }
}
```

**つるぎ**「これは Python の duck typing に近い使い方。動的トレイトオブジェクトを使えば、型をまとめて扱えるよ。」

## まとめ

| 概念        | Python（duck typing） | Haskell（typeclass） | Rust（trait）                  |
| --------- | ------------------- | ------------------ | ---------------------------- |
| 動的/静的     | 動的（実行時チェック）         | 静的（コンパイル時保証）       | 静的（traitにより明示的に制約）           |
| 抽象インタフェース | 存在しない（構文的自由）        | class + instance   | trait + impl                 |
| 多態性の指定方法  | 特に制限なし              | `a =>` で型クラス指定     | `<T: Trait>` または `dyn Trait` |
| 使いやすさ     | 手軽で柔軟だが危険も多い        | 安全だが学習コストあり        | 安全で高速。制御がきめ細かい               |

**ずんだもん**「Rust の trait、思ったよりわかりやすいのだ！」

**めたん**「型の安全性と柔軟性を両立してるところが魅力なんだよ。」

**つるぎ**「次は、**非同期処理**（async/await）と並行処理（threads）を、Python や Haskell と比較してみようか？」
