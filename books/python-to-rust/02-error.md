---
title: "第2章：型とエラー処理とパターンマッチ！"
free: true
---


## 【1】静的型 vs 動的型

**ずんだもん**「Python って型がないのが楽なのだ！」

**めたん**「でも Rust や Haskell は静的型言語だから、コンパイル時にミスが見つかるのが強みなんだよ。」

**Python**
```python
def double(x):
    return x * 2

print(double(3))      # 6
print(double("abc"))  # abcabc
```

**Haskell**
```haskell
double :: Int -> Int
double x = x * 2

-- double "abc" は型エラーになる！
```

**Rust（静的型）**
```rust
fn double(x: i32) -> i32 {
    x * 2
}

println!("{}", double(3));      // OK
// println!("{}", double("abc")); // コンパイルエラー
```

**つるぎ**「Rust は Haskell ほど型推論が強くないけど、その分わかりやすいね。」

## 【2】エラー処理：try / Maybe / Result


**ずんだもん**「Python では `try` でエラーをつかまえるのだ！」

**めたん**「Rust では `Result<T, E>` 型、Haskell では `Maybe` や `Either` を使うよ。」

**Python**
```python
try:
    x = int("abc")
except ValueError:
    x = 0
```

**Haskell**
```haskell
readInt :: String -> Maybe Int
readInt s = case reads s of
              [(x, "")] -> Just x
              _         -> Nothing
```

**Rust**
```rust
fn parse_number(s: &str) -> Result<i32, std::num::ParseIntError> {
    s.parse::<i32>()
}

match parse_number("abc") {
    Ok(n) => println!("成功: {}", n),
    Err(e) => println!("エラー: {}", e),
}
```

**つるぎ**「Rust の `Result` は成功と失敗の両方を明示的に扱えるから、安全性が高いんだよ。」


## 【3】パターンマッチの威力

**ずんだもん**「パターンマッチって if 文より便利なの？」

**めたん**「Rust も Haskell もデータ構造にマッチさせるのが超得意なんだ。」

**Python**
```python
match x:
    case 0:
        print("ゼロ")
    case 1:
        print("イチ")
    case _:
        print("その他")
```

（※ Python 3.10以降の match 文）

**Haskell**
```haskell
describe 0 = "ゼロ"
describe 1 = "イチ"
describe _ = "その他"
```

**Rust（静的型）**
```rust
fn describe(n: i32) -> &'static str {
    match n {
        0 => "ゼロ",
        1 => "イチ",
        _ => "その他",
    }
}
```

**ずんだもん**「`match` 文って見やすいのだ！」

**つるぎ**「さらに Rust では `Option<T>` や `Result<T, E>` に対してもマッチできるのが強力なんだよ。」

**【補足】`Option` のパターンマッチ（Rust限定）**

```rust
fn find_even(vec: Vec<i32>) -> Option<i32> {
    for x in vec {
        if x % 2 == 0 {
            return Some(x);
        }
    }
    None
}

match find_even(vec![1, 3, 4, 7]) {
    Some(n) => println!("最初の偶数: {}", n),
    None => println!("偶数がなかったよ"),
}
```


### 【まとめ】

**めたん**「まとめると…」

|テーマ|Python|Haskell        |Rust|
|-----|-------|-------------  |----|
|型   |動的型 |静的型 + 型推論|	静的型 + 明示的な型注釈|
|エラー処理|try/except	Maybe / Either|	Result<T, E>|
|パターンマッチ|3.10以降のmatch文|関数定義で使える|強力な match 式|


**ずんだもん**「Rust って、Haskell のきっちり感と Python の手軽さをいいとこ取りしてるのだ！」

**めたん**「その分、最初はちょっとむずかしいけど慣れれば楽しいよ。」

**つるぎ**「次は所有権と借用を Python や Haskell の感覚とどう違うか比較してみようか？」

