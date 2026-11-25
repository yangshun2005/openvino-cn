# OpenVINO™ Python* 开发工具

## 概述
OpenVINO 包含以下工具：
* openvino.tools.benchmark

详情请参阅 https://docs.openvino.ai。

## 安装

### 前提条件

先安装前提条件：

#### 1. Python

安装 **Python** 前提条件：

- [Python3][python3]
- [setuptools][setuptools]

在 Ubuntu* 上运行以下命令安装这些前提条件：
```bash
sudo apt-get install python3 python3-dev python3-setuptools python3-pip
```

Python setuptools 和 Python 包管理器（pip）默认将包安装到系统目录。有以下几种选择：

- 在[虚拟环境][virtualenv]中工作（最佳方案）。
- 为所有 `pip` 命令使用 `--user` 选项。
- 以 *sudo* 权限安装所有依赖。

要使用虚拟环境，需先安装：

```bash
python3 -m pip install virtualenv
python3 -m virtualenv -p `which python3` <directory_for_environment>
```

在虚拟环境中开始工作前，需先激活：

```bash
source <directory_for_environment>/bin/activate
```

可使用以下命令停用虚拟环境：

```bash
deactivate
```

#### 2. 安装包

可以在 `pip install` 命令中指定工具的 `setup.py` 路径来安装工具：

```bash
python3 -m pip install <tools_folder>/
```
例如，安装 Benchmark Tool，使用以下命令：
```bash
python3 -m pip install benchmark_tool/
  ```

### 配置

每个子包有各自的配置。详情请参阅各子包的文档。

[python3]: https://www.python.org/downloads/
[setuptools]: https://pypi.python.org/pypi/setuptools