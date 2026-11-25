# GitHub Actions CI 使用的 OpenVINO Runner

在 GitHub Actions 中，执行工作流命令的机器称为 _Runner_。

本仓库提供两类 Runner：

* [GitHub Actions Runner](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners/about-github-hosted-runners) - 由 GitHub 提供和管理的 Runner
* [自托管 Runner](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners) - 由 OpenVINO CI 团队创建和管理、并关联到 OpenVINO 仓库的 Runner

一般建议轻量任务（如标签处理、代码风格检查等）使用 GitHub Actions Runner，而耗时较长的工作流（如构建或功能测试）应使用自托管 Runner。

每个任务通过 `runs-on` 键指定 Runner。

以下是 [`ubuntu_22.yml`](./../../../../.github/workflows/ubuntu_22.yml) 工作流中 `Build` 任务的示例，使用 `aks-linux-16-cores-32gb` Runner 组：

```yaml
Build:
  ...
  runs-on: aks-linux-16-cores-32gb
  ...
```


## 可用的 GitHub Actions Runner

GitHub 提供不同资源配置和软件组合的 Runner。OpenVINO 仓库使用以下 Runner：

* [默认 Runner](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources)，
  用于内存和 CPU 需求不高的任务：`ubuntu-22/20.04`、`windows-2019/2022`、
  `macos-12/13` 等。

* [大型 Runner](https://docs.github.com/en/actions/using-github-hosted-runners/about-larger-runners/about-larger-runners#machine-sizes-for-larger-runners)，
  用于内存和 CPU 密集型任务，详见 [Runner 页面](https://github.com/openvinotoolkit/openvino/actions/runners)。


## 可用的自托管 Runner

提供两组自托管 Runner：

* 动态创建的 Linux 和 Windows Runner，仅具备 CPU 能力
* 专用 Linux Runner，具备 GPU 能力

### 动态创建的 Linux 和 Windows Runner

这些自托管 Runner 会为每个请求的流水线动态创建。提供多种配置，通过不同的组名区分。组名一般遵循以下模式：
`aks-{OS}-{CORES_N}-cores-|{RAM_SIZE}gb|-|{ARCH}|`，其中：

* `{OS}` - 操作系统：`win`/`linux`（目前仅提供 Windows 和 Linux 自托管 Runner）。
* `{CORES_N}` - 该组 Runner 可用的核心数：`4`/`8`/等。
* `|{RAM_SIZE}gb|` - **_可选_**，该组 Runner 可用的内存大小（GB）：`8`/`16`/等。
  * **注意**：未指定 `{RAM_SIZE}` 的组由 32 GB 内存的 Runner 组成
* `|{ARCH}|` - **_可选_**，该组 Runner 的架构：`arm`
  * **注意**：未指定 `{ARCH}` 的组由 `x86_64` Runner 组成

示例：
* `aks-win-16-cores-32gb` - Windows x86_64 Runner，16 核、32 GB 内存
* `aks-linux-16-cores-arm` - Linux ARM64 Runner，16 核、32 GB 内存

可用配置如下：

|             | 组名              | CPU 核心数   | 内存（GB）       | 架构                 | 示例                                               |
|-------------|-------------------|--------------|------------------|----------------------|----------------------------------------------------|
| Windows     | `aks-win-*`       | `4`/`8`/`16` | `8`/`16`/`32`    | `x86_64`<sup>*</sup> | `aks-win-4-cores-8gb`/`aks-win-16-cores-32gb`      |
| Linux       | `aks-linux-*`     | `4`/`8`/`16` | `16`/`32`        | `x86_64`<sup>*</sup> | `aks-linux-4-cores-16gb`/`aks-linux-16-cores-32gb` |
| Linux ARM64 | `aks-linux-*-arm` | `16`         | `32`<sup>*</sup> | `arm`                | `aks-linux-16-cores-arm`                           |

* `*` - 组名中未指定

### 专用 GPU Runner

OpenVINO 组织的 GitHub Actions 中，所有仓库均可使用 18 个具备 GPU 能力（集成和独立，iGPU 和 dGPU）的 Runner。

这些 Runner 是运行 Ubuntu 22.04 的虚拟机，规格如下：

* CPU：i9-12900k
* 全部具备 iGPU（UHD 770 Graphics）
* 其中 12 个同时具备 iGPU（UHD 770 Graphics）和 dGPU（Arc A770）

可通过任务配置中 `runs-on` 字段提供的标签选择这些 Runner。可用标签：

* `gpu` - 涵盖全部 18 个 Runner
* `igpu` - 涵盖全部 18 个 Runner（均具备 iGPU）
* `dgpu` - 涵盖 12 个具备 dGPU 的 Runner

以下示例中，`GPU Tests` 任务使用 `gpu` 标签，运行在任意可用的 GPU Runner 上：
```yaml
  GPU:
    name: GPU Tests
    needs: [ Build, Docker, Smart_CI ]
    runs-on: [ self-hosted, gpu ]
    container:
      image: ${{ fromJSON(needs.docker.outputs.images).ov_test.ubuntu_22_04_x64_igpu }}
      options: --device /dev/dri:/dev/dri --group-add 109 --group-add 44
      volumes:
        - /dev/dri:/dev/dri
  ...
```

例如，若任务需要 dGPU，应改用 `dgpu` 标签：
```yaml
  GPU:
    name: GPU Tests
    needs: [ Build, Smart_CI ]
    runs-on: [ self-hosted, dgpu ]
  ...
```

**注意**：由于这些是持久化 Runner，使用 GPU Runner 的任务应通过 Docker 运行。可用镜像及选择方法详见 [OpenVINO Docker 镜像概览](./docker_images.md)。

## 如何选择 Runner

任务（构建、测试、其他任务）所需的 Runner 配置取决于任务性质。内存和/或 CPU 需求更高的任务需要更强的配置。

[`ubuntu_22.yml`](./../../../../.github/workflows/ubuntu_22.yml) 工作流中的 `Build` 任务通过 `runs-on` 键使用 `aks-linux-16-cores-32gb` 组：
```yaml
Build:
  ...
  runs-on: aks-linux-16-cores-32gb
  ...
```

`aks-linux-16-cores-32gb` 组包含 16 核 CPU、32 GB 内存的机器。这些资源适合 `Build` 任务中的构建工具并行使用。

[`ubuntu_22.yml`](./../../../../.github/workflows/ubuntu_22.yml) 工作流中的 `C++ unit tests` 任务使用 `aks-linux-4-cores-16gb` 组：
```yaml
CXX_Unit_Tests:
  name: C++ unit tests
  ...
  with:
    runner: aks-linux-4-cores-16gb
    ...
```

C++ 测试无法像 `Build` 任务中的构建工具那样利用大量核心并行执行，因此对它们使用 `aks-linux-16-cores-32gb` 组没有收益。

反之，对于能充分利用资源的任务，建议使用核心数/内存更多的 Runner。

可以先尝试不同配置再做决定。在不同组的 Runner 上运行任务并比较收益。若收益显著，例如 4 核 Runner 上需 60 分钟而 16 核 Runner 上仅需 15 分钟，则应使用核心数更多的配置。