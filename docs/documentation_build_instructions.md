# 使用 CMake 构建文档

**注意**：以下步骤已在基于 Ubuntu 22.04 Docker 镜像的容器中测试通过。

1. 克隆 OpenVINO 仓库并初始化子模块
```
$ git clone <openvino_repository_url> <repository_path>
$ cd <repository_path>
$ git submodule update --init --recursive
```
2. 使用 OpenVINO 根目录下的 `install_build_dependencies.sh` 脚本安装构建依赖
```
$ chmod +x install_build_dependencies.sh
$ ./install_build_dependencies.sh
```
3. 安装构建文档所需的额外包
```
$ apt install -y doxygen graphviz texlive
```
4. 创建 Python 虚拟环境并安装所需库
```
$ python3 -m venv env
$ source env/bin/activate
(env) $ pip install --upgrade setuptools pip
(env) $ pip install -r docs/requirements.txt
```
5. 安装 sphinx 主题
```
(env) $ python -m pip install docs/openvino_sphinx_theme
```
6. 安装自定义 sphinx sitemap
```
(env) $ pip install docs/openvino_custom_sphinx_sitemap
```
7. 创建构建目录：
```
(env) $ mkdir build && cd build
```
8. 使用以下命令构建文档：
```
(env) $ cmake .. -DENABLE_DOCS=ON
(env) $ cmake --build . --target sphinx_docs
```
可根据需要，在第一次 cmake 调用时添加以下变量：
- 构建 C/C++ API：`-DENABLE_CPP_API=ON`
- 构建 Python API：`-DENABLE_PYTHON_API=ON`
- 构建 GenAI API：`-DENABLE_GENAI_API=ON`
- 构建 Notebooks：`-DENABLE_NOTEBOOKS=ON`
- 构建 OVMS：`-DENABLE_OVMS=ON -DOVMS_DOCS_DIR=<path_to_OVMS_repo>`

> **注意：** 构建 Python API 文档时，会通过[安装脚本](./scripts/install_appropriate_openvino_version.py)自动下载 [conf.py](./sphinx_setup/conf.py) 中指定的 OpenVINO 包版本。
> 如需特定版本或使用已安装的 OpenVINO 包，请相应修改这些文件。