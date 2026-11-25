# 配置

Physical AI 用简单、可重复的方式描述配置：**选一个 Python 类**，**传入其构造函数所需的参数**。同一份描述可以存在于 Python、YAML、CLI 或 Studio 中。

## 配置配方

配置配方指定类及其构造函数参数：

```yaml
class_path: physicalai.capture.UVCCamera
init_args:
  device: /dev/v4l/by-id/usb-Example_Camera-video-index0
  width: 640
  height: 480
```

- `class_path` — 类的导入路径（例如某个相机驱动）。
- `init_args` — 传给 `__init__` 的关键字参数。

需要通过 YAML 往返的类可以用 `@export_config` 选择加入。之后 `Config.from_instance(live_object)` 会记录对象的创建方式：你传入了哪些参数，不包含已打开的设备、套接字或其他运行时状态。

```text
live object -> Config.from_instance() -> YAML/JSON -> config.instantiate() -> new object
```

`Config.instantiate()` 只运行构造函数。你的应用仍需在需要时调用 `connect()` 或 `run()` 等方法。

## 工作流文件（`physicalai run`）

运行配置在同一配方外增加结构。例如，`physicalai run` 要求运行时设置放在 `runtime:` 下，可选的运行选项放在 `run:` 下。你也可以直接传入导出的 `RobotRuntime` 配方；加载器会将其重塑为该形式。

嵌套部分（机器人、策略、相机、回调）使用相同的 `class_path` + `init_args` 模式，必要时嵌套在 `init_args` 内。

## 推理清单

导出的策略文件夹包含一个**清单**：文件路径、预处理器、运行器和兼容性元数据。该格式面向加载训练好的包。机器人和运行时 YAML 通常改用 `Config` 配方。

描述**如何运行**某物时使用工作流 YAML。描述**导出内容里有什么**时使用清单。

## 训练与类型化设置

当你已知 Python 类型（训练器、dataclass、CLI 模型）时，用 **jsonargparse** 构建：为该类型定义解析器，加载 YAML 或 CLI 值，然后调用 `instantiate()`。

```python
parser = ArgumentParser(exit_on_error=False)
parser.add_class_arguments(Trainer, "trainer")
parsed = parser.parse_object(document, defaults=False)
trainer = parser.instantiate(parsed).trainer
```

`physicalai.config.Config` 是可移植 YAML 的共享配方格式：`class_path` 加 `init_args`。当活动组件需要保存到磁盘并在之后重建时，配合 `@export_config` 使用。

## 安全

加载配置会从 `class_path` 导入 Python 模块。只使用你自己编写的或在该机器上信任的文件。不要用从网络收到的机器人或相机元数据构建对象。