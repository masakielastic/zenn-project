---
title: "PHP スクリプトを実行する Rust CLI ツールを作る"
emoji: "🦀"
type: "tech"
topics: ["rust", "php"]
published: true
---


序文
----

PHP スクリプトを Rust 製の CLI ツールから実行できたら便利だと思ったことはありませんか？

本記事では、PHP に標準で用意されている埋め込み用インターフェース（embed SAPI）を使って、Rust から PHP スクリプトを実行する最小限の CLI ツールを構築してみます。C 言語で php_embed_init() を呼び出し、Rust 側からは FFI 経由でこの関数を叩く構成です。

具体的には、以下の 2 つの処理例を紹介します：

 * .php ファイルを実行する CLI ツール -  `zend_execute_scripts()` 
 * REPL の作成 - `zend_eval_string()`

Rust 側は FFI の導入にとどめ、ビルド設定や環境の調整も含めて、できるだけシンプルな構成を心がけました。PHP モジュール（例: mbstring）の読み込みや php.ini の指定などにも触れながら、embed SAPI の動作を Rust から検証していきます。C言語やPHPエクステンションの知識がなくても読めるように構成していますので、PHPとRustの橋渡しに関心のある方はぜひ最後まで読んでみてください。

検証環境
----

Debian 12 でDEB.SURY.ORG から  php8.4-dev や libphp8.4-embed などがインストールされていることを前提とします。`php.ini` や拡張モジュールのファイルは  `/etc/php/8.4/embed/` 以下にインストールされています。

PHP スクリプトを実行する CLI の作成
-----------------------------------

Rust のプロジェクトを作成します。

### 構成 

Rust から PHP を埋め込み実行する CLI ツール（および REPL）プロジェクトの 最小構成ファイルツリー を表すアスキーアートです：

```
php-cli/
├── Cargo.toml
├── build.rs
├── wrapper.c
├── php
│   └── test.php
└── src/
    └── main.rs

```

ファイル概要：

 * `Cargo.toml`：Rust プロジェクトのメタデータと依存関係。
 * `build.rs`：cc クレートを使って `wrapper.c` をビルド・リンク。
 * `wrapper.c`：C 言語で PHP エンジン初期化・実行を行うラッパー関数群。
 * `php/test.php`：実行する PHP スクリプト（例：`<?php var_dump("Hello");`）。
 * `src/main.rs`：CLI 引数や REPL を受け取って `wrapper.c` の関数を呼び出す。

### コード

```toml:Cargo.toml
[package]
name = "php_cli"
version = "0.1.0"
edition = "2021"
build = "build.rs"

[build-dependencies]
cc = "1.0"

[dependencies]
libc = "0.2"
```


```rust:build.rs
fn main() {
    println!("cargo:rustc-link-lib=php");
    println!("cargo:rustc-link-search=native=/usr/lib");

    cc::Build::new()
        .file("wrapper.c")
        .include("/usr/include/php/20240924")
        .include("/usr/include/php/20240924/main")
        .include("/usr/include/php/20240924/Zend")
        .include("/usr/include/php/20240924/TSRM")
        .include("/usr/include/php/20240924/sapi/embed")
        .compile("wrapper");
}
```

```c:wrapper.h
#ifndef WRAPPER_H
#define WRAPPER_H

int run_php_script(const char* filename);

#endif
```


```c:wrapper.c
#include <php_embed.h>
#include <php_main.h>
#include <Zend/zend_string.h>

int run_php_script(const char* filename) {
    zend_file_handle file_handle;

    if (php_embed_init(0, NULL) == FAILURE) {
        return 1;
    }

    zend_string *fname = zend_string_init(filename, strlen(filename), 0);

    file_handle.type = ZEND_HANDLE_FILENAME;
    file_handle.filename = fname;
    file_handle.opened_path = NULL;
    file_handle.handle.fp = NULL;
    file_handle.buf = NULL;
    file_handle.primary_script = 1;

    zend_execute_scripts(ZEND_REQUIRE, NULL, 1, &file_handle);

    php_embed_shutdown();
    return 0;
}
```

```rust:src/main.rs
use std::ffi::CString;
use std::env;

extern "C" {
    fn run_php_script(filename: *const libc::c_char);
}

fn main() {
    let args: Vec<String> = env::args().collect();
    if args.len() < 2 {
        eprintln!("Usage: {} <script.php>", args[0]);
        std::process::exit(1);
    }

    let filename = CString::new(args[1].clone()).expect("CString::new failed");
    unsafe {
        run_php_script(filename.as_ptr());
    }
}
```

