---
title: "Rust で最小構成の HTTP サーバー"
emoji: "🦀"
type: "tech"
topics: ["rust", "hyper"]
published: false
---

## 検証環境

Debian 12 Bookworm で検証しました。

## ファイル構成

```
http_server_min/
├── Cargo.toml
└── src/
    └── main.rs
```

## 最小構成の HTTP サーバー

### コード
```toml:Cargo.toml
[package]
name = "http_min_auto"
version = "0.1.0"
edition = "2024"

[dependencies]
tokio = { version = "1.4", features = ["full"] }
hyper = { version = "1.6", features = ["http1", "http2", "server"] }
hyper-util = { version = "0.1", features = ["server-auto", "tokio"] }
http-body-util = "0.1"
bytes = "1"
anyhow = "1"
```

```rs:src/main.rs
use hyper::{Request, Response};
use hyper::body::Incoming;
use hyper_util::rt::{TokioExecutor, TokioIo};
use hyper_util::server::conn::auto::Builder;
use tokio::net::TcpListener;
use hyper::service::service_fn;
use http_body_util::Full;
use bytes::Bytes;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:3000").await?;
    println!("Listening on http://127.0.0.1:3000");

    loop {
        let (stream, _) = listener.accept().await?;
        let executor = TokioExecutor::new();
        let builder = Builder::new(executor);
        let io = TokioIo::new(stream);
        let service = service_fn(handle);

        tokio::spawn(async move {
            if let Err(e) = builder.serve_connection(io, service).await {
                eprintln!("error: {}", e);
            }
        });
    }
}

async fn handle_request(_req: Request<Incoming>) -> Result<Response<Full<Bytes>>, hyper::Error> {
    let body = Full::new(Bytes::from("Hello World!"));
    Ok(Response::new(body))
}
```

### ビルドと実行

```bash
cargo run
```

もしくは

```bash
cargo build --release
cargo target/release/http_server_min
```

を実行します。

## ファイルの読み込み

今度はファイルを読み込み、表示する HTTP サーバーを作ります。ちょっと機能を追加するだけでコードの分量がけっこう増えるので、別のプロジェクトフォルダを作ることをおすすめします。

### プログラムコード
```toml:Cargo.toml
[package]
name = "http_server_min"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { version = "1.4", features = ["full"] }
hyper = { version = "1.6", features = ["http1", "http2", "server"] }
hyper-util = { version = "0.1", features = ["server-auto", "tokio"] }
http-body-util = "0.1"
bytes = "1"
anyhow = "1"
mime_guess = "2"
```

```rust:src/main.rs
use hyper::{
    Request, Response, StatusCode,
    body::Incoming,
    service::service_fn,
};

use hyper_util::rt::{TokioExecutor, TokioIo};
use hyper_util::server::conn::auto::Builder;
use tokio::{fs, net::TcpListener};
use http_body_util::Full;
use std::path::PathBuf;
use bytes::Bytes;
use mime_guess::from_path;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let listener = TcpListener::bind("127.0.0.1:3000").await?;
    println!("Listening on http://127.0.0.1:3000");

    loop {
        let (stream, _) = listener.accept().await?;
        let executor = TokioExecutor::new();
        let builder = Builder::new(executor);
        let io = TokioIo::new(stream);
        let service = service_fn(handle_request);

        tokio::spawn(async move {
            if let Err(e) = builder
                .serve_connection(io, service)
                .await
            {
                eprintln!("Error: {:?}", e);
            }
        });
    }
}

async fn handle_request(_req: Request<Incoming>) -> Result<Response<Full<Bytes>>, hyper::Error> {
    let path = PathBuf::from("static/hello.txt");

    match fs::read(&path).await {
        Ok(contents) => {
            let mime = from_path(&path).first_or_octet_stream();
            let mut response = Response::new(Full::new(Bytes::from(contents)));
            response.headers_mut().insert(
                hyper::header::CONTENT_TYPE,
                hyper::header::HeaderValue::from_str(mime.as_ref()).unwrap(),
            );
            Ok(response)
        }
        Err(_) => {
            let mut response = Response::new(Full::new(Bytes::from("404 Not Found")));
            *response.status_mut() = StatusCode::NOT_FOUND;
            Ok(response)
        }
    }
}
```

### ビルドと実行

```bash
cargo run
```

を実行します。
