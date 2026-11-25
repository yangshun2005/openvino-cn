# 机器人

机器人实现结构化接口，因此无需继承。

```python
class MyRobot:
    joint_names = ["shoulder", "elbow", "wrist"]

    def connect(self) -> None: ...
    def disconnect(self) -> None: ...
    def get_observation(self) -> MyObservation: ...
    def send_action(self, action, *, goal_time: float = 0.1) -> None: ...
    def is_connected(self) -> bool: ...
```

## 观测

机器人观测至少应暴露以下字段。

```python
joint_positions: np.ndarray
timestamp: float
sensor_data: dict[str, np.ndarray] | None
images: dict[str, Frame] | None
```

## 动作契约

`send_action()` 接收单个动作向量，其顺序与 `joint_names` 一致。

```python
action.shape == (len(robot.joint_names),)
```

机器人实现负责在断开连接时将硬件置于安全状态。