# Camera API 参考

Camera 实现应继承自 `Camera`。

```python
class Camera:
    def connect(self, timeout: float = 5.0) -> None: ...
    def disconnect(self) -> None: ...
    def read(self, timeout: float = 2.0) -> Frame: ...
    def read_latest(self) -> Frame: ...
    async def async_read(self, timeout: float = 2.0) -> Frame: ...

    @property
    def is_connected(self) -> bool: ...

    @property
    def device_id(self) -> str: ...
```

## 颜色模式

```python
class ColorMode(StrEnum):
    RGB = "rgb"
    BGR = "bgr"
    GRAY = "gray"
```

## Camera 类型

```python
class CameraType(StrEnum):
    UVC = "uvc"
    IP = "ip"
    REALSENSE = "realsense"
    GENICAM = "genicam"
    BASLER = "basler"
```

## 读取语义

| 方法            | 语义                                                           |
| --------------- | -------------------------------------------------------------- |
| `read()`        | 返回下一帧，阻塞，保持帧序。                                   |
| `read_latest()` | 返回最新帧，不阻塞，可能跳帧。                                 |
| `async_read()`  | 为 `read()` 提供异步封装。                                     |

Camera 实例不是线程安全的。