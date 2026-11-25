# OpenVINO GitHub Actions CI 概览

欢迎阅读 OpenVINO 开发者指南中关于 GitHub Actions 基础设施的部分。
本文档将概述 OpenVINO 中使用的配置，并在必要时指引你查阅更详细的说明。

## 目录

* [工作流概览](#workflows)
  * [复用 GitHub Actions](#reusing-github-actions)
  * [触发器和调度](#workflows-triggers-and-schedule)
  * [工作流结构](#workflow-structure)
  * [安全注意事项](#security-considerations)
* [查找结果、构建产物和日志](#finding-results-artifacts-and-logs)
* [自定义 Action 概览](#custom-actions)
* [机器概览](#machines)
* [Docker 镜像概览](#docker-images)
* [缓存概览](#caches)
* [如何添加新测试](#adding-new-tests)
* [基于 PR 变更优化工作流](#optimizing-workflows-based-on-PR-changes)
* [合并队列](#merge-queue)
* [Agentic 工作流](#agentic-workflows)

## 工作流

GitHub Actions 工作流是可配置的自动化流程，运行一个或多个连续的任务（详情请参阅
[GitHub Actions 官方文档](https://docs.github.com/en/actions/using-workflows/about-workflows)）。
它们包括：

* 一系列你通常会在终端中逐条执行的命令
* 关于命令执行环境的信息


你可以在 [workflows 文件夹](/.github/workflows)中找到本仓库的所有工作流。
提供不同操作系统广泛覆盖的主要工作流包括：
* [Linux Ubuntu 22.04](/.github/workflows/ubuntu_22.yml)
* [Linux Ubuntu 24.04](/.github/workflows/ubuntu_24.yml)
* [Linux Ubuntu 26.04](/.github/workflows/ubuntu_26.yml)
* [Windows](/.github/workflows/windows_vs2022_release.yml)
* [macOS ARM64](/.github/workflows/mac_arm64.yml)（提交后和定时验证）

此外，还有几个辅助工作流为其他操作系统和处理器架构构建和测试 OpenVINO：
* [Android](/.github/workflows/android.yml)
* [Linux 条件编译](/.github/workflows/linux_conditional_compilation.yml)
* [Linux RISC-V](/.github/workflows/linux_riscv.yml)
* [Windows 条件编译](/.github/workflows/windows_conditional_compilation.yml)

### 复用 GitHub Actions

OpenVINO 工作流同时使用官方和社区构建的 Action，例如 `actions/checkout`
和 `actions/upload-artifact`。此外，多个工作流中出现的任务被提取为_可复用工作流_。
你可以进一步了解[使用和编写它们](./reusable_workflows.md)，
查看如何[复用工作流](https://docs.github.com/en/actions/using-workflows/reusing-workflows)，
以及了解如何[获取额外的 Action](https://github.com/marketplace?type=actions)。

### 工作流触发器和调度

工作流在由预定义[事件](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)触发时运行。
这些触发器**不是**互斥的，一个工作流可以使用多个触发器。
[Linux 工作流](/.github/workflows/ubuntu_22.yml)使用以下触发器：

* `on: schedule` - 调度触发器
  * 此触发器按指定间隔运行工作流（例如每晚）。
  * 在下面的示例中：`'0 0 * * 3,6'` - 了解更多关于 [cron 语法](https://crontab.guru/)
* `on: workflow_dispatch` - 手动触发器
* `on: pull_request` - 提交前触发器
  * 此触发器在创建针对 `master` 或 `release` 分支的 Pull Request（PR）时运行工作流，
    并在 PR 每次更新新提交时运行。
* `on: merge_group` - 合并队列验证触发器
* `on: push` - 提交后触发器。
  * 此触发器在提交被推送到 `master` 或 `release` 分支时运行工作流
    （例如 PR 被合并时）。

昂贵的验证由 [Smart CI](./smart_ci.md) 选择，而非工作流级别的路径过滤器。

每个工作流的触发器可以在工作流文件开头的 `on` 键中找到。你还应了解如何使用
[paths](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onpushpull_requestpull_request_targetpathspaths-ignore)。




```yaml
on:
  schedule:
    # Wednesday and Saturday with the latest oneDNN
    - cron: '0 0 * * 3,6'
    # Tuesday and Thursday without the latest oneDNN
    - cron: '0 0 * * 2,4'
  workflow_dispatch:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]
  merge_group:
  push:
    branches:
      - master
      - 'releases/**'
```

---
**注意**

必需检查通过分支保护配置，可能独立于本文档发生变化。请查看 Pull Request 上显示的必需状态检查。
提交后和定时工作流（如 macOS ARM64 工作流）不验证 Pull Request。

---


### 工作流结构

Linux、Windows 和 macOS 的工作流结构基本相同：

1. 克隆 OpenVINO 仓库和所需资源
2. 安装构建依赖
3. 从源码构建 OpenVINO
4. 打包并上传构建产物（构建好的 OpenVINO 和测试）
5. 在并行任务中下载并使用构建产物运行不同的测试
6. 收集测试结果并作为构建产物上传

**注意**：部分工作流可能使用相同结构，而其他工作流可能缺少最后 3 步，
测试紧跟在 `Build` 步骤之后。

[Linux 工作流](/.github/workflows/ubuntu_22.yml)概览。
其中包含多个任务：

```yaml
jobs:
  Smart_CI: ...
  Docker: ...
  Build: ...
  Debian_Packages: ...
  Samples: ...
  Conformance: ...
  CXX_Unit_Tests: ...
  Python_Unit_Tests: ...
  Python_API_Tests: ...
  CPU_Functional_Tests: ...
  TensorFlow_Models_Tests_Precommit: ...
  PyTorch_Models_Tests: ...
  JAX_Models_Tests_Precommit: ...
  GGUF_Models_Tests_Precommit: ...
  NVIDIA_Plugin: ...
  Openvino_tokenizers: ...
  iGPU: ...
  Overall_Status: ...
```

`Build` 任务执行前 4 步：
* 克隆 OpenVINO
* 安装依赖
* 使用 `cmake` 从源码构建
* 使用 `actions/upload-artifact` 打包并上传构建产物

其他任务负责使用构建产物运行不同的测试。它们：
* 使用 `actions/download-artifact` 下载并解包构建产物
* 安装所需依赖
* 运行测试
* 收集测试结果
* 将测试结果作为[流水线构建产物](#artifacts)上传

#### 单个任务概览

每个任务都有若干描述其环境的键。建议查看全面的
[语法概览](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)。

本节介绍 OpenVINO CI 环境的特殊之处。

[Linux 工作流](/.github/workflows/ubuntu_22.yml)的 `Python_Unit_Tests` 任务概览：
```yaml
  Python_Unit_Tests:
    name: Python unit tests
    needs: [ Docker, Build, Smart_CI ]
    uses: ./.github/workflows/job_python_unit_tests.yml
    with:
      runner: 'aks-linux-4-cores-16gb'
      image: ${{ fromJSON(needs.docker.outputs.images).ov_test.ubuntu_22_04_x64 }}
      affected-components: ${{ needs.smart_ci.outputs.affected_components }}
      python-version: '3.11'
```

* 使用构建产物的测试任务在 `needs` 中包含 `Build`。使用自定义镜像的任务还依赖
  `Docker`，感知组件的任务依赖 `Smart_CI`。
* 任务使用的机器通过 `runs-on` 键指定。
  * 此例中使用 `aks-linux-4-cores-16gb`。[了解更多](#machines)关于可用机器
    以及如何为任务选择机器。
* 部分任务在 Docker 容器内运行。可复用工作流通过 `image` 输入接收镜像，并在内部配置容器。
  * [了解更多](#docker-images)关于可用镜像以及何时使用。
* 部分任务可能受益于缓存，例如 Python 依赖或 `cmake` 构建产物。
  * [了解更多](#caches)关于如何为任务使用缓存。
* 任务必须定义 `steps` - 在预定义环境中执行的一系列命令。
  * 所有步骤在 `defaults: run:` 下 `shell` 键指定的 shell 中执行，
    除非在步骤中直接指定了 shell。

### 安全注意事项

提交包含 GitHub Actions 工作流变更的 PR 之前，请查阅[工作流安全指南](security.md)。

## 查找结果、构建产物和日志

### 结果

要了解哪些任务已成功通过、哪些正在运行、哪些已失败，
请查看以下内容：

**对于 Pull Request：**
  * 打开 Pull Request 并滚动到页面底部。你将看到最新提交的任务列表，
    包括已完成和仍在运行的任务：

    ![check_results](../../assets/CI_check_results.png)

**对于定时运行：**
  * 导航到 [OpenVINO 仓库 Actions](https://github.com/openvinotoolkit/openvino/actions)
  * 从左侧列表中选择所需的工作流
  * 点击 `Event` 并选择 `schedule` 来筛选运行记录
    * 你还可以按分支、操作者和结果进一步筛选

### 构建产物

构建产物（即工作流生成的文件）仅在已完成的流水线中可用，
无论成功或失败。要查找流水线的构建产物，请按以下步骤操作：

1. 打开 Pull Request 并滚动到任务列表，如上所述。
2. 点击所选任务右侧的 `Details`，查看更多信息。
3. 点击窗口左侧任务列表上方的 `Summary`：

   ![jobs_list](../../assets/CI_completed_job_list.png)

4. 滚动到页面底部
5. 你将找到**此流水线中所有任务**生成的构建产物：

   ![pipeline_artifacts](../../assets/CI_pipeline_artifacts.png)

6. 点击构建产物名称下载。

### 初始 GOLD BOM 基线

缺失的 GOLD BOM 可通过手动
[`Initialize GOLD BOM Baseline` 工作流](../../../../.github/workflows/initialize_gold_boms.yml)生成。
它接受成功的构建工作流运行 ID，从这些运行中收集所有 `bom_*` 构建产物，
并针对启动工作流时选择的分支（包括 `master` 或 `releases/*`）创建 PR。
清单文件存储在 `boms/<configuration>/<component>.json` 下。
配置目录来自构建产物名称（例如，
`bom_ubuntu_22_04_x86_64` 变为 `boms/ubuntu_22_04_x86_64/`），因此 Linux 发行版、
架构和构建变体可以各自拥有自己的基线。构建产物内的平台子目录
只是构建产物的布局细节；不会在 GOLD 路径中重复。

从 **Actions → Initialize GOLD BOM Baseline → Run workflow** 运行，先选择目标
分支。输入来自同一仓库的成功构建运行 ID 的 JSON 数组，
覆盖要初始化的配置。PR 运行和分支运行均可接受，
无论其来源或目标分支。
选择用于初始化的分支仅决定 GOLD BOM PR 的创建位置。
例如：

```json
["123456789", "123456790", "123456791"]
```

工作流仅添加所选分支上缺失的 BOM 文件。现有 GOLD 文件
永远不会被覆盖。如果所有选定的清单文件都已存在，工作流将失败且不
创建 PR。对于现有配置，开发者必须在更改打包文件列表的 PR 中手动更新变更的 BOM。

当添加常规 BOM 比较时，应针对具体配置。如果预期的
GOLD BOM 不存在，则没有可比较的基线：检查应发出警告并附上指向本文档的链接，
并跳过该 BOM 的比较。此引导工作流随后可以通过经过审查的 PR
初始化缺失的 BOM，而无需更改现有基线。

### 日志

要查找流水线的日志：
1. 打开 Pull Request 并滚动到任务列表，如上所述。
2. 点击所选任务右侧的 `Details`，查看更多信息。
3. 点击某个步骤查看其日志

## 自定义 Action

有几个 Action 是专门为 OpenVINO 工作流的需求编写的。在[自定义 Action 文档](./custom_actions.md)中
了解更多关于可用自定义 Action 及其功能的信息。

查看[复用 GitHub Actions](#reusing-github-actions)部分获取更多信息。

## 机器

执行工作流命令的机器在 GitHub Actions 中称为_运行器_。
OpenVINO 组织可使用两种类型的运行器：

* [GitHub Actions 运行器](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners/about-github-hosted-runners) - 由 GitHub 提供和管理的运行器
* [自托管运行器](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners) - 由 OpenVINO CI 团队创建和管理并关联到 OpenVINO 仓库的运行器

工作流根据任务需求使用适当的运行器。在 [OpenVINO 运行器概览](./runners.md)中
了解更多关于可用运行器以及如何选择的信息。

## Docker 镜像

你可以在 Docker 容器中运行任务。
请参阅[语法概览文档](https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container)。

工作流根据任务需求使用适当的 Docker 镜像。在 [OpenVINO Docker 镜像概览](./docker_images.md)中
了解更多关于可用镜像以及如何选择的信息。

## 缓存

有三种类型的缓存可用：
* [GitHub Actions 缓存](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
* 共享驱动器缓存
* 通过 [Azure Blob Storage](https://azure.microsoft.com/en-us/products/storage/blobs) 的远程构建缓存

工作流根据任务需求使用适当的缓存。在 [OpenVINO 缓存概览](./caches.md)中
了解更多关于可用缓存以及如何使用的信息。

## 添加新测试

如果你想添加新测试，请参阅[如何添加测试](./adding_tests.md)文档。

## 基于 PR 变更优化工作流

要仅运行验证 Pull Request 变更实际所需的任务来优化提交前工作流，
你可以使用 Smart CI 功能 - [了解更多](./smart_ci.md)。

## 合并队列

OpenVINO GitHub 组织使用合并队列来自动化 Pull Request 的合并流程。在[官方文档](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue)中了解更多关于合并队列和可用设置的信息。

合并队列有几个按仓库配置的设置。[OpenVINO 仓库](https://github.com/openvinotoolkit/openvino)使用以下设置：
* **合并方式**：*Squash and Merge*
* **构建并发**：
  * **最大构建 Pull Request 数**：*5*
* **仅合并非失败的 Pull Request**：*否*
* **状态检查超时**：*360 分钟*
* **合并限制**：
  * **最少合并 Pull Request 数**：*1*
  * **最多合并 Pull Request 数**：*2*

如果你想为 OpenVINO GitHub 组织中的其他仓库启用合并队列，请将这些设置作为默认值并根据需要调整。

## Agentic 工作流

OpenVINO CI 还包含 **Agentic 工作流** —— 将调查或维护任务委托给 AI Agent 而非运行固定脚本的 GitHub Actions 工作流。目前有两个 CI 故障诊断器：按需 Pull Request 调查器（`/ci-doctor`）和自动合并队列故障调查器。在 [Agentic 工作流](./agentic_workflows.md)文档中了解更多。

## 另请参阅

* [Agentic 工作流](./agentic_workflows.md)
* [GitHub Actions 官方文档](https://docs.github.com/en/actions)