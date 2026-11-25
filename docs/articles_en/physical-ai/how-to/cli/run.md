# CLI：运行

使用此命令从 YAML 文件运行机器人运行时。

```bash
physicalai run --config runtime.yaml
```

可按需限制运行时长。

```bash
physicalai run --config runtime.yaml --run.duration_s=60
```

直接构造运行时后，当前 Python API 也支持相同的时长限制。

```python
from physicalai.runtime import RobotRuntime

runtime = RobotRuntime(...)

with runtime:
    runtime.run(duration_s=60)
```

运行时命令位于 `physicalai` 包中。训练命令应由训练包或插件入口点提供，以保持运行时包轻量。