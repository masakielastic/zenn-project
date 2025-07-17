---
title: "文字とコードポイントを相互変換するコマンドを作成する"
emoji: "🔁"
type: "tech"
topics: ["bash", "unicode", "python", "rust"]
published: false
---

文字とコードポイントを相互変換する処理はシェル芸のよくある題材です。
テストケースでこれらの相互変換をよく使うので Bash、Python、Rust でコマンドとして作成してみました。

Bash での実装
-------------

`ord` と `chr` ファイルを用意して次のコードを記載します。

```sh:ord
input="$1"
echo -n "$input" \
  | grep -oP . \
  | while IFS= read -r char; do
      printf '%X ' "'$char"
    done
echo
```


```sh:chr
for cp in "$@"; do
  printf '%b' "\\U$(printf '%08X' 0x$cp)"
done
printf '\n'
```

ord では grep が使われていますが、UTF-8 ロケールであればインデックスアクセスが利用できます。

```
$ export LC_CTYPE=en_US.UTF-8

$ str="あいうえお"
$ echo "${#str}"
5     # ← 文字数としてカウント！

$ echo "${str:0:1}"
あ    # ← 正しく1文字を取り出している
```

実行権限を付与します。

```
chmod +x ord
chmod +x chr
```

ord と chr を PATH が通っている場所にインストールします。

基本的な機能が使えるか確認します。

```
> ord あ
3042 
```

```
> chr 3042
あ
```

複数の文字やコードポイントにも対応しています。

```
> ord あいうえお
3042 3044 3046 3048 304A 
```

```
> chr 3042 3044 3046 3048 304A
あいうえお
```

Python による実装
-----------------

移植性を考慮して Python による実装を示します。

```python:ord
#!/usr/bin/env python3

import sys

text = sys.argv[1] if len(sys.argv) > 1 else ""
for char in text:
    print(f"{ord(char):X}", end=" ")
print()
```


```python:chr
#!/usr/bin/env python3
import sys

def cps_to_chars(codepoints):
    chars = []
    for cp in codepoints:
        cp = cp.upper().lstrip("U+")
        try:
            chars.append(chr(int(cp, 16)))
        except ValueError:
            print(f"無効なコードポイント: {cp}", file=sys.stderr)
            sys.exit(1)
    return ''.join(chars)

if __name__ == "__main__":
    if sys.stdin.isatty():
        cps = sys.argv[1:]
    else:
        cps = sys.stdin.read().strip().split()

    print(cps_to_chars(cps))
```

Rust による実装
---------------

1つのプロジェクトで2つのコマンドツールを作成してみましょう。

charutil というフォルダーを作成します。

```toml:Cargo.toml
[package]
name = "charutil"
version = "0.1.0"
edition = "2021"

[dependencies]
```

```rust:src/bin/ord.rs
fn main() {
    let input = std::env::args().nth(1).unwrap_or_default();
    for c in input.chars() {
        print!("{} ", char_to_hex(c));
    }
    println!();
}

fn char_to_hex(c: char) -> String {
    format!("{:X}", c as u32)
}
```

```rust:src/bin/chr.rs
use std::env;
use std::io::{self, Read};

fn main() {
    let args: Vec<String> = env::args().skip(1).collect();
    let input = if !args.is_empty() {
        args.join(" ")
    } else {
        let mut buf = String::new();
        io::stdin().read_to_string(&mut buf).unwrap();
        buf
    };

    for hex in input.split_whitespace() {
        match u32::from_str_radix(hex.trim_start_matches("U+"), 16)
            .ok()
            .and_then(std::char::from_u32)
        {
            Some(c) => print!("{}", c),
            None => eprintln!("Invalid codepoint: {}", hex),
        }
    }
    println!();
}
```

ビルドします。

```
cargo build
```

個別にビルドすることもできます。

```
cargo build --bin ord
cargo build --bin chr
```

テストします。

```
> target/debug/ord あ漢🍣
3042 6F22 1F363 
```

```
> target/debug/chr 3042 6F22 1F363
あ漢🍣
```

```
> echo "3042 6F22 1F363" | target/debug/chr
あ漢🍣
```

