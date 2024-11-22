# Cargo 下载慢，使用国内镜像
1. 设置 Rustup 镜像， 修改配置 ~/.zshrc or ~/.bashrc
```console
export RUSTUP_DIST_SERVER="https://rsproxy.cn"
export RUSTUP_UPDATE_ROOT="https://rsproxy.cn/rustup"
```

2. 配置～/.cargo/config
```toml
[source.crates-io]
replace-with = 'rsproxy-sparse'

[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"

[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"

[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"

[net]
git-fetch-with-cli = true 
```

参考：https://rsproxy.cn/