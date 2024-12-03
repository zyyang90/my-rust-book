# 错误处理

## 错误类型

rust 中，错误可以分为两种类型：

1. 不可恢复的错误

* panic!：程序崩溃，直接退出
* todo!()/ unimplemented!()：功能未实现，程序退出
* unreachable!()：代码不可达，程序退出
* assert!()：断言失败，程序退出

2. 可恢复的错误： Result<T, E>，错误作为返回值传递，交给调用者处理

## 错误处理要做什么事情？

错误处理相关的工作包括：

1. 错误的表示和构造
2. 错误的传递
3. 错误处理

下面是一个简单的示例：

```rust
use std::error::Error;
use std::fmt::{Debug, Display, Formatter};

// 函数 foo 成功时，返回字符串，失败时返回 Box<dyn Error> 类型的错误
fn foo(s: &str) -> Result<String, Box<dyn Error>> {
    if s.len() == 0 {
        return Err(Box::new(MyError {}));
    }
    Ok(s.to_string())
}

// 通过 derive 宏为 MyError 实现 Debug trait
#[derive(Debug)]
struct MyError {}

// 为 MyError 实现 Display trait
impl Display for MyError {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        write!(f, "length cannot be zero")
    }
}

// 为 MyError 实现 Error trait
impl Error for MyError {}

fn main() {
    match foo("") {
        // 成功时，打印字符串
        Ok(s) => {
            println!("{}", s);
        }
        // 错误处理
        Err(err) => {
            print!("{}", err.to_string());
        }
    }
}
```

### Result<T, E>

Result<T, E> 是一个枚举类型，有两个变体：Ok(T) 和 Err(E)。 T 是成功时返回值的类型，E 是错误时返回值的类型。
注意：E 可以是任何类型，但通常是实现了 std::error::Error trait 的类型。

```rust
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### Error trait

std::error::Error trait 是标准库提供的错误处理 trait，用于表示错误。

```rust
pub trait Error: Debug + Display {
    // ...
}
```

## 常见的错误

### std::io::Error

std::io::Error 是标准库提供的 I/O 错误类型，实现了 Error trait。std::io::Error 是一个结构体，包含了错误码、错误信息等。

```rust
fn main() {
    let err = Error::new(std::io::ErrorKind::Other, "An error occurred");
    println!("{:?}", err);
}
```

### parse Error

```rust
use std::char::ParseCharError;
use std::fmt::{Debug, Display};
use std::net::{AddrParseError, Ipv4Addr};
use std::num::{ParseFloatError, ParseIntError};
use std::str::ParseBoolError;

fn main() {
    let res: ParseIntError = "abc".parse::<i32>().err().unwrap();
    println!("{:?}", res);

    let res: ParseFloatError = "abc".parse::<f32>().err().unwrap();
    println!("{:?}", res);

    let res: ParseCharError = "abc".parse::<char>().err().unwrap();
    println!("{:?}", res);

    let res: ParseBoolError = "abc".parse::<bool>().err().unwrap();
    println!("{:?}", res);

    let res: AddrParseError = "abc".parse::<Ipv4Addr>().err().unwrap();
    println!("{:?}", res);
}
```

## 用 thiserror 构建自定义错误

最常见的，是通过 enum 来定义错误类型。

```rust
use std::io::{Error, ErrorKind};
use thiserror::Error;

