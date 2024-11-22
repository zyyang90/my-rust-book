# 加快 Rust 的编译速度
## 更新 rust 版本
Rust 官方在不断的优化编译器，提高编译速度，因此，升级你的 rust 版本。
```console
# 升级当前 toolchain
rustup update

# 查看 cargo 版本
cargo -V

# 查看当前环境的 toochain
rustup toolchain list

# 安装
rustup install nightly

# 设置 nightly 为默认
rustup default nightly

# rustc 版本
rustc -V
```

## 使用 cargo check 而非 cargo build
如果只想检查代码是否存在错误，是不必进行完整的编译过程的。所以，使用 cargo check可以快速的检查代码的语法、类型和借用。cargo check 不会生成二进制文件。
```console
# 检查
cargo check

# 简写
cargo c
```

## 使用 cargo watch -c
当使用 cargo watch -c 它是可以自动在代码发生变化时进行代码检查，这样你就可以更快地发现错误并进行及时修复。实际测试，我感觉这个watch 的功能不是特别实用。
```console
# 安装
cargo install cargo-watch

# Run check then tests
cargo watch -x check -x test
```

## 检查并删除没有用的依赖项
machete 可以检查项目中的依赖，并输出哪些依赖是无用的，减少构建时间和资源消耗及减小项目体积。
```console
# 安装 machete
cargo install cargo-machete

# 检查无用的依赖
cargo machete
```

## 找出编译中耗时的 crate
执行`cargo build`时，添加`--timings`参数，会统计编译期间的耗时，并生成统计报告。
```console
# 在 target/timings 下生成耗时的统计
cargo build --timings
```

## 查看编译器的统计信息
cargo rustc 命令用于在构建过程中传递自定义的编译器标志。它允许你直接调用 rustc 编译器，并传递特定的选项。例如：
```console
# 在 release 模式下编译代码，并将优化级别设置为 3
cargo rustc --release -- -C opt-level=3
```
运行下面的命令，可以编译当前项目，并生成 Rust 编译过程的详细信息。
```console
# 生成 taosx-xxxx.mm_profdata 文件
cargo rustc -- -Zself-profile
```
mm_profdata 文件需要用 summarize 工具打开
```console
# 安装 summarize
cargo install --git https://github.com/rust-lang/measureme --branch stable summarize

# 分析
summarize summarize ./taosx-0088944.mm_profdata
```

## 使用 sccache
sccache 是一个类似 ccache 的编译器缓存工具。它用作编译器包装器，并在可能的情况下避免编译，将缓存结果存储在本地磁盘或云存储中。
```console
# 安装
cargo install sccache

# 编译，启动 sccache
RUSTC_WRAPPER=sccache cargo build

# 查看 sccache
sccache --show-stats
```
每次为 cargo build 传递环境变量很麻烦，可以使用 direnv 这种工具。direnv 可以为当前目录添加一个 hook。每当打开 shell 时，自动生效设置的环境变量。
```console
# 安装 direnv
curl -sfL https://direnv.net/install.sh | bash

# 将 direnv 添加到 zsh 的环境变量中，这样只要开启 zsh，direnv hook 就能生效
vim ~/.zshrc

# 在 .zshrc 中添加如下内容

# direnv START
eval "$(direnv hook zsh)"
# direnv END
```
为当前目录设置环境变量
```console
cd /Users/yangzy/RustProjects/taosx

# direnv 允许为当前路径添加 direnv hook
direnv allow .

# 将环境变量添加到 .envrc 中
echo export RUSTC_WRAPPER=sccache > .envrc

# 查看环境变量是否生效
echo $RUSTC_WRAPPER
```

## 用 nextest 加快单元测试
cargo-nexttest 号称比 cargo test 快 3 倍，在单元测试时会更快。同时，nextest 的输出也更加简洁。
```console
# 安装 nextest
cargo install cargo-nextest

# 查看版本
cargo nextest --version

# 运行所有 test
cargo nextest run
```

## 更好的硬件
换个更好的电脑～

## 优化 openssl 编译
如果 mac 本地安装了 openssl，在编译 openssl 时，会自动尝试检测 openssl。
```console
# brew 检查是否安装了 openssl
brew list -l | grep openssl
```
如果 vendored 特性被配置了，openssl-src crate 将用于编译并静态链接到 OpenSSL 的副本。因此，我们本地已经有 openssl 库了，可以修改 Cargo.toml，去掉 vendored 特性，以加快编译。
编辑 taosx-core/Cargo.toml:
```toml
[target.'cfg(all(unix, not(target_arch = "loongarch64")))'.dependencies]
openssl = { version = "0.10.55" }
#openssl = { version = "0.10.55", features = ["vendored"] }
 
 [target.'cfg(target_arch="loongarch64")'.dependencies]
 openssl = { version = "0.10.55" }
```

## 优化 rdkafka 编译
rdkafka 如果使用了 cmake-build 特性，会尝试使用 sasl2-src 源码编译，为了加快本地编译速度。可以使用 dynamic-linking 特性，即使用本地的 librdkafka。
首先，需要确定本地是否安装了 librdkafka。
```console
# brew 查是否安装了 librdkafka
brew list -l | grep librdkafka

# otool 等效 ldconfig -p
 otool -L /usr/local/lib/librdkafka.dylib 
 
 # ll 
 ll /usr/local/lib/librdkafka.dylib
```
dynamic-linking 要求，librdkafka 的系统版本必须与此 crate 捆绑的 librdkafka 版本完全匹配。我实际测试了一下，用 brew install 的办法安装不上，最后选择本地编译 librdkafka。
先在 github 找到 rdkafka 对应的版本 tag，从仓库可以看到对应的 librdkafka 的 commit id，从 librdkafka 仓库拉取代码后，check 到对应的 commit id，然后编译。
```console
./configure && make && sudo make install
```
成功后，librdkafka 会安装在 /usr/local/lib/librdkafka.dylib。
之后，编辑 taosx-core/Cargo.toml，修改 rdkafka 的特性为 dynamic-linking
```toml
rdkafka = { version = "0.36.2", features = ["dynamic-linking"] }
```