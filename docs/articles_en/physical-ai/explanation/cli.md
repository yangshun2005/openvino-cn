# CLI

CLI 是 Python 所用同一套配置 API 的轻量封装。

标准命令是 `physicalai`，同时安装了行为完全相同的简写别名 `pai`。

```bash
physicalai run --config runtime.yaml --run.duration_s=60
```

等价的 Python 控制流：

```python
runtime = RobotRuntime(...)

with runtime:
    runtime.run(duration_s=60)
```

Shell 补全通过 `jsonargparse` + `shtab` 提供。用 `completion` 命令打印补全脚本，然后在 shell 中 source 或安装。安装训练包后，其插件子命令会自动纳入补全。

```bash
source <(physicalai completion zsh)
source <(pai completion zsh)
```

## 运行时命令

| 命令                        | 用途                                             |
| --------------------------- | ------------------------------------------------ |
| `physicalai run`            | 在机器人硬件上运行训练好的策略（或任意动作源）。 |
| `physicalai robot serve`    | 由前台属主进程提供一台共享机器人。               |
| `physicalai robot discover` | 列出可访问的共享机器人属主。                     |

## 训练命令

训练命令应来自训练包或入口点插件。

```toml
[project.entry-points."physicalai.cli.subcommands"]
fit = "physicalai.cli.fit:register"
benchmark = "physicalai.cli.benchmark:register"
export = "physicalai.cli.export:register"
```

导入 `physicalai` 不应引入训练依赖。