```php:test.php
<?php
echo "Loaded ini file: " . php_ini_loaded_file() . PHP_EOL;
echo "Scanned ini files: " . php_ini_scanned_files() . PHP_EOL;
echo "mbstring loaded: " . (extension_loaded("mbstring") ? "yes" : "no") . PHP_EOL;
```

### ビルドと実行

ビルドします。

```sh
cargo clean
cargo build --release
```

スクリプトを実行します。

```sh
target/release/php_cli php/test.php
```

結果は次のようになります。

```sh
Loaded ini file: /etc/php/8.4/embed/php.ini
Scanned ini files: /etc/php/8.4/embed/conf.d/10-opcache.ini,
/etc/php/8.4/embed/conf.d/10-pdo.ini,
/etc/php/8.4/embed/conf.d/15-xml.ini,
/etc/php/8.4/embed/conf.d/20-calendar.ini,
/etc/php/8.4/embed/conf.d/20-ctype.ini,
/etc/php/8.4/embed/conf.d/20-curl.ini,
/etc/php/8.4/embed/conf.d/20-dom.ini,
/etc/php/8.4/embed/conf.d/20-exif.ini,
/etc/php/8.4/embed/conf.d/20-ffi.ini,
/etc/php/8.4/embed/conf.d/20-fileinfo.ini,
/etc/php/8.4/embed/conf.d/20-ftp.ini,
/etc/php/8.4/embed/conf.d/20-gettext.ini,
/etc/php/8.4/embed/conf.d/20-iconv.ini,
/etc/php/8.4/embed/conf.d/20-intl.ini,
/etc/php/8.4/embed/conf.d/20-mbstring.ini,
/etc/php/8.4/embed/conf.d/20-phar.ini,
/etc/php/8.4/embed/conf.d/20-posix.ini,
/etc/php/8.4/embed/conf.d/20-readline.ini,
/etc/php/8.4/embed/conf.d/20-shmop.ini,
/etc/php/8.4/embed/conf.d/20-simplexml.ini,
/etc/php/8.4/embed/conf.d/20-sockets.ini,
/etc/php/8.4/embed/conf.d/20-sysvmsg.ini,
/etc/php/8.4/embed/conf.d/20-sysvsem.ini,
/etc/php/8.4/embed/conf.d/20-sysvshm.ini,
/etc/php/8.4/embed/conf.d/20-tokenizer.ini,
/etc/php/8.4/embed/conf.d/20-xdebug.ini,
/etc/php/8.4/embed/conf.d/20-xmlreader.ini,
/etc/php/8.4/embed/conf.d/20-xmlwriter.ini,
/etc/php/8.4/embed/conf.d/20-xsl.ini

mbstring loaded: yes
```

REPL の作成
-----------

今度は `zend_eval_string()` の使い方を学ぶために REPL を作成します。

### コード

```toml:Cargo.toml
[package]
name = "php_repl"
version = "0.1.0"
edition = "2021"

[dependencies]
libc = "0.2"
rustyline = "13"

[build-dependencies]
cc = "1.0"

[build]
build = "build.rs"

[profile.release]
strip = "debuginfo"
```

```rust:build.rs
fn main() {
    println!("cargo:rustc-link-lib=php");
    println!("cargo:rustc-link-search=native=/usr/lib");
    println!("cargo:rerun-if-changed=wrapper.c");

    cc::Build::new()
        .file("wrapper.c")
        .include("/usr/include/php/20240924")
        .include("/usr/include/php/20240924/main")
        .include("/usr/include/php/20240924/Zend")
        .include("/usr/include/php/20240924/TSRM")
        .include("/usr/include/php/20240924/sapi/embed")
        .compile("wrapper");
}
```

```c:wrapper.c
#include <php_embed.h>
#include <zend.h>

void eval_php_code(const char *code) {
    PHP_EMBED_START_BLOCK(0, NULL)
    zend_eval_string((char *)code, NULL, "REPL");
    PHP_EMBED_END_BLOCK()
}
```

