# 实例化组件

`Config` 配方指向一个类及其构造函数的参数。

```yaml
class_path: physicalai.capture.UVCCamera
init_args:
  device: /dev/video0
  width: 640
  height: 480
```

## 从配方构建

`Config.instantiate()` 从可信的本地配方创建新对象。它只执行 `__init__`——你仍需自行调用 `connect()`、`run()` 等。

```python
from physicalai.config import Config

config = Config.from_dict({
    "class_path": "physicalai.capture.UVCCamera",
    "init_args": {"device": "/dev/video0", "width": 640, "height": 480},
})
camera = config.instantiate()
camera.connect()
```

## 保存对象的创建方式

给类加上 `@export_config`，Physical AI 就能记录你实际传入的构造函数参数（省略的默认值仍保持省略）。

```python
from physicalai.capture import UVCCamera
from physicalai.config import Config

camera = UVCCamera(device="/dev/video0", width=640, height=480)
config = Config.from_instance(camera)
```

嵌套组件在 `init_args` 内使用相同的配方结构。值必须对 JSON 友好：路径转为字符串，元组转为列表，无效浮点数会被拒绝。

```python
Config.from_instance(camera).save("camera.yaml")
```

## 安全

`class_path` 会在你的机器上加载并运行 Python 代码。请使用自己的配置文件或其他可信来源。对于来自其他进程或网络的元数据或消息，跳过实例化。

推理**清单**（策略导出）使用带别名和构建产物路径的独立格式。运行时构建 YAML 用 `Config`；加载导出的策略包时用清单 API。