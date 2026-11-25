# 可复用工作流

为减少重复、提高可维护性，不同工作流中的通用作业被提取为_可复用工作流_。

关于复用 [actions](https://github.com/marketplace?type=actions) 和[工作流](https://docs.github.com/en/actions/using-workflows/reusing-workflows)的更多信息，参见 GitHub 文档。

可复用工作流在多个验证工作流中以 `jobs` 形式引用。其结构与运行方式与[常规作业](./overview.md#single-job-overview)相同，拥有独立的环境、运行器和执行步骤。

本文档介绍 OpenVINO GitHub Actions 中的相关设置。

## 工作流组织

本仓库的所有工作流位于 [workflows 文件夹](../../../../.github/workflows)。

工作流分为两类：
* 以操作系统名开头的文件，例如：[`ubuntu_22.yml`](./../../../../.github/workflows/ubuntu_22.yml)、[`windows_conditional_compilation.yml`](./../../../../.github/workflows/windows_conditional_compilation.yml)。这些是验证工作流，包含对应操作系统、架构和工具集的构建与测试。详见[工作流页面](./overview.md#workflow-structure)。
* 以 `job` 开头的文件，例如：[`job_cxx_unit_tests.yml`](./../../../../.github/workflows/job_cxx_unit_tests.yml)、[`job_samples_tests.yml`](./../../../../.github/workflows/job_samples_tests.yml)。这些是_可复用工作流_，在其他工作流中作为作业使用。

例如，[`job_python_unit_tests.yml`](./../../../../.github/workflows/job_python_unit_tests.yml) 可复用工作流在 [`ubuntu_22.yml`](./../../../../.github/workflows/ubuntu_22.yml)、[`linux_arm64.yml`](./../../../../.github/workflows/linux_arm64.yml) 和 [`mac_arm64.yml`](./../../../../.github/workflows/mac_arm64.yml) 工作流中作为 `Python_Unit_Tests` 作业使用：
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

## 使用可复用工作流

可复用工作流应以 `job` 形式引用。
以下是 [`ubuntu_22.yml`](./../../../../.github/workflows/ubuntu_22.yml) 工作流中引用 [`job_python_unit_tests.yml`](./../../../../.github/workflows/job_python_unit_tests.yml) 可复用工作流的示例：
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
其中：
* `name` - 作业的显示名称；
* `needs` - 作业依赖：必须在本作业启动前完成的作业；
* `uses` - 可复用工作流的路径；
* `with` - 传给可复用工作流的输入键。输入详情参见对应工作流文件。语法参考见 [GitHub Actions 文档](https://docs.github.com/en/actions/using-workflows/reusing-workflows#using-inputs-and-secrets-in-a-reusable-workflow)。

完整参考见 [GitHub Actions 可复用工作流文档](https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow)。

## 添加可复用工作流

若要在多个工作流中添加相似阶段并减少重复，请创建可复用工作流。

在 OpenVINO GitHub Actions CI 中，可复用工作流通常包含：
* 以 `job_` 开头的文件名，例如 [`job_cxx_unit_tests.yml`](./../../../../.github/workflows/job_cxx_unit_tests.yml)
* `runner` 输入，指定执行作业步骤所用的运行器名称。详见[可用运行器及其用法](./runners.md)
* `image` 输入，指定可复用作业使用的 Docker 镜像。可复用工作流定义其所需的容器挂载。详见[在工作流中使用 Docker](./docker_images.md)
* *可选* `affected-components` 输入，表示 Smart CI Action 所定义提交中受影响的组件。详见 [Smart CI 系统](./smart_ci.md)

>**注意**：根据 [GitHub 文档](https://docs.github.com/en/actions/using-workflows/about-workflows#about-workflows)，所有工作流应放在 [`./.github/workflows`](./../../../../.github/workflows) 下。

可复用工作流的结构与行为与作业相同，可参考[添加测试页面](./adding_tests.md)了解如何创建作业，并利用可复用工作流的相关信息将作业转换为可复用工作流。