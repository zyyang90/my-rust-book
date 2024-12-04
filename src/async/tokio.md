# tokio

## 基础

### async/await

`async` 关键字修饰 function 或者代码块，返回的是一个 Future；`await` 关键字，会在当前线程等待 Future 执行完毕。下面是一个示例：

```rust

#[tokio::main]
async fn main() {
    // async block
    let a = async {
        println!("hello rust");
    };
    // async function
    let b = say_hello();
    // wait until b completed
    b.await;
    // wait until a completed
    a.await;
}

async fn say_hello() {
    println!("hello world");
}
```

### #[tokio::main]

`#[tokio::main]` 是一个宏，可以将 `async fn main()` 转换成一个同步的 `fn main()`，初始化异步 Runtime 并执行 main。
它等效于下面的代码：

```rust
fn main() {
    let mut rt = tokio::runtime::Runtime::new().unwrap();
    rt.block_on(async {
        println!("hello world!");
        tokio::time::sleep(std::time::Duration::from_secs(1)).await;
        println!("goodbye world!");
    });
}
```

### tokio::spawn

tokio::spawn会创建一个异步任务，并交给 Runtime 运行。这个函数接收一个实现了 Future trait 的参数，并立即在后台执行这个
Future。tokio::spawn返回一个 JoinHandle，你可以通过JoinHandle来获取结果。

```rust
#[tokio::main]
async fn main() {
    let op = say_hello();

    tokio::time::sleep(Duration::from_secs(1)).await;
    println!("hello ");

    op.await;
}

async fn say_hello() {
    let a = tokio::spawn(async {
        tokio::time::sleep(Duration::from_secs(1)).await;
        println!("aaa");
    });

    let b = tokio::spawn(async {
        tokio::time::sleep(Duration::from_secs(2)).await;
        println!("bbb");
    });

    println!("world~");

    a.await.unwrap();
    b.await.unwrap();
}
```

###                                

如果异步 task 有返回值的，可以用 handle 来获取。

```rust

```

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