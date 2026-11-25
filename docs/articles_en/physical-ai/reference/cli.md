# CLI 参考

运行时 CLI 命令与 Python API 使用相同的 Schema。

标准命令是 `physicalai`。同时安装了简写别名 `pai`，行为完全一致。

## `physicalai run`

```bash
physicalai run --config runtime.yaml [--run.duration_s=60]
```

参数：

| 参数               | 必填 | 说明                         |
| ------------------ | ---- | ---------------------------- |
| `--config`         | 是   | 运行时配置 YAML              |
| `--run.duration_s` | 否   | 达到指定秒数后停止           |

直接构造运行时后，当前 Python API 也支持相同的时长限制。

```python
runtime = RobotRuntime(...)

with runtime:
    runtime.run(duration_s=60)
```

## `physicalai robot serve`

```bash
physicalai robot serve --config examples/so101/serve.yaml
```

YAML 字段包括 `name`、必填的 `robot`（一个含 `class_path` + `init_args` 的 `Config`）、可选的 `allow_remote`（默认 `false`）以及可选的正数 `rate_hz`（默认 100 Hz）。直接使用 CLI 参数时名称相同，包括嵌套的机器人构造值，如 `--robot.init_args.port /dev/ttyACM0`。不支持扁平的 `robot_class` / `robot_kwargs`。服务在前台运行，直到收到 SIGINT 或 SIGTERM；启动、循环、重复读取或断连失败时返回非零值。

使用 `--verbose` 可输出驱动构造、锁获取、初始观测、端点声明和清理的详细信息。

`--allow_remote` 会暴露一个未认证的物理动作端点。仅在隔离的机器人单元 VLAN/防火墙内，或配合 Zenoh ACL/TLS 使用。

## `physicalai robot discover`

```bash
physicalai robot discover [--allow_remote] [--timeout 2] [--json]
```

结果按机器人名称和主机排序。人类可读输出为 ASCII 表格，包含名称、类、主机和关节数，随后是结果数量和发现耗时。JSON 模式向 stdout 写入恰好一个数组；发现为空也算成功，写入 `[]`。

## Shell 补全

Shell 补全脚本可直接从 CLI 打印并在 Shell 中 source。补全包含所有已安装插件的子命令，例如 studio 训练命令。

```bash
source <(physicalai completion zsh)
source <(pai completion bash)
```

## 插件命令

训练包可通过 entry points 添加命令。

```toml
[project.entry-points."physicalai.cli.subcommands"]
fit = "physicalai.cli.fit:register"
validate = "physicalai.cli.validate:register"
test = "physicalai.cli.test:register"
predict = "physicalai.cli.predict:register"
benchmark = "physicalai.cli.benchmark:register"
export = "physicalai.cli.export:register"
```