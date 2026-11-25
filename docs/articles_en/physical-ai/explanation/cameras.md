# 相机

相机提供一个小型采集接口，用于连接设备并获取帧。

```python
camera.connect()
frame = camera.read_latest()
camera.disconnect()
```

## 读取模式

| 方法            | 行为                          | 用途                     |
| --------------- | ----------------------------- | ------------------------ |
| `read()`        | 下一帧，阻塞                  | 录制或完整帧流           |
| `read_latest()` | 最新帧，非阻塞                | 实时控制                 |
| `async_read()`  | `read()` 的异步封装           | 异步应用                 |

## 运行时使用

控制循环通常更关心帧的新鲜度，而非完整性。

```python
observation["image.wrist"] = wrist_camera.read_latest()
```

相机实例不是线程安全的。每个相机实例使用一个线程，或添加外部同步。

## 设备标识

`discover()` 会报告每个相机的 `hardware_payload`：后端特定的标识字段（序列号、传感器端口、总线路径等），不保证键名，也不保证在重启或重新插拔后保持稳定——其结构因后端和设备而异。要重新打开特定相机，将其 `hardware_payload`（或其他标识字符串，例如序列号）作为 `device_id` 传回后端；后端会解析出最匹配的设备，若多个设备匹配程度相同则抛出异常。仅当你确认设备报告的是持久信息（例如真实的 USB 序列号）时，才持久化 `hardware_payload`。

## SharedCamera

`SharedCamera` 让一个发布者子进程独占相机的硬件连接，同时任意数量的订阅者通过 iceoryx2 读取帧。它满足与直接驱动相同的 `Camera` 协议。

构造使用 `camera=` 或 `from_config()`，与 `SharedRobot` 一致。共享时优先使用 `from_config()`（或 YAML）。已断开的 `@export_config` 相机可直接传给 `camera=`；这会导出其配置配方，而不是将已打开的设备交给子进程。发布者始终重新打开设备。共享时切勿让直接相机保持连接到同一设备：另一个已连接的持有者会导致打开失败。

```python
from physicalai.capture import SharedCamera
from physicalai.config import Config

# 优先使用仅配置 / 已断开的导出——不保持直接相机连接
shared = SharedCamera.from_config(
    {
        "class_path": "physicalai.capture.UVCCamera",
        "init_args": {"device": "/dev/video0", "backend": "v4l2"},
    },
)
# 从已断开的实时驱动等效写法：
# shared = SharedCamera.from_config(Config.from_instance(driver))
shared.connect()

# 可安全地从多个线程/进程读取
frame = shared.read_latest()
```

`create_camera(..., shared=True)` 仍是可共享内置相机（`uvc`、`realsense`、`basler`、`ip`）的便捷方式，它将类型打包进 `SharedCamera(camera=...)`。

`from_config()` 会为任意 `class_path`（包括第三方驱动）推导 `service_name` 为 `physicalai/camera/<ClassName>/<device_id>/frame`。若两个相机类共享类名和设备 ID，请显式传入 `service_name=`。

`SharedCamera` 是生产部署中多个消费者需要相机帧时的推荐方案。它免去手动同步的需要，并高效处理帧分发。