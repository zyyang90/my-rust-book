# mdbook

mdbook 可以用 Markdown 文档创建书籍。

## Quick Start

1. 安装

```console
cargo install mdbook
```

2. 配置 book.toml

```toml
[rust]
edition = "2021"

[book]
# 书籍的作者
authors = ["zyyang90"]
# 书籍的语言
language = "zh-CN"
# 书籍的标题
title = "zyyang90's Rust Book"
# markdown 文档的根目录
src = "src"
```

3. 使用 serve 模式打开

```console
mdbook serve --open
```

## 给 mkbook 添加主题

1. 安装 mdbook-theme

```console
cargo install mdbook-theme
```

2. 在 book.toml 中添加

```toml
[preprocessor.theme]
pagetoc = true

# some variables related (defined in theme/css/variables.css)
# `content-max-width` + `pagetoc-width` = 95% seems the best
pagetoc-width = "30%"
content-max-width = "75%"
pagetoc-fontsize = "14.5px"
sidebar-width = "300px"
menu-bar-height = "40px"   # memu-bar = the bar on the top
page-padding = "15px"
mobile-content-max-width = "98%"

# layout
content-padding = "0 10px"
content-main-margin-left = "2%"
content-main-margin-right = "2%"
nav-chapters-max-width = "auto"
nav-chapters-min-width = "auto"
chapter-line-height = "2em"
section-line-height = "1.5em"

# modify some fontsizes
root-font-size = "70%"    # control the main font-size
body-font-size = "1.5rem"
code-font-size = "0.9em"
sidebar-font-size = "1em"    # sidebar = toc on the left

# modify some colors under ayu/coal/light/navy/rust theme
coal-inline-code-color = "#ffb454"
light-inline-code-color = "#F42C4C"
navy-inline-code-color = "#ffb454"
rust-inline-code-color = "#F42C4C"
light-links = "#1f1fff"
rust-links = "#1f1fff"

# if true, never read and touch the files in theme dir
turn-off = false

[output.html]
additional-css = ["theme/pagetoc.css"]
additional-js = ["theme/pagetoc.js"]
```