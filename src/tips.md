# 查看 sqlite 数据文件

```console
# 在 taosx 的数据目录下
sqlite3 taosx.dev.db

# 查看所有表
.tables

# 查询 task
select * from tasks;
```

# nextest 运行某些用例

```console
# 运行 taosx-core 的所有 case
cargo nextest run --package taosx-core

# 运行 taosx-core 的 case，失败不立即退出
cargo nextest run --no-fail-fast --package taosx-core

# 运行 taosx-core 的 case，除了以 _with_taos 结尾的 case
cargo nextest run --no-fail-fast --package taosx-core -E "not test(/_with_taos$/)"
```

# 检查 commit message format

1. 配置pre-commit.yaml

```yaml
repos:
  - repo: local
    hooks:
      - id: check-commit-msg
        name: check commit message format
        entry: ./scripts/check_commit_msg.sh
        language: script
        stages: [ commit-msg ]
```

注意，这个check 使用了 commit-msg hook，需要在pre-commit 中安装。

```console
pre-commit install --hook-type commit-msg
```

2. 在 scripts 下，添加脚本

```bash
#!/bin/bash

# 读取信息
commit_msg_file="$1"
commit_msg=$(<"$commit_msg_file")

# 定义合法的类型
valid_types="fix|feat|doc|docs|perf|build|ref|refactor|test"

# 正则表达式检查
if ! [[ "$commit_msg" =~ ^($valid_types)\([a-zA-Z0-9_-]+\):\ (.+)(\#[a-zA-Z0-9_-]+)$ ]]; then
  echo "ERROR: Commit message must be in the format 'type(scope): subject' and can optionally include a '#subject' at the end."
  exit 1
fi 
```

# cargo clippy 自动优化

直接在代码上应用 clippy 的优化，不需要手动修改：

```console
cargo clippy --fix --no-deps --allow-dirty --allow-staged -p taosx-core
```

参数：

- --fix：应用修复
- --no-deps：依赖包不检查
- --allow-dirty：没有 git add 在 dirty 区的文件也会应用 fix
- --allow-staged：没有 git commit 在 stage 区的文件也会应用 fix
- -p（--package）：指定包

# 用 perf 生成火焰图

1. 打点

```console
perf record -F 99 -a -g -p `pidof taosx`
```

2. 生成火焰图

```console
# 下载 FlameGraph
git clone https://github.com/brendangregg/FlameGraph.git

# 生成火焰图
perf script -i perf.data &> out.perf

./stackcollapse-perf.pl out.perf > out.folded
./flamegraph.pl out.folded > out.svg
```

# 用 perf top 查看线程的运行情况

```console
perf top -gp `pidof taosx`
```

# 用 git subtree 将外部仓库添加到当前仓库
1. 在 taosx 仓库中新建一个分支
```console
git checkout -b enh/td-31475
```
2. 将 

