# 如何向 GitHub Actions CI 添加测试

OpenVINO 仓库有[多个工作流](./../../../../.github/workflows)，其中包含构建和测试 OpenVINO 的任务。其总体和结构概览见 [OpenVINO GitHub Actions CI 页面](./overview.md)。

本文档说明如何创建新工作流并添加测试。

## 目录

* [技能校验](#技能校验)
* [向现有工作流添加测试](#向现有工作流添加测试)
  * [向现有测试套件添加测试](#向现有测试套件添加测试)
  * [在任务中创建步骤](#在任务中创建步骤)
  * [创建新任务](#创建新任务)
  * [创建工作流](#创建工作流)
* [测试时间与用量](#测试时间与用量)
  * [添加步骤](#添加步骤)
  * [添加任务](#添加任务)
  * [添加工作流](#添加工作流)

## 技能校验

[文档工作流](../../../../.github/workflows/build_doc.yml)在其现有的 `Check_Doc_Links` 任务中校验技能元数据。仅当 Pull Request 或 merge-group 差异涉及 `.claude/skills/**` 或 `.agents/skills` 符号链接时，校验步骤才会运行，并检查 `.claude/skills/` 中的所有技能。Agent 通过 `.agents/skills/` 访问同一集合。校验遵循文档工作流现有的 Smart CI 和草稿 PR 条件。`.github/agents-prototype/skills` 集合不在该校验的范围内。

校验器检查 [Agent Skills 元数据](https://agentskills.io/specification)、目录与名称一致性、重复的 YAML 键以及非空指令。现有的 Lychee 步骤检查 Markdown 链接，包括技能引用。技能脚本不会被执行；指令质量和 Agent 行为仍需人工审查。

从仓库根目录运行（即使只修改校验器或工作流本身而未改动技能，也需运行，因为 CI 中会跳过元数据校验）：

```sh
python -m pip install -r .github/scripts/skills/requirements.txt
python -m unittest discover -s .github/scripts/skills -p 'test_*.py'
python .github/scripts/skills/validate.py
```

## 向现有工作流添加测试

### 向现有测试套件添加测试

如果新测试可以作为现有测试套件的一部分执行，例如新增的 OVC Python API 测试，则无需修改工作流。新增测试会在对应步骤中自动运行。

查看[工作流](./../../../../.github/workflows)及其任务，确认哪些测试已启用。此外，检查组件测试及其执行方式。

### 在任务中创建步骤

如果任务中没有包含所需测试套件的步骤，可以向该任务添加新步骤。步骤是在单个任务下连续执行并分组在一起的命令。更多信息参见 [GitHub Actions 官方文档](https://docs.github.com/en/actions/using-workflows/about-workflows)。

来自 [`job_python_unit_tests.yml`](./../../../../.github/workflows/job_python_unit_tests.yml) 的示例步骤：
```yaml
...
steps:
...
  - name: OVC unit tests
    if: fromJSON(inputs.affected-components).OVC.test
    run: python3 -m pytest -s ${INSTALL_TEST_DIR}/ovc/unit_tests --junitxml=${INSTALL_TEST_DIR}/TEST-OpenVinoConversion.xml
...
```
该步骤包含：
* `name`：`OVC unit tests`。
* `if` 条件：`fromJSON(inputs.affected-components).OVC.test`
  * 仅当条件为 `true` 时执行该步骤。
  * 这是为 OpenVINO 工作流实现的 Smart CI 系统的一部分。阅读 [Smart CI 概览](./smart_ci.md)了解该系统及其用法。
* `run` 部分，包含要执行的命令。

要添加包含新测试的新步骤，请定位到任务，并使用上述模板（或该任务中任意现有步骤）作为新步骤。参见[可复用工作流页面](./reusable_workflows.md)了解工作流和任务组织。

### 创建新任务

如果新测试与所有工作流中的任何现有任务都不匹配，可以为其创建专用任务。

来自 [`ubuntu_22.yml`](./../../../../.github/workflows/ubuntu_22.yml) 的单个测试集专用任务示例：
```yaml
NVIDIA_Plugin:
  name: NVIDIA plugin
  needs: [ Docker, Build, Smart_CI ]
  timeout-minutes: 30
  defaults:
    run:
      shell: bash
  runs-on: aks-linux-16-cores-32gb
  container:
    image: ${{ fromJSON(needs.docker.outputs.images).ov_build.ubuntu_22_04_x64_nvidia }}
    volumes:
      - /mount:/mount
      - /home/runner/secrets/:/secrets:ro
    options: -e SCCACHE_AZURE_BLOB_CONTAINER
  env:
    CMAKE_BUILD_TYPE: 'Release'
    CMAKE_GENERATOR: 'Ninja Multi-Config'
    CMAKE_CUDA_COMPILER_LAUNCHER: sccache
    CMAKE_CXX_COMPILER_LAUNCHER: sccache
    CMAKE_C_COMPILER_LAUNCHER: sccache
    INSTALL_DIR: /__w/openvino/openvino/install
    OPENVINO_DEVELOPER_PACKAGE: /__w/openvino/openvino/install/developer_package
    OPENVINO_REPO: /__w/openvino/openvino/openvino
    OPENVINO_CONTRIB_REPO: /__w/openvino/openvino/openvino_contrib
    NVIDIA_BUILD_DIR: /__w/openvino/openvino/nvidia_plugin_build
    DEBIAN_FRONTEND: 'noninteractive'
    SCCACHE_AZURE_KEY_PREFIX: ubuntu22_x86_64_Release
  if: fromJSON(needs.smart_ci.outputs.affected_components).NVIDIA

  steps:
    - name: Append the environment variable - load SCCACHE_AZURE_CONNECTION_STRING from file
      shell: bash
      run: |
        SCCACHE_AZURE_CONNECTION_STRING="$(cat /secrets/sccache/connection-string)"
        echo "::add-mask::${SCCACHE_AZURE_CONNECTION_STRING}"
        echo "SCCACHE_AZURE_CONNECTION_STRING=${SCCACHE_AZURE_CONNECTION_STRING}" >> $GITHUB_ENV
        echo "✓ Connection string loaded and masked"
  ...
```

完整语法参考见 [GitHub Actions 官方文档](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#about-yaml-syntax-for-workflows)。

任务：
* 需要名称，由 `name` 键提供
* 需要运行器来执行 `steps`，由 `runs-on` 键提供
  * 了解[可用运行器](./runners.md)
* 可能使用 Docker 执行 `steps`，由 `container` 键配置
  * 了解[可用 Docker 镜像](./docker_images.md)
* 可能使用缓存加速构建和/或测试
  * 有多种缓存类型可用。了解[可用缓存](./caches.md)
* 可能使用 Smart CI 系统通过 `if` 键进行条件执行
  * 更多信息参见 [Smart CI 概览](./smart_ci.md)
* 需要一系列要执行的命令，由 `steps` 键提供
  * 参见[在任务中创建步骤](#在任务中创建步骤)了解 `steps`
* 可能使用 `Build` 任务的构建产物
  * 可使用 `actions/download-artifact` 下载产物，工作流结构详见 [OpenVINO GitHub Actions CI 概览](./overview.md#workflow-structure)

如果任务可在多个工作流中使用，可将其转换为可复用工作流。详见[可复用工作流页面](./reusable_workflows.md)。

## 创建工作流

要创建工作流，向 [`.github/workflows`](./../../../../.github) 文件夹添加新的 `<name>.yml` 文件。完整语法参考见 [GitHub Actions 官方文档](https://docs.github.com/en/actions/using-workflows/about-workflows#create-an-example-workflow)，并浏览 [`.github/workflows`](./../../../../.github) 中的现有工作流。

可参考[现有工作流的结构概览](./overview.md#workflow-structure)作为新工作流的模板。

[`ubuntu_22.yml`](./../../../../.github/workflows/ubuntu_22.yml) 工作流示例包含：
* 以下任务：
  * `Smart_CI` - [Smart CI 系统](./smart_ci.md)。
  * `Build` - 安装前置依赖、使用指定 CMake 配置构建 OpenVINO、打包并上传产物。
  * `Overall_Status` - 收集其他任务状态的任务
* 使用 `actions/upload-artifact` 和 `actions/download-artifact` 在任务间上传和下载构建产物
* 使用 [Smart CI 系统](./smart_ci.md)
* 使用[自托管运行器](./runners.md)和 [Docker 镜像](./docker_images.md)
* 使用[缓存](./caches.md)

## 测试时间与用量

添加新步骤、任务和工作流时，注意时间和运行器用量。添加任何新测试或更改现有测试范围时，务必在 Pull Request 中检查测试执行时间。经验法则：precommit 中任何工作流的执行时间应少于 60 分钟。另请注意，即使按建议执行时间少于 60 分钟，在没有适当工作负载的情况下长时间运行强大的多核运行器也可能成本过高。

### 添加步骤

在任务中添加步骤时，检查其执行时间与其他任务的对比。尽量与这些任务保持一致，避免过度延长整体任务时长。

如果步骤耗时过长，考虑[将其提取为单独任务](#添加任务)，以便与其他任务并行运行。此外，对于步骤较长的任务，可以选择[更强大的运行器](./runners.md)以缩短执行时间。

### 添加任务

添加任务时，检查工作流中现有任务的执行时间。新任务的执行时间不应超过工作流中最长任务的时间。

如果任务耗时较长，考虑在 post-commit/nightly/weekly 基础上运行，而非 pre-commit。参见 [OpenVINO GitHub Actions CI 概览](./overview.md#workflows-triggers-and-schedule)了解触发器和调度。此外，[使用更强大的运行器](./runners.md)有助于缩短执行时间。

### 添加工作流

添加新工作流时，检查现有工作流的执行时间。新工作流的执行时间不应超过最长工作流的时间。

如果工作流耗时较长，考虑在 post-commit/nightly/weekly 基础上运行，而非 pre-commit。参见 [OpenVINO GitHub Actions CI 概览](./overview.md#workflows-triggers-and-schedule)了解触发器和调度。此外，确保为每个任务选择[合适的运行器](./runners.md)以优化执行时间。