```rust:src/main.rs
use rustyline::{Editor};
use rustyline::history::DefaultHistory;
use rustyline::error::ReadlineError;
use std::ffi::CString;


extern "C" {
    fn php_embed_init(argc: i32, argv: *mut *mut i8) -> i32;
    fn php_embed_shutdown() -> i32;
    fn zend_eval_string(code: *const i8, retval: *mut std::ffi::c_void, name: *const i8) -> i32;
}

fn main() {
    let args: Vec<std::ffi::CString> = std::env::args().map(|arg| CString::new(arg).unwrap()).collect();
    let mut c_args: Vec<*mut i8> = args.iter().map(|arg| arg.as_ptr() as *mut i8).collect();
    c_args.push(std::ptr::null_mut());

    unsafe { php_embed_init((c_args.len() - 1) as i32, c_args.as_mut_ptr()); }

    let mut rl = Editor::<(), DefaultHistory>::new().unwrap();
    println!("Rust PHP REPL（exit で終了）");

    loop {
        match rl.readline("php> ") {
            Ok(line) => {
                if line.trim() == "exit" {
                    break;
                }
                rl.add_history_entry(line.as_str()).unwrap();
                let code = CString::new(line).unwrap();
                let name = CString::new("REPL").unwrap();
                unsafe {
                    zend_eval_string(code.as_ptr(), std::ptr::null_mut(), name.as_ptr());
                }
            }
            Err(ReadlineError::Interrupted | ReadlineError::Eof) => break,
            Err(e) => {
                eprintln!("エラー: {e}");
                break;
            }
        }
    }

    unsafe { php_embed_shutdown(); }
}
```

### ビルドと実行

コマンドを実行して REPL を起動させます。

```sh
target/release/php_repl
```

PHP スクリプトの実行結果を表示させるには `echo` と改行の組み合わせ、もしくは `var_dump` を使います。

```sh
Rust PHP REPL（exit で終了）
php> echo "Hello", PHP_EOL;
Hello
php> echo "Hello";
php> var_dump("Hello");
REPL:1:
string(5) "Hello"
php> exit
```


各言語の REPL 実装にみる「標準入力」「標準出力」「一時ファイル」処理の比較
------------------------------------------------

REPL（Read-Eval-Print Loop）は、対話的にコードを実行できる便利な仕組みですが、その実装方法には各言語の設計思想やランタイムの特性が反映されています。ここでは、標準入力／出力の扱いや一時ファイル生成の有無に注目して、PHP（標準／PsySH）、Python、Haskell（GHCi）を比較します。

### 🔵 PHP 標準の REPL（php -a）

 * **標準入力**：`php -a` は標準入力を逐次読み取って評価します。行末に `;` がないと未完了とみなされ、複数行のコードも構築可能です。
 * **標準出力**：`print_r` や `var_dump` の結果はそのまま標準出力に出力されます。
 * **一時ファイル**：使用しません。すべてが stdin → eval で処理されます。
 * **制限**：エラー出力と通常出力が混在しやすく、補完や履歴機能がありません。

### 🟣 PsySH（PHP 用高度 REPL）

 * **標準入力**：`readline()` ベースで構築されており、履歴・補完・マルチライン対応。
 * **標準出力**：REPL 内部でラップされており、`echo` や `print_r` も整形されて表示されます。
 * **一時ファイル**：実行ごとに一時ファイルを生成して `require` しています。これにより「スコープを保持」しつつ例外処理が可能。
 * **メリット**：デバッグ機能、オブジェクトインスペクション、トレースバック付き例外など、REPL をフル IDE のように使える。

### 🟡 Python 標準 REPL（CPython）

 * **標準入力**：stdin を Python プロンプトが読み取り、コードブロック（if 文、関数定義など）も自動的にインデントを保持して読込。
 * **標準出力**：`print()` などの出力はすべて `stdout` に送られ、値は暗黙的に `repr()` によって表示されます。
 * **一時ファイル**：使用しません。実行はインメモリの `code.InteractiveConsole` 上で行われます。
 * **特徴**：一時ファイルを使わずに状態を保持し、_ 変数で前回の出力値を参照できる機能もある。

