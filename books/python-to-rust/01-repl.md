---
title: "第1章：REPL入門"
free: true
---

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

**つむぎ**「ここで、3つの言語の REPL を比べてみましょう。」

|言語   |REPL名|起動方法|特徴|
| ----- | ---- | ------ | -- |
|Python |REPL  |python3	|シンプルで定番、初心者向け|
|Haskell|GHCi  |ghci	|型が強く、:t で型の確認ができる|
|Rust	|evcxr |evcxr	|型付き + クレート追加可能 :dep も便利|

**ずんだもん**「3つを同時に試したいときはどうすればいいの？」

**めたん**「Chromebook の Linux ターミナルならタブを使えば簡単だよ。タブを開くには Ctrl + Shift + T。」

**つむぎ**「もしくは tmux を入れれば画面分割もできるよ。」

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

**つむぎ**「まとめると…」

Rust で REPL を使うなら evcxr

 * Python や Haskell との比較もタブでできる
 * Chromebook でも問題なく動作する
 * クレート追加など Rust 特有の機能も学べる

**ずんだもん**「Rust の勉強がはかどりそうなのだ！PythonやHaskellでやった課題も試せるのだ！」

**めたん**「それなら Rust に書き換えていくチュートリアルも作ってみようか？」

**つむぎ**「言語横断で学べるって、強いよね。」

