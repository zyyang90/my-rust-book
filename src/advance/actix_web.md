# Actix-Web

## 概念

- App Struct：表示应用程序，用于配置路由和一些公共参数。
- HttpServer Struct：表示 web server，用于实例化 sever。
- Web 模块：
- HttpRequest、HttpResponse Struct：表示 Http 请求和响应。
- Handler：一个异步函数，接收 0 个或多个参数，这些参数是从 http 请求中使用 extractor 提取的；返回一个可以被转换为
  HttpResponse 的结果。
- Extractor：提取器，从 HTTP 报文中提取。

## App

### State

每个 worker 可以有自己独立的状态，用web::Data<T> extractor 访问。

```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(move || {
        // 每个 worker 都有自己的 AppState 实例
        App::new()
            .app_data(web::Data::new(AppState {
                app_name: "Actix-web".to_string(),
            }))
            .service(state)
    }).bind(("127.0.0.1", 8080))?
        .run()
        .await
}

struct AppState {
    app_name: String,
}

#[get("/state")]
async fn state(state: web::Data<AppState>) -> String {
    // 从 app_data 中获取 AppState 实例
    format!("state: {}", state.app_name)
}
```

### 共享可变的State

不同的 worker 之间要想共享同一个可变 State，必须使用一个可共享的对象，实现了 Send + Sync。

```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // 共享状态
    let counter = web::Data::new(AppCounter {
        count: Mutex::new(0),
    });

    HttpServer::new(move || {
        App::new()
            // 将共享状态添加到应用程序中
            .app_data(counter.clone())
            .service(state)
    }).bind(("127.0.0.1", 8080))?
        .run()
        .await
}

struct AppCounter {
    count: Mutex<i32>,
}

#[get("/count")]
async fn state(state: web::Data<AppCounter>) -> String {
    // 访问共享状态
    let mut count = state.count.lock().unwrap();
    *count += 1;
    format!("count: {}", count)
}
```

### scope

使用 scope 可以限定一组 API 的 URL 前缀

```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        // 使用 scope 方法创建一个作用域
        let g1 = web::scope("/g1").service(hello).service(hey);
        let g2 = web::scope("/g2").service(hello).service(hey);
        // 将 g1 和 g2 作为服务添加到 App 中
        App::new().service(g1)
            .service(g2)
    }).bind(("127.0.0.1", 8080))?
        .run()
        .await
}

#[get("/hello")]
async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

#[get("/hey")]
async fn hey() -> impl Responder {
    HttpResponse::Ok().body("Hey there!")
}
```

### guard

使用 guard 可以过滤 Http 请求的Header

```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(
                web::scope("/")
                    .guard(guard::Host("www.rust-lang.org"))
                    .route("", web::to(|| async { HttpResponse::Ok().body("www") })),
            )
            .service(
                web::scope("/")
                    .guard(guard::Host("users.rust-lang.org"))
                    .route("", web::to(|| async { HttpResponse::Ok().body("user") })),
            )
            .route("/", web::to(HttpResponse::Ok))
    })
        .bind(("127.0.0.1", 8080))?
        .run()
        .await
}
```

## Server

### 多线程处理请求

默认情况下，HttpServer 创建和 CPU 核数一样的线程数。也可以自己指定。

```rust
#[actix_web::main]
async fn main() {
    HttpServer::new(|| App::new().route("/", web::get().to(HttpResponse::Ok)))
        .workers(4);
    // <- Start 4 workers
}
```

### 优雅退出

HttpServer 收到终止信号时，所有的 worker 线程有默认的30s来停止正在执行的请求。超过 shutdown_timeout 后，会强制停止。

## handler

处理 Http 请求的过程分为 2 个阶段：

1. 调用 handler 对象进行处理，返回一个实现了Responder接口的对象。
2. 调用 respond_to方法，将Responder转换为一个HttpResponse或Error。