#[derive(Debug, Error)]
enum DataStoreError {
    #[error("data store disconnected, cause: {0}")]
    Disconnect(#[from] std::io::Error),
    #[error("the data for key `{0}` is not available")]
    Redaction(String),
    #[error("invalid header (expected: {expected}, found: {found})")]
    InvalidHeader { expected: String, found: String },
    #[error("unknown data store error")]
    Unknown,
}

fn main() {
    let error = DataStoreError::Disconnect(Error::new(ErrorKind::Other, "my error"));
    println!("{}", error);

    let error = DataStoreError::Redaction("abc".to_string());
    println!("{}", error);

    let error = DataStoreError::InvalidHeader {
        expected: "a".to_string(),
        found: "b".to_string(),
    };
    println!("{}", error);

    let error = DataStoreError::Unknown;
    println!("{}", error);
}
```

## 用 anyhow 传递错误

```rust
use thiserror::Error;

#[derive(Debug, Error)]
enum DataStoreError {
    #[error("data store disconnected, cause: {0}")]
    Disconnect(#[from] std::io::Error),
    #[error("the data for key `{0}` is not available")]
    Redaction(String),
    #[error("invalid header (expected: {expected}, found: {found})")]
    InvalidHeader { expected: String, found: String },
    #[error("unknown data store error")]
    Unknown,
}

// 处理 DataStoreError 类型的错误，如果 error 类型不是 DataStoreError，则立即返回一个 anyhow::Error
fn handle_data_store_error(errors: Vec<Box<dyn std::error::Error>>) -> anyhow::Result<usize> {
    let mut count = 0;
    for error in errors {
        // downcast
        let err = error
            .downcast_ref::<DataStoreError>()
            .ok_or(anyhow::anyhow!(
                "unable to downcast to DataStoreError, error: {error}"
            ))?;

        println!("{}", err);
        count += 1;
    }

    Ok(count)
}

fn main() {
    let errors: Vec<Box<dyn std::error::Error>> = vec![
        Box::new(DataStoreError::Disconnect(std::io::Error::new(
            std::io::ErrorKind::Other,
            "my error",
        ))),
        Box::new(DataStoreError::Redaction("abc".to_string())),
        Box::new(DataStoreError::InvalidHeader {
            expected: "a".to_string(),
            found: "b".to_string(),
        }),
        // std::io::Error
        Box::new(std::io::Error::new(std::io::ErrorKind::Other, "my error")),
        // ParseIntError
        Box::new("abc".parse::<i32>().unwrap_err()),
        Box::new(DataStoreError::Unknown),
    ];

    let res = handle_data_store_error(errors);
    assert!(res.is_err());
    println!("anyhow::Result: {:?}", res)
}
```

### 使用 anyhow::Error 为 Error 添加更多上下文信息

```rust
use std::io::ErrorKind;

/// std::error::Error 是一个 trait，它是标准库中所有错误类型的基本特征。
/// std::error::Error 要求实现 Display 和 Debug 两个 trait。
/// ```rust
/// pub trait Error: Debug + Display {
/// ...
/// }
/// ```
fn main() {
    // std::io::Error 是一个 struct，它实现了 std::error::Error trait。
    // 通过 std::io::Error::new 创建一个 std::io::Error 实例。
    let io_err = std::io::Error::new(ErrorKind::Other, "An error occurred");
    // 打印 {io_err:?} 时，调用的是 std::fmt::Debug trait 的实现。
    // 输出为：Custom { kind: Other, error: "An error occurred" }
    println!("{io_err:?}");
    // 打印 “{io_err}” 时，调用的是 std::fmt::Display trait 的实现。
    // 输出为：An error occurred
    println!("{io_err}");

    // anyhow::Error 也是一个 struct，它也实现了 std::error::Error trait。
    // anyhow::Error 为 std::error::Error trait 提供了更多的上下文信息。
    // 通过 anyhow::Error::new 创建一个 anyhow::Error 实例。
    let anyhow_err = anyhow::Error::new(io_err)
        // context 方法可以为 anyhow::Error 添加上下文信息。
        .context("A error")
        .context("B error")
        .context("main error");
    // 打印 {res:?} 时，调用的是 std::fmt::Debug trait 的实现。
    // 输出了 anyhow::Error 的详细信息，包括caused by 和 backtrace。
    println!("{anyhow_err:?}");
    // 打印 “{res}” 时，调用的是 std::fmt::Display trait 的实现。
    // 输出为：main error，即：当前错误的 context。
    println!("{anyhow_err}");
    // root_cause 方法可以获取原始错误。
    // 输出为：An error occurred，即：原始错误的 error。
    println!("{}", anyhow_err.root_cause());
    // 打印 “{:#}” 时，调用的是 std::fmt::Debug trait 的实现。
    // 输出为：main error: B error: A error: An error occurred, 即：当前 context 到 root cause 的链。
    println!("{:#}", anyhow_err);
}
```

## 错误处理的最佳实践

1. 使用 thiserror::Error 来构建自定义的错误类型。
2. 使用 anyhow::Error 来传递错误。
3. 在日志中打印错误信息时，打印 {:#}，打印从当前 cause 到 root cause 的链，且在同一行打印。

### {}, {:#}, {:?}, {:#?} 的区别

{}：默认格式化，调用 Display trait 的实现。
{:#}：美化的默认格式化，也是调用 Display trait 的实现，输出更具可读性（通常会插入换行符和缩进），由具体实现决定。
{:?}：调试格式化，调用 Debug trait 的实现，输出的信息可能比默认格式化更详细。
{:#?}：美化的调试格式化，也是调用 Debug trait 的实现，由具体实现决定。

### anyhow::Error 的 Display 实现

```rust
pub(crate) unsafe fn display(this: Ref<Self>, f: &mut fmt::Formatter) -> fmt::Result {
    write!(f, "{}", unsafe { Self::error(this) })?;

    if f.alternate() {
        let chain = unsafe { Self::chain(this) };
        for cause in chain.skip(1) {
            write!(f, ": {}", cause)?;
        }
    }

    Ok(())
}
```

可以看到，当使用 {:#} 时，会打印从当前 cause 到 root cause 的链，且在同一行打印。
