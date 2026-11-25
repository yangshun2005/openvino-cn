# 自定义 GitHub Actions

专为 OpenVINO 工作流编写的自定义 Action 及其源代码位于 [actions](../../../../.github/actions) 文件夹中。

更多信息请参阅官方[自定义 GitHub Action 文档](https://docs.github.com/en/actions/creating-actions/about-custom-actions)。

## 可用的自定义 Action

* [Python 环境安装](#python-环境安装)
* [系统信息打印](#系统信息打印)
* Smart CI（详见：[功能文档](./smart_ci.md)）
* OpenVINO Provider（详见：[文档](./openvino_provider.md)）
* Docker 镜像处理（详见：[Docker 镜像文档](./docker_images.md)）
* 构建产物存储与恢复（`store_artifacts`、`restore_artifacts`）
* 依赖与编译器缓存管理（`cache`）
* Wheel 安装（`install_ov_wheels`）
* Manifest 创建（`create_manifest`）
* 覆盖率收集（`coverage_toolkit`）
* 等待检查完成（`wait-for-check-completion`）

## Python 环境安装

此自定义 Action 在 runner 上安装所需的 Python 版本并设置环境变量。
底层根据机器架构使用 GitHub 提供的 `actions/setup-python` 或社区提供的 `deadsnakes-action`。
由于 `actions/setup-python` 在 Linux ARM64 机器上无法工作，因此改用 `deadsnakes-action`。

在 Windows x64 上，Python 3.15 和 3.15t 使用官方 CPython NuGet 包（固定为 3.15.2），以避免使用全机器范围的安装程序。该 Action 将所选解释器添加到 `PATH`，并导出 `PYTHON315_EXECUTABLE` 或 `PYTHON315T_EXECUTABLE` 供需要显式路径的构建使用；这些安装不会注册到 `py`。

### 用法
```yaml
  - name: Setup Python ${{ env.PYTHON_VERSION }}
    uses: ./openvino/.github/actions/setup_python
    with:
      version: '3.11'
      use-pip-proxy: 'true'
      set-pip-install-path: 'true'
      self-hosted-runner: 'true'
```
其中：
* `version` - 要安装的 Python 版本，格式为 `MAJOR.MINOR`
* `pip-cache-path` - 挂载共享上 `pip` 缓存的路径。详见[共享与缓存](./caches.md)文档
* `set-pip-install-path` - 指示该 Action 是否应设置 `PIP_CACHE_DIR` 和 `PIP_INSTALL_PATH` 环境变量以供后续使用
* `self-hosted-runner` - 指示 runner 是否为自托管。了解[可用 runner](./runners.md)

## 系统信息打印

此自定义 Action 将系统信息打印到标准输出：
* 操作系统
* 机器架构
* CPU、RAM 和内存信息

支持 Linux、macOS 和 Windows。

### 用法
```yaml
  - name: System info
    uses: ./openvino/.github/actions/system_info
```