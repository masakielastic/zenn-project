---
title: "第6章：Rust のライフタイムってなに？参照を安全に使いたい！"
free: true
---

## PythonとHaskellには“ライフタイム”がない？

**ずんだもん**「Rust のライフタイムってよく聞くけど、Python や Haskell にはないのだ？」

**めたん**「そう。GC（ガーベジコレクション）やイミュータブルな構造に頼ってるから、明示的な“生存期間”は指定しないんだよ。」

**Python**
```python
def get_first_word(s):
    return s.split()[0]

print(get_first_word("hello world"))
```

**Haskell**
```haskell
getFirstWord :: String -> String
getFirstWord s = head (words s)
```

**つるぎ**「でも Rust では `&str` や `&T` を構造体に持たせるときに、誰がどれだけ生きてるかをはっきりさせる必要があるんだよ。」

## Rust のライフタイム付き参照

```rust
fn longest<'a>(a: &'a str, b: &'a str) -> &'a str {
    if a.len() > b.len() { a } else { b }
}
```

**ずんだもん**「`<'a>` ってなに！？ 変な記号なのだ！」

**めたん**「これはライフタイム注釈。`a` も `b` も、同じくらい長く生きてる参照じゃないと返しちゃダメってことを明示してるんだよ。」

## 構造体に参照を含める場合

```rust
struct Highlight<'a> {
    part: &'a str,
}

fn main() {
    let text = String::from("Rust is great!");
    let h = Highlight { part: &text[0..4] };
    println!("{}", h.part);  // Rust
}
```

つるぎ「この `'a` は、text が生きているあいだだけ Highlight が有効だと保証してるの。」

## Haskell の構造体はどう？

```haskell
data Highlight = Highlight { part :: String }

main = let h = Highlight { part = take 4 "Rust is great!" }
       in putStrLn (part h)
```

**めたん**「Haskell では文字列がコピーされるし、GC もあるから、ライフタイム管理は必要ないんだ。」

## ライフタイムが必要な場面 vs 不要な場面

| パターン        | Rust（ライフタイム必要）                 | Python / Haskell（暗黙で安全） |
| ----------- | ------------------------------ | ----------------------- |
| 参照を返す関数     | `&'a str` など明示                 | 値のコピーまたは GCで管理          |
| 構造体に参照を含む   | `struct S<'a> { part: &'a T }` | 構造体自体にコピー保持             |
| 短命な一時オブジェクト | 借用チェッカで拒否されることも                | 問題にならないことが多い            |


## ライフタイムの推論（初心者救済機能）

**ずんだもん**「Rust って難しそうだけど、毎回 'a とか書かないといけないの？」

**めたん**「いい質問！関数の引数が `&` だけなら、Rust コンパイラが自動でライフタイムを推論してくれるんだよ。」

```rust
fn get_len(s: &str) -> usize {
    s.len()
}  // ライフタイム注釈なしでOK！
```

## まとめ：ライフタイムは所有権の延長線！

| 特徴        | Rust                | Python / Haskell |
| --------- | ------------------- | ---------------- |
| 明示的な管理    | `<'a>` を使って参照の寿命を記述 | ガーベジコレクションに依存    |
| 構造体での注意点  | 参照を持たせるならライフタイム必須   | 値をコピーする設計が多い     |
| 学習の難所だけど… | 所有権・借用の理解があればすぐ慣れる  | 基本的に意識しなくても問題ない  |

**ずんだもん**「なるほど…！Rust のライフタイムって、参照の安全な取り扱いのためなのだ！」

**つるぎ**「怖がる必要はないよ。借用チェックが自動でミスを防いでくれるから、慣れれば味方になる。」

**めたん**「次は、**トレイト**（trait）と型クラス、**ジェネリクス**（generics）の話にしようか。」
