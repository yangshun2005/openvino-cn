# 使用执行模式

`Execution` 组件决定推理在哪里运行，以及请求如何调度。

## 同步

此模式在运行时线程中执行推理。

```python
from physicalai.runtime import SyncExecution

execution = SyncExecution()
```

此模式适用于简单部署和调试。

## 线程工作器

此模式在后台线程中执行推理。

```python
from physicalai.runtime import AsyncExecution

execution = AsyncExecution(fps=30)
```

当模型延迟不应阻塞机器人时序时，使用此模式。由于推理后端通常会释放 GIL，基于线程的执行在大多数场景下都能良好工作。

## 远程

> **预览：** `RemoteExecution` 尚未实现。

此模式将推理请求发送到策略服务器。

```yaml
execution:
  class_path: physicalai.runtime.RemoteExecution
  init_args:
    endpoint: http://robot-server:8080
```

当机器人主机不应持有策略权重或加速器依赖时，使用此模式。