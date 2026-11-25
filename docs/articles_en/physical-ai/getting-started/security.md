# 安全模型

运行时在设计上将几类输入视为可信，不再对其进行沙箱隔离。如果你加载的配置、清单或导出的策略并非由你编写或完整审查，那么该内容将以与 `physicalai` 进程本身相同的权限运行。以下各节描述每个信任边界。

## 配置和清单可执行任意代码

通过 `--config` 标志传入的 YAML/JSON 配置，以及导出策略的 `manifest.json`，都使用 `class_path` 值动态导入并构造 Python 对象——机器人、相机、预处理器、后处理器、动作源、回调。目前，没有任何机制限制 `class_path` 可以指向哪个类。

只加载来自可信来源的配置、导出策略和清单。另请参阅 [配置 Schema 参考](../reference/config-schema.md#security) 中的 `class_path` 说明。

## 只使用可信、经过审查的策略

导出的策略包（`manifest.json` 加构建产物）以与 `physicalai` 进程相同的权限运行。

**通过 `InferenceModel.from_pretrained()` 加载：** 将 `revision` 固定为你已审查并信任的版本的提交 SHA，而不是可变的分支或标签，这样每次运行加载的内容都恰好是你审查过的内容。

**通过 `export_dir` 加载：** `physicalai run` 和直接构造 `InferenceModel(export_dir=...)` 都会加载该本地目录中已有的任何包。只把经过审查、可信的导出放在那里。把填充该目录（下载、复制、解压）视为你决定信任其内容的时刻。

## 远程机器人共享没有内置安全控制

`SharedRobot` 网络传输（用于跨进程共享一个机器人连接）自身没有身份验证、访问控制或加密。

如果你启用 `allow_remote=True`（`physicalai robot serve`/`discover` 上的 `--allow_remote`），只应在隔离的、有防火墙的机器人单元网络（VLAN/防火墙）上使用，或自行配置 Zenoh ACL/TLS——这与 [CLI 参考](../reference/cli.md#physicalai-robot-serve) 中记录的要求相同。若不具备其中任一条件，任何能访问该网络的人都可以观察机器人状态，并且在无其他限制的情况下向机器人发送动作。

## 运行时回调以完全信任运行

注册到 `RobotRuntime` 的回调（参见[添加运行时回调](../how-to/runtime/add-runtime-callbacks.md)）可以在每个控制周期检查和修改发送给机器人的动作。目前，运行时在将回调输出发送到硬件之前不会对其进行验证。

只注册你编写或已审查的回调，尤其是任何能转换传出动作的回调。

## CLI 子命令从活动 Python 环境加载

`physicalai <subcommand>` 通过当前环境中已安装包注册的 Python 入口点发现第三方子命令。没有允许哪些包可以注册子命令的允许列表。

将安装包到同一环境视为授予其作为 `physicalai` 子命令运行的能力，并对其应用你对待任何其他依赖时相同的安全审查。