### 🟢 Haskell GHCi

 * **標準入力**：ghci は独自の対話的環境を提供しており、標準入力でのコード読込、コマンド（`:type`、`:reload`など）を分離。
 * **標準出力**：評価結果は `Show` インスタンスを使って整形されて出力。
 * **一時ファイル**：明示的に `:load` などでソースファイルを読み込む場合を除き、式の評価はインメモリ（インタプリタ）で完結。
 * **特徴**：型情報を保持しつつ、REPL 内で関数定義・パターンマッチなどを柔軟に試すことができる。REPL 内部で匿名関数や `do` 記法も扱える。
 * **一時ファイル**：明示的に `:load` などでソースファイルを読み込む場合を除き、式の評価はインメモリ（インタプリタ）で完結。

特徴：型情報を保持しつつ、REPL 内で関数定義・パターンマッチなどを柔軟に試すことができる。REPL 内部で匿名関数や do 記法も扱える。

### 🟢 Rust evcxr_repl

 * **読み方**：エヴクサー・リープル（evcxr = evaluate cxr、公式読みは "ev-czar" に近い）
 * **標準入力**：Rust ソース断片を入力として受け取り、即座に main 関数に挿入される形で評価。
 * **標準出力**：任意の型の値を Display ではなく Debug（{:?}）形式で表示。カスタム表示用のプリンタ定義も可能。
 * **一時ファイル**：毎回 Cargo プロジェクトを内部で生成し、ソースをファイルに書き出してコンパイル実行（cargo check 風）。
 * **特徴**：
   * Rust コンパイラ (rustc) をそのまま使っており、コンパイルエラーもそのまま表示。
   * `:dep rand = "0.8"` のように REPL 内から依存クレートの追加が可能。
   * `let x = 1;` などの状態が REPL セッションを通じて保持される。


### 比較表（まとめ）

| 言語/ツール | 標準入力        | 標準出力            | 一時ファイル生成   | 備考             |
| ---------- | ---------------- | ------------------- | ------------------ | ---------------- |
| PHP `-a`   | 行単位           | 直接出力            | ×                  | 簡易的、補完なし |
| PsySH      | 高機能入力       | 整形表示            | ○                  | require を活用して柔軟性 |
| Python     | ブロック構文OK   | repr 表示           | ×                  | `_` で前回値参照可 |
| GHCi       | 型推論あり       | Show 表示           | ×（ただしロード可）| コマンドと式を区別可能 |
| evcxr_repl | Cargo スニペット | Debug 表示（`{:?}`）| ○                  |毎回コードを再構築・ビルドする|

### Rust × PHP CLI 実装に応用するなら？

今回の Rust 製 PHP CLI のように `zend_eval_string()` を利用した簡易 REPL を実装する場合、**標準入力からの読み取り＋その場評価**という PHP 標準の `-a` モードに近い構成になります。ただし `eval` ではスコープが継続されないため、PsySH のように一時ファイルを用いるとスコープ維持・例外管理などが容易になります。

**補足**：

* Rust + `rustyline` で履歴・補完対応
* 評価には `zend_eval_string()` または `zend_execute_scripts()` を利用
* 例外処理や変数保持には `eval` 方式より `require` 方式のほうが好ましい

PHP の C ヘッダを bindgen で解析しようとして失敗
-----------------------------------------------

Rust で PHP の C API（たとえば `php_embed.h` や `zend_API.h`）を利用しようとしたとき、自然に考えるのが bindgen の活用です。自動で C ヘッダを解析して FFI バインディングを生成してくれるので、便利なツールとして知られています。

今回は PHP 8.4 (deb.sury.org 由来) の開発ヘッダを対象に bindgen を使ってみたのですが、結論から言うとうまくいきませんでした。以下、その経緯と教訓を簡単にまとめておきます。


### ✅ 目的

Rust のコードから `php_embed_init()` や `php_execute_script()` などを呼び出すために、PHP のヘッダ（特に `/usr/include/php/20240924/sapi/embed/php_embed.h`）を bindgen で Rust に変換しようとしました。


### ❌ 実行してみたが……

```sh
bindgen /usr/include/php/20240924/sapi/embed/php_embed.h -o bindings.rs
```

このように単純にコマンドを打っても、以下のようなエラーが発生しました：

 * fatal error: 'main/php.h' file not found

 * `#include_next` ディレクティブが原因でパス解決に失敗

 * `ZEND_API` や `BEGIN_EXTERN_C` など、PHP独自のマクロが多数定義されており、前提のヘッダをすべて含めないと解決不能

### 🔍 問題の原因

 * `include` パスが複雑すぎる
