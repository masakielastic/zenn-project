---
title: "第1章：REPL入門"
free: true
---

## REPL

**ずんだもん**「Rust を勉強してるのだ！でも Python や Haskell みたいに REPL ってあるの？」

**めたん**「あるよ、evcxr っていう対話環境があって、Rust でもすぐコードを試せるの。」

**ずんだもん**「やったー！Chromebook の Linux 環境でも使えるの？」

**めたん**「もちろん！次のコマンドを入力すればOK。」

```bash
sudo apt install libclang-dev
cargo install evcxr_repl
```

**ずんだもん**「これで evcxr が使えるのだ！」

**めたん**「そう。起動したらこんな感じで使えるよ。」

```bash
>> let x = 2 + 3;
>> println!("x = {}", x);
```

**ずんだもん**「おお！Python っぽい！」

**【REPL の比較】Python / Haskell / Rust**

**つるぎ**「ここで、3つの言語の REPL を比べてみましょう。」

|言語   |REPL名|起動方法|特徴|
| ----- | ---- | ------ | -- |
|Python |REPL  |python3	|シンプルで定番、初心者向け|
|Haskell|GHCi  |ghci	|型が強く、:t で型の確認ができる|
|Rust	|evcxr |evcxr	|型付き + クレート追加可能 :dep も便利|

**ずんだもん**「3つを同時に試したいときはどうすればいいの？」

**めたん**「Chromebook の Linux ターミナルならタブを使えば簡単だよ。タブを開くには Ctrl + Shift + T。」

**つるぎ**「もしくは tmux を入れれば画面分割もできるよ。」

```
sudo apt install tmux
tmux
```

**【補足：evcxr の便利コマンド】**

**めたん**「Rust ならではの機能も紹介しておくね！」

```rust
>> :dep rand = "0.8"   // クレートの追加
>> use rand::Rng;
>> rand::thread_rng().gen_range(1..=6)
```

**ずんだもん**「サイコロが振れたのだ！」

**まとめ**

**つるぎ**「まとめると…」

Rust で REPL を使うなら evcxr

 * Python や Haskell との比較もタブでできる
 * Chromebook でも問題なく動作する
 * クレート追加など Rust 特有の機能も学べる

**ずんだもん**「Rust の勉強がはかどりそうなのだ！PythonやHaskellでやった課題も試せるのだ！」

**めたん**「それなら Rust に書き換えていくチュートリアルも作ってみようか？」

**つるぎ**「言語横断で学べるって、強いよね。」


## Python・Haskell でやった課題を Rust に書き換えてみよう！

### 課題1：フィボナッチ数列（再帰）

**ずんだもん**「ぼくが昔やった課題の中で、一番印象に残ってるのはフィボナッチ数列なのだ！」

**めたん**「Python と Haskell の例、それに Rust での実装を比べてみよう。」


**Python（再帰）**

```python
def fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)
```

**Haskell（再帰）**

```haskell
fib n
  | n <= 1    = n
  | otherwise = fib (n - 1) + fib (n - 2)
```

**Rust（再帰）**

```rust
fn fib(n: u32) -> u32 {
    if n <= 1 {
        n
    } else {
        fib(n - 1) + fib(n - 2)
    }
}
```

**つるぎ**「Rust は型が厳しいから、u32 と型注釈が必須なのがポイントだね。」

**ずんだもん**「なるほどなのだ！」


### 課題2：FizzBuzz

**ずんだもん**「次は FizzBuzz！ 3でFizz、5でBuzz、15でFizzBuzzなのだ！」

**Python**
```python
for i in range(1, 21):
    if i % 15 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```

**Haskell**
```haskell
mapM_ putStrLn [ if i `mod` 15 == 0 then "FizzBuzz"
                 else if i `mod` 3 == 0 then "Fizz"
                 else if i `mod` 5 == 0 then "Buzz"
                 else show i
               | i <- [1..20] ]
```


**Rust**

```rust
for i in 1..=20 {
    if i % 15 == 0 {
        println!("FizzBuzz");
    } else if i % 3 == 0 {
        println!("Fizz");
    } else if i % 5 == 0 {
        println!("Buzz");
    } else {
        println!("{}", i);
    }
}
```

**めたん**「Rust のループは 1..=20 で for、あとは println! を使うのが基本だね。」


### 課題3：階乗（再帰）

**Python**
```python
def fact(n):
    return 1 if n == 0 else n * fact(n - 1)
```

**Haskell**
```haskell
fact 0 = 1
fact n = n * fact (n - 1)
```

**Rust**
```rust
fn fact(n: u32) -> u32 {
    if n == 0 {
        1
    } else {
        n * fact(n - 1)
    }
}
```

**ずんだもん**「Rust って思ったより直感的なのだ！」

## まとめ：Rust は型と所有権がキモ！

**つるぎ**「Rust は型注釈と所有権があるから、Haskell の厳密さと Python のわかりやすさの中間的な書き方になるの。」

**めたん**「次は Option<T> や Result<T, E> みたいな型を使ってエラー処理を Python の try / Haskell の Maybe と比較してみるのも面白いかも。」

**ずんだもん**「それ、絶対やりたいのだ！」

