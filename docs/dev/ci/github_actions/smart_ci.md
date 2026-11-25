# Smart CI 概述

Smart CI 是一项用于优化 pre-commit CI 工作流的功能，它只运行验证给定 Pull Request（PR）中变更所必需的构建和测试。

例如，如果某个 PR 只修改了 CPU 插件，那么该 PR 在 pre-commit 阶段会跳过 GPU 插件测试，因为二者无关。这种方式缩短了产品组件中孤立变更的执行时间，并降低了对有限硬件资源的负载。

> **产品组件**是负责特定产品功能的独立功能单元。它由仓库中一组文件定义（例如 openvino/src/_some_folder_/**），这些文件包含该功能的实现。

本文档介绍 OpenVINO GitHub Actions pre-commit 工作流中 Smart CI 的实现，并说明如何添加或修改其规则。

>**注意**：需要具备 [GitHub Actions 工作流](https://docs.github.com/en/actions)的基础知识。

## 目录

* [实现](#implementation)
* [Smart CI 规则配置](#configuration-of-smart-ci-rules)
  * [产品组件定义：.github/labeler.yml](#product-components-definition)
  * [组件间依赖定义：.github/components.yml](#definition-of-dependencies-between-components)
  * [流水线行为细节](#specifics-of-pipeline-behavior)
* [如何贡献](#how-to-contribute)
  * [添加新组件](#adding-a-new-component)
  * [为组件添加验证](#adding-validation-for-a-component)
  * [为工作流添加 Smart CI 支持](#adding-support-for-smart-ci-to-a-workflow)
  * [针对特定变更跳过整个工作流](#skipping-an-entire-workflow-for-specific-changes)
  * [为 OpenVINO 仓库外的组件添加 Smart CI](#adding-smart-ci-for-components-outside-the-openvino-repository)


## 实现

Smart CI 以[自定义 GitHub Action](https://docs.github.com/en/actions/creating-actions/about-custom-actions) 的形式实现，存放在 openvino 仓库的 [.github/actions/smart-ci](../../../../.github/actions/smart-ci) 中。在 GitHub Actions 工作流中，该 Action 作为独立 Job 的第一步被调用：
```yaml
jobs:
  Smart_CI:
    outputs:
      affected_components: "${{ steps.smart_ci.outputs.affected_components }}"
      skip_workflow: "${{ steps.smart_ci.outputs.skip_workflow }}"
    steps:
      - name: Get affected components
        id: smart_ci
        uses: ./.github/actions/smart-ci
        ...
```
它以 PR 数据为输入，[输出](https://docs.github.com/en/actions/using-jobs/defining-outputs-for-jobs)受该 PR 影响的产品组件列表，以及每个组件的验证范围。验证范围可以是仅"build"，或同时包含"build"和"test"（组件必须先构建才能测试）。

以下是一个 PR 仅修改 TensorFlow Frontend 组件（包括 src/frontends/tensorflow 内的文件）时的输出示例：
```
changed_component_names: {'TF_FE'}  # TF_FE 是我们为 TensorFlow Frontend 组件选择的别名
affected_components={
    "TF_FE": {"test": true, "build": true},
    "OVC": {"test": true, "build": true},
    "CPU": {"build": true},
    "Python_API": {"build": true},
    ...
}
```

Smart CI Job 结束后，验证 Job 开始执行。根据 Smart CI 的输出，部分 Job 可被完全跳过，其他 Job 则只跳过特定步骤。这通过 GitHub Actions [条件](https://docs.github.com/en/actions/using-jobs/using-conditions-to-control-job-execution)实现。例如，以下 Job 仅在 PR 涉及 TensorFlow 或 TensorFlow Lite Frontend 且需要运行 `test` 范围时执行：
```yaml
TensorFlow_Models_Tests_Precommit:
  needs: [Docker, Build, Smart_CI, Openvino_tokenizers]
  ...
  if: fromJSON(needs.smart_ci.outputs.affected_components).TF_FE.test ||
      fromJSON(needs.smart_ci.outputs.affected_components).TFL_FE.test
  steps:
    - ...
```

## Smart CI 规则配置

Smart CI 依据 OpenVINO 仓库中两个配置文件所描述的规则运行。

### 产品组件定义

[.github/labeler.yml](../../../../.github/labeler.yml)

该文件包含源代码路径到对应组件名称的映射。它作为 [actions/labeler](https://github.com/marketplace/actions/labeler?version=v4.3.0) GitHub Action 的配置，用于根据 PR 变更集自动为 PR 分配标签。Smart CI 复用了该配置，因此其中每个标签都被视为一个组件名称。labeler Action 会自动识别每个 PR 中变更了哪些组件。例如：
```yaml
'category: CPU':
- 'src/plugins/intel_cpu/**/*'
- 'src/common/snippets/**/*'
- 'thirdparty/xbyak/**/*'
```
如果 PR 修改了至少一个匹配上述任一 [minimatch glob 模式](https://github.com/isaacs/minimatch#readme)的文件，该 PR 就会被赋予"category: CPU"标签。使用 Smart CI 功能的 GitHub Actions 工作流会将名为"CPU"的组件视为已变更（组件名中省略"category:"前缀）。

### 组件间依赖定义

[.github/components.yml](../../../../.github/components.yml)

部分组件并非完全独立，修改它们可能影响其他组件。此时，除了验证被修改组件本身（包括构建和测试），还需验证依赖组件（仅构建，或构建加测试）。该文件描述组件间的关系，例如：
```yaml
PyTorch_FE:       # 组件名称
  revalidate:     # 定义当上述组件被修改时需要重新验证（build + test）的组件列表
    - OVC         # 该组件依赖 PyTorch_FE，需要完整重新验证
  build:          # 定义当 PyTorch_FE 被修改时需要构建的组件列表（跳过其测试运行）
    - CPU         # 当 PyTorch_FE 被修改时，该组件及下方组件必须构建
    - Python_API
