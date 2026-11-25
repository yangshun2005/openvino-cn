# 跨进程共享机器人

`SharedRobot` 让一个进程独占机器人的硬件连接，其他任意数量的进程可通过 [Zenoh](https://zenoh.io/) 读取其状态并发送动作。它满足与直接驱动相同的 `Robot` 协议，因此可即插即用地替换任何需要机器人的位置（包括 `RobotRuntime`）。

## 安装

```bash
pip install "physicalai[transport]"
```

## 创建或附加

每个 `SharedRobot` 都有一个必填的、由调用方指定的逻辑 `name`——它直接作为 Zenoh 主题的键。对于给定 `name`，第一个构造的 `SharedRobot` 若未发现已有所有者，就会创建一个（在分离的子进程中）；后续实例（同进程或不同进程，相同 `name`）则附加到它。

构造使用 `robot=` 或 `from_config()`。若已有配方，优先用 `from_config()`。未连接的 `@export_config` 驱动可直接传给构造函数，或显式导出：

```python
import numpy as np
from physicalai.config import Config
from physicalai.robot import SO101, SharedRobot

driver = SO101(
    port="/dev/ttyUSB0",
    calibration="~/.cache/calibration/so101.json",  # 路径保持相对/原样
)
robot = SharedRobot.from_config(driver, name="left-arm")
# 等价于：SharedRobot.from_config(Config.from_instance(driver), name="left-arm")
# 或：SharedRobot("left-arm", robot={"class_path": "physicalai.robot.SO101", "init_args": {...}})
robot.connect()

obs = robot.get_observation()          # 拉取最新状态，非阻塞
robot.send_action(np.asarray(obs.joint_positions), goal_time=0.1)

robot.disconnect()                     # 分离；所有者继续运行
```

任何可导入的 `@export_config` 机器人类都可用（包括第三方插件）——传入其公开的 `class_path` + `init_args`；没有扁平的 `robot_class` / `robot_kwargs` API。

## 在前台提供机器人服务

当由 shell、systemd、Docker 或 Kubernetes 管理机器人生命周期时，使用运维命令：

```bash
physicalai robot serve --config examples/so101/serve.yaml
```

该命令在自己的前台进程中构造并连接驱动。正常输出会报告就绪状态、状态订阅者增减、每 30 秒一次的健康摘要，以及干净关闭。加 `--verbose` 查看启动和清理细节。该命令不会守护化；后台监管请使用你的服务管理器。

列出可达所有者，无需导入其声明的驱动类：

```bash
physicalai robot discover
physicalai robot discover --json
physicalai robot discover --allow_remote
```

除非显式指定 `--allow_remote`，发现仅限本地。人类可读输出是排序的 ASCII 表格。JSON 模式向 stdout 写入一个排序数组，无机器人响应时包含 `[]`。

注意：

- `get_observation()` 返回所有者发布的最新状态；若自上次调用以来没有新样本到达，则返回缓存的最后已知观测。陈旧程度可通过 `obs.timestamp` 查看。
- `obs.state` 是**所有者计算**的状态向量（例如 WidowXAI 提供位置 + 速度，共 14 个值），因此订阅端无需机器人特定逻辑即可获得正确的模型输入。
- `obs.images` 始终为 `None`——相机帧走 `SharedCamera`（采集传输），不走机器人传输。
- 动作是**绝对关节目标**，最新优先、即发即忘。无待处理动作时，所有者保持最后指令位置。

## 附加到已知所有者

对于手动启动或远程所有者，仅按名称附加——无需构造配方：

```python
from physicalai.robot import SharedRobot

robot = SharedRobot.attach("left-arm")
robot.connect()
```

枚举可达机器人：

```python
from physicalai.robot.transport import discover_robots

for metadata in discover_robots():
    print(metadata["name"], metadata["robot_class"], metadata["joint_names"])
```

## 复用远程客户端

对于反复发现或附加远程所有者的长运行进程，使用 `SharedRobotClient(allow_remote=True)`。它保持一个 Zenoh 发现会话打开，首次操作建立远程路由，后续操作复用：

```python
from physicalai.robot.transport import SharedRobotClient

with SharedRobotClient(allow_remote=True) as client:
  robots = client.discover()
  robot = client.attach(robots[0]["name"])
  robot.connect()

  # 后续发现复用已建立的远程会话。
  robots = client.discover()
```

`SharedRobotClient` 仅用于附加：它从不启动所有者进程。上下文退出时，它会断开通过 `attach()` 创建的所有机器人，再关闭共享会话。首次 `discover()` 调用为 Zenoh 发现使用一秒预算；后续调用使用已预热会话的 0.1 秒。传 `timeout=` 可覆盖任一预算。超时仍是通配查询收集窗口，因此网络或所有者变更后需要更完整清单时，请使用更大的显式值。`SharedRobotClient()` 也适用于同主机所有者；省略 `allow_remote=True` 可使该客户端仅限本地。

## 网络范围：默认仅本地

`allow_remote=False`（默认）使所有者的 Zenoh 会话在主机外不可达——多播/gossip 发现被禁用，所有者仅监听 `127.0.0.1`。同主机的创建或附加仍可工作，不依赖多播：所有者和订阅者从 `name` 推导出相同的确定性回环端口。

需要时显式启用跨主机可达：

```python
robot = SharedRobot.from_config(
    {
        "class_path": "physicalai.robot.SO101",
        "init_args": {"port": "/dev/ttyUSB0", "calibration": "calibration.json"},
    },
    name="left-arm",
    allow_remote=True,
)
```

每个调用方有自己的 Zenoh 会话。对于附加者，`allow_remote` 仅控制该会话发现远程所有者的能力。对于创建所有者的调用方，它还固定了所有者整个生命周期的可达性。后续附加者无法更改现有所有者的范围。参见下文[安全](#security-trusted-network-required)。

远程所有者在其 `/metadata` 响应中省略物理 `device_ids`。其他发现信息，包括逻辑名称、驱动类、主机、关节布局和状态维度，仍对可达对等方可见。

## 物理设备标识 vs. 逻辑名称

`name` 是你选择的、作为 Zenoh 主题键的名称——解析它从不需要构造驱动。物理设备标识（`Robot.device_ids`，例如 `("serial:ttyUSB0",)`）是另一回事，由_所有者_用于强制主机本地独占：两个不同的 `name` 不能同时占用同一物理设备（`RobotDeviceAlreadyOwned`），而对_同一_ `name` 使用_不同_设备的竞争会被拒绝（`RobotNameConflict`），而不是由先启动的进程静默选中。

现有所有者声明的 `robot_class` 仅作为诊断与你的比较（不匹配时记录日志，从不致命，也从不从网络导入）——子类、包装器和重新导出都可保持线上契约。

## 所有者生命周期

- 每个 `name` 一个所有者进程，每个物理设备一个所有者——均由主机本地、崩溃安全的 `flock` 锁强制（Linux 下位于 `$XDG_RUNTIME_DIR/physicalai/robot-locks/`，其他 Unix 系统下位于每用户私有临时目录）。若两个进程竞争创建同一 `name`，失败者附加（相同设备）或抛出 `RobotNameConflict`（不同设备）。
- 所有者以固定速率运行单个写优先控制循环（`rate_hz` 创建参数，默认 100 Hz；当硬件测量支持不同值时可按实例覆盖）。
- 最后一个订阅者断开（正常或崩溃）后，所有者等待 `idle_timeout` 秒，然后调用驱动的 `disconnect()`——遵守安全状态契约（保持/归位）——并退出。
- 显式 `physicalai robot serve` 所有者没有空闲超时。它保持前台直到被中断或所有者循环失败。
- 订阅者的 `disconnect()` 从不停止机器人电机；安全状态由所有者负责。
- 订阅者在声明动作发布者之前，会拒绝声明不支持传输协议版本的所有者。

## 安全：需要可信网络

在 `allow_remote=True` 模式下，此传输对在其 Zenoh `/action` 键上收到的任何动作**无认证、无加密**地应用——任何能到达所有者 Zenoh 会话的对等方都能移动物理机器人。该模式专为**可信机器人单元网络**（隔离的 LAN/VLAN）设计。隔离网络——通过 VLAN/防火墙分段或 Zenoh 自身的 ACL/TLS 功能——是部署者的责任。`allow_remote=False`（默认）通过使所有者主机外不可达，完全避免此暴露。