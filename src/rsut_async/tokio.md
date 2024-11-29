# tokio

## 基础

## 异步 Runtime

```rust
async fn foo() {
    // async block 是一个异步闭包，类型是 impl Future<Output = ()> + Sized
    let a = async {
        println!("Hello, world!");
    };
    // 调用 await
    a.await;
}

fn main() {
    tokio::runtime::Builder::new_multi_thread()
        .enable_all()
        .build()
        .unwrap()
        .block_on(foo())
}
```

## tokio 读写文件

```rust
use std::path::Path;
use tokio::fs::File;
use tokio::io::{AsyncReadExt, AsyncWriteExt};

#[tokio::main]
async fn main() {
    let file = "foo.txt";

    write_file(file).await.expect("write file error");

    let content = read_file(file).await.expect("read file error");
    println!("{}", String::from_utf8(content).unwrap());
}

async fn write_file(file: impl AsRef<Path>) -> std::io::Result<()> {
    let mut f = File::create(file).await?;
    f.write_all(b"Hello, world!").await?;

    Ok(())
}

async fn read_file(file: impl AsRef<Path>) -> std::io::Result<Vec<u8>> {
    let mut f = File::open(file).await?;
    let mut buf = vec![];
    f.read_to_end(&mut buf).await?;

    Ok(buf)
}
```

## 定时器

```rust
use chrono::Local;
use std::time::Duration;

#[tokio::main]
async fn main() {
    let mut interval = tokio::time::interval(Duration::from_secs(1));

    for _ in 0..10 {
        interval.tick().await;
        println!("{:?}", Local::now().to_rfc3339());
    }
}
```