```
对于上述示例，仅修改 PyTorch_FE 时将执行以下流水线：

* 构建 PyTorch_FE
* 测试 PyTorch_FE
* 构建 OVC
* 测试 OVC
* 构建 CPU
* 构建 Python_API

>**注意**：依赖关系**不**具有传递性。例如，组件"A"依赖组件"B"，组件"B"依赖组件"C"，并不自动认为"A"依赖"C"。每个组件必须显式列出其所有依赖方。

### 流水线行为细节
* 如果被修改的组件**未在** components.yml 中定义，则视为它影响所有其他组件，并执行完整验证范围。

* 如果 PR 影响**多个**组件，则执行验证所有这些组件所需的全部 Job。

* 如果 PR 修改的文件**不属于任何已知组件**，则执行完整验证范围，以避免未标记变更带来的潜在回归。为此，使用了 [actions/labeler v4.3.0](https://github.com/marketplace/actions/labeler?version=v4.3.0) 的一个[补丁版本](https://github.com/akladiev/labeler/releases/tag/v4.3.1)，功能相同但增加了额外特性。这使开发者能够检测 PR 修改了不匹配 labeler.yml 配置中任何模式的文件的情况。

## 如何贡献

### 添加新组件

1. 在 [.github/labeler.yml](../../../../.github/labeler.yml) 中添加新记录。根级键是组件（标签）名称，值是一组 glob，用于定义与该组件相关的源代码路径。参见 [labeler 用法](https://github.com/marketplace/actions/labeler?version=v4.3.0)了解 glob 语法。
2. 在 [.github/components.yml](../../../../.github/components.yml) 中添加新记录。根级键是组件名称，与上一步定义的标签名相同，但省略"category:"前缀（如有）。如果标签名中有空格，用下划线替换。例如：labeler.yml 中的 `'category: LP transformations'` -> components.yml 中的 `LP_transformations`。填写值时，查看 components.yml 中的其他组件，选择可能受新组件变更影响的组件。需要完整重新验证（build 和 test）的组件放在 `revalidate` 键下；仅需构建的放在 `build` 键下。示例记录：
    ```yaml
    your_component:
      revalidate:
        - component_1
        - component_2
      build:
        - component_3
    ```
    如果你的组件不影响任何其他组件，在 `revalidate` 和 `build` 下都指定空列表：
    ```yaml
    your_component:
      revalidate: []
      build: []
    ```
    如果要显式表明某组件影响所有其他组件，在 `revalidate` 下使用"all"表示法。这会在你的组件变更时执行完整流水线，等同于在 components.yml 中完全省略该记录。或者，在 `build` 下使用"all"表示法，表示你的组件变更会触发构建（但不测试）所有其他组件：
    ```yaml
    your_component_1:
      build: 'all'

    your_component_2:
      revalidate: 'all'
    ```
4. 查看 components.yml 中列出的其他组件。检查当这些组件发生变更时，你的组件是否需要被验证。如果需要，将你的组件名添加到对应组件的 `revalidate` 或 `build` 部分下。

### 为组件添加验证
如果要为新组件添加验证 Job 或选择现有 Job，请前往 [.github/workflows](../../../../.github/workflows) 中所需的工作流。主要工作流包括 [ubuntu_22.yml](../../../../.github/workflows/ubuntu_22.yml)、[windows_vs2022_release.yml](../../../../.github/workflows/windows_vs2022_release.yml) 和 [mac_arm64.yml](../../../../.github/workflows/mac_arm64.yml)。如果流水线已启用 Smart CI，你会在工作流开头看到 Smart_CI Job：
```yaml
jobs:
  Smart_CI:
    ...
    steps:
      - name: Get affected components
        id: smart_ci
      ...
