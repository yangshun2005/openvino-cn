# Robot API 参考

机器人实现需满足 `Robot` 协议。

```python
class Robot(Protocol):
    def connect(self) -> None: ...
    def disconnect(self) -> None: ...
    def get_observation(self) -> RobotObservation: ...
    def send_action(self, action: np.ndarray, *, goal_time: float = 0.1) -> None: ...
    def is_connected(self) -> bool: ...

    @property
    def joint_names(self) -> list[str]: ...
```

## `RobotObservation`

```python
class RobotObservation(Protocol):
    joint_positions: np.ndarray
    timestamp: float
    sensor_data: dict[str, np.ndarray] | None
    images: dict[str, Frame] | None
```

## 要求

- `joint_positions` 的顺序必须与 `joint_names` 一致。
- 传给 `send_action()` 的动作必须与 `joint_names` 一致。
- `disconnect()` 必须让硬件处于安全的静止状态。
- `connect()` 应幂等，或失败时给出明确错误。