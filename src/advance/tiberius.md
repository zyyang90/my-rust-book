# tiberius

tiberius 是一个原生的 Microsoft SQLServer （TDS） 客户端。tiberus 是一个来自德国公司 prisma 开发维护的 crate。tiberius 实现了
SQL Server 的 TDS 协议。TDS（Tabular Data Stream） 协议是客户端用于连接到 SQL Server 的应用程序层协议。

## 创建连接

```rust
#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let mut config = Config::new();
    config.host("192.168.3.40");
    config.port(1433);
    config.authentication(AuthMethod::sql_server("aaAdmin", "aaAdmin"));
    config.trust_cert();

    let tcp = TcpStream::connect(config.get_addr()).await?;
    tcp.set_nodelay(true)?;
    let mut client = Client::connect(config, tcp.compat_write()).await?;
}
```

注意：

1. 要开放 SQL Server 的连接，可以使用 SQL Server 的验证连接
2. tiberius 默认使用 native-tls，链接到操作系统的 TLS 库。在 mac 上需要`vendored-openssl`，配置如下：

```toml
tiberius = { version = "0.12.2", default-features = false, features = ["tds73", "winauth", "vendored-openssl"] }
```