```

或者，你可以为测试组件创建独立工作流。更多信息请参阅以下页面：
- [为工作流添加 Smart CI 支持](#adding-support-for-smart-ci-to-a-workflow)
- [使用工作流](https://docs.github.com/en/actions/using-workflows/about-workflows) - GitHub 官方文档


拥有验证组件的 Job 后：
* 将 Smart_CI 添加到该 Job 的 [needs](https://docs.github.com/en/actions/using-jobs/using-jobs-in-a-workflow#defining-prerequisite-jobs) 块中。这确保该 Job 能访问 Smart CI 输出：
  ```yaml
  job_that_validates_your_component:
    needs: Smart_CI  # 如果此处已指定其他 Job，将 Smart_CI 加入列表： [Other_Job_ID, Smart_CI]
    ...
  ```
* 添加 ["if" 条件](https://docs.github.com/en/actions/using-jobs/using-conditions-to-control-job-execution)引用 Smart CI 输出。要有条件地运行整个 Job，将其添加到与"needs"键相同的层级：
  ```yaml
  # 当 YOUR_COMPONENT 受影响且需要 "test" 范围时，以下 Job 将启动
  job_that_validates_your_component:
    needs: [Build, Smart_CI]
    ...
    if: fromJSON(needs.smart_ci.outputs.affected_components).YOUR_COMPONENT.test # 或 <...>.build（如需要）
    steps:
      - ...
  ```
  如果只需在组件变更时执行 Job 中的特定步骤，将"if"添加到所需步骤。语法见 [GitHub Actions 文档](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#runsstepsif)：
  ```yaml
  job_that_validates_your_component:
    needs: [Build, Smart_CI]
    ...
    steps:
      # 当 YOUR_COMPONENT 受影响且需要 "build" 范围时，以下步骤将启动
      - name: step_name
        if: fromJSON(needs.smart_ci.outputs.affected_components).YOUR_COMPONENT.build # 或 <...>.test（如需要）
  ```
  >**注意**：为 Job 内步骤添加 Smart CI 条件时，确保 Job 本身不会在仅修改你的组件时被跳过。为此，检查 Job 级别的"if"条件。它要么不存在（此时 Job 始终执行），要么在你希望条件步骤执行的所有情况下返回"True"。

  你也可以使用任意布尔运算符编写复杂条件，例如：
  ```yaml
    # 当 COMPONENT_1 或 COMPONENT_2 被修改时，以下条件将强制 Job/步骤运行
    if: fromJSON(needs.smart_ci.outputs.affected_components).COMPONENT_1.test ||
        fromJSON(needs.smart_ci.outputs.affected_components).COMPONENT_2.test
  ```
  参见[表达式](https://docs.github.com/en/actions/learn-github-actions/expressions)页面了解条件中的表达式。

### 为工作流添加 Smart CI 支持
要在工作流中使用 Smart CI，在 `jobs` 块下、所有将使用它的 Job 之前添加以下代码：
```yaml
jobs:
  Smart_CI:
    runs-on: ubuntu-latest
    outputs:
      affected_components: "${{ steps.smart_ci.outputs.affected_components }}"
      skip_workflow: "${{ steps.smart_ci.outputs.skip_workflow }}"
    steps:
      - name: checkout action
        uses: actions/checkout@v4
        with:
          sparse-checkout: .github/actions/smart-ci

      - name: Get affected components
        id: smart_ci
        uses: ./.github/actions/smart-ci
        with:
          repository: ${{ github.repository }}
          pr: ${{ github.event.number }}
          commit_sha: ${{ github.sha }}
          component_pattern: "category: (.*)"
          repo_token: ${{ secrets.GITHUB_TOKEN }}