`/usr/include/php/20240924` の下に `main/`, `Zend/`, `TSRM/`, `sapi/` など複数ディレクトリがあり、どれも重要な依存関係を持っています。
 * ビルド時に使われる `-D` 定義が大量に必要 `ZEND_API`, `PHPAPI`, `ZTS, HAVE_STDARG_PROTOTYPES` などのマクロ定義が不足しており、bindgen での解決が難しい。
 * `php.h` の中で `#include_next` が使われている GCC/Clang 依存のディレクティブが含まれ、bindgen の処理が中断される。

### 🧠 教訓

 * PHP のヘッダファイルは「embed 用」でもそのまま bindgen で解析できるような設計にはなっていない。
 * FFI で呼び出したい関数が数個に限られている場合は、自前で extern "C" 宣言を書く方が早くて安全。
 * Rust 側から PHP を embed するなら、wrapper.c で橋渡し関数を作ってバイナリリンクする方が実用的。
 * bindgen は万能ではなく、対象のヘッダが「解析可能」であることが前提条件になる。

### ✅ 代替策
最終的には以下のような構成に落ち着きました：

 * `wrapper.c` で PHP の初期化・スクリプト実行関数をまとめる
 * Rust 側では `extern "C"` でラップされた関数だけを呼び出す
 * `build.rs` で `cc::Build` を使って `wrapper.c` を静的リンクする

```rust
extern "C" {
    fn run_php_script(path: *const c_char);
}
```

### 📝 まとめ
bindgen は強力なツールですが、「レガシーで巨大な C プロジェクト」や「ビルド前提のプリプロセッサ依存が強いヘッダ」には向かないケースもあります。PHP のような複雑なヘッダ構成を持つ言語処理系では、最小限の C ラッパーを書く方がスムーズに Rust から呼び出せるようになります。


なぜ `zend_execute_script` ではなく `zend_execute_scripts` を使ったのか？
----------------------------

PHP エンジンを Rust から呼び出す場合、C 言語の API に直接アクセスする必要があります。スクリプトを実行するための関数としては、`zend_execute_script` と `zend_execute_scripts` の2つが代表的に存在します。

今回は、`zend_execute_scripts` を採用しました。その理由は以下の通りです：

 * `zend_execute_scripts` は 複数のスクリプトを順に実行できる柔軟な関数です。
 * `zend_execute_script` は 単一スクリプト専用であり、構造体 `zend_file_handle` の準備が正しくできている必要があります。
 * `zend_execute_scripts` は `ZEND_REQUIRE` を指定することで、`require` されたスクリプトと同様の扱いができます。

そして実際のところ、PHP の CLI 実行（たとえば `php -f file.php`）でも `zend_execute_scripts` が使われています。

このように、柔軟性と互換性の面から `zend_execute_scripts` のほうが CLI ツールには向いていると判断しました。

### `zend_execute_script` の出番は？

とはいえ、`zend_execute_script` も無視できる関数ではありません。特に `php-src/sapi/` 以下の各 SAPI 実装（たとえば Apache や CGI、FPM など）では、この関数が頻繁に使われています。PHP が Web サーバー上で動作する際の「実行の起点」がこの関数なのです。

参考になる実装：
 * `php-src/sapi/cli/php_cli.c`
 * `php-src/sapi/cgi/cgi_main.c`
 * `php-src/sapi/fpm/fpm_main.c`

興味のある方は、これらのソースコードを読んでみることで、PHP の SAPI レイヤーがどのように動いているのかを深く理解できるでしょう。

### LiteSpeed SAPI と `.htaccess`

今回の調査中に、**LiteSpeed SAPI** が `.htaccess` を**自前で処理している**ことを知りました。これは驚きでした。

通常 `.htaccess` の処理は Apache 側で行うものと考えられがちですが、LiteSpeed は自分で `.htaccess` を読み込み、内部で再構成された設定を PHP に反映しているのです。これは SAPI の設計と実装が Web サーバーに深く結びついている証拠でもあり、**SAPI によって PHP の挙動が異なる**ことをあらためて実感させられました。

PHP の動作の裏側を理解するには、**C 言語による SAPI 実装のソースコードを読むことが最高の学び**になります。Rust や PHP の知識と組み合わせることで、もっと柔軟で自由なツール作りが可能になります。