```
如需要，可向"Get affected components"步骤传递更多参数，完整列表见 [.github/actions/smart-ci/action.yml](../../../../.github/actions/smart-ci/action.yml)。

之后，你可以在验证 Job 中引用 Smart_CI 的输出，如[为组件添加验证](#adding-validation-for-a-component)一节所述。要了解 GitHub Actions 工作流语法，参见[工作流](https://docs.github.com/en/actions/using-workflows/about-workflows)页面。

### 针对特定变更跳过整个工作流

要根据特定条件跳过整个工作流，可以使用 GitHub 默认提供的 [paths-ignore](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onpushpull_requestpull_request_targetpathspaths-ignore) 功能。但该功能不能用于包含标记为合并"Required"的 Job 的工作流。详见[被跳过但必需的检查](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/troubleshooting-required-status-checks#handling-skipped-but-required-checks)。

作为变通方案，Smart CI 返回一个指示器，当 PR _仅_带有给定标签和/或仅修改匹配给定 [fnmatch](https://docs.python.org/3/library/fnmatch.html) 模式的文件时，可完全跳过工作流。这些标签和模式通过 `with` 块作为输入传给 Smart CI Action。该指示器作为名为 `skip_workflow` 的独立输出返回，例如：
```yaml
  Smart_CI:
    runs-on: ubuntu-latest
    outputs:
      ...
      # 仅当工作流可完全跳过时，以下输出才会被设置，否则为空
      skip_workflow: "${{ steps.smart_ci.outputs.skip_workflow }}"
    steps:
      ...
      - name: Get affected components
        id: smart_ci
        uses: ./.github/actions/smart-ci
        with:
          ...
          # 用于跳过整个工作流的逗号分隔规则
          skip_when_only_listed_labels_set: 'docs'
          skip_when_only_listed_files_changed: '*.md,*.rst,*.png,*.jpg,*.svg'
```
然后可用 `skip_workflow` 输出有条件地运行工作流中的**父** Job。父 Job 是所有其他 Job 之前必须通过的 Job，在所有 Job 的"needs"块中指定。例如，Build Job 可视为父 Job。

`skip_workflow` 条件如下：
```yaml
Build:
    needs: Smart_CI
    ...
    if: "!needs.smart_ci.outputs.skip_workflow"
    ...
```
>**注意**：如果工作流有多个父 Job，必须为每个父 Job 添加该条件。

此方案有效，是因为 GitHub 将跳过的检查视为成功。与通过路径过滤跳过的必需工作流不同，它们不会阻止合并。

Smart CI 还在 [`smart_ci.py`](../../../../.github/actions/smart-ci/smart_ci.py) 中为 agentic 工作流和 rerunner 自动化维护了一份共享文件允许列表。它涵盖 gh-aw 源文件、生成的锁文件、共享片段、Action 配置、CI Doctor Schema、safe-output 脚本、rerunner 脚本/错误数据库/测试、其编译检查和 rerunner 工作流，以及 agentic 工作流技能。这些路径与各工作流的文件跳过模式合并。仅包含匹配文件的 PR 或合并队列变更集将跳过昂贵的产品验证，即使 `category: CI`、`github_actions` 或 `no-match-files` 原本会请求完整范围。任何不匹配的文件都会保留正常验证；post-commit 运行不会被跳过。不要笼统地豁免 `.github/**`：构建/测试工作流、共享 Action 和 Smart CI 配置仍必须接受正常验证。

工作流运行本身（包括必需状态聚合）仍会启动；Smart CI 跳过其昂贵的 Job。专用编译、工作流扫描和其他独立触发的检查继续运行。