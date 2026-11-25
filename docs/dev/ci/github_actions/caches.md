# 缓存

OpenVINO 使用缓存来加速构建和测试，同时尽量减少网络使用。

## 目录

* [可用缓存](#可用缓存)
* [GitHub Actions 缓存](#github-actions-缓存)
* [共享驱动器缓存](#共享驱动器缓存)
* [通过 Azure Blob Storage 的云存储](#通过-azure-blob-storage-的云存储)


## 可用缓存

有三种缓存可用：
* [GitHub Actions 缓存](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
  * 适用于 GitHub 托管和自托管运行器。
  * 可通过 `actions/cache` action 访问。
  * 每个仓库限制为 10 GB。
  * 适用于小型依赖缓存和可复用构建产物。
* [共享驱动器缓存](#共享驱动器缓存)
  * 仅适用于自托管运行器。
  * 通过预定义路径自动可用。
  * 大容量存储。
  * 适用于大型缓存，如构建缓存、模型和数据集。
* 通过 [Azure Blob Storage](https://azure.microsoft.com/en-us/products/storage/blobs) 的云存储
  * 仅适用于自托管运行器。
  * 用于通过 [`sccache`](https://github.com/mozilla/sccache) 缓存和共享构建产物。

工作流中的作业根据需求使用这些缓存。

## GitHub Actions 缓存

此缓存用于在运行之间共享小型依赖或构建产物。
完整参考请参阅 [GitHub Actions 官方文档](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)。

[`ubuntu_22.yml`](./../../../../.github/workflows/ubuntu_22.yml) 工作流中的 `CPU functional tests` 作业
使用此缓存共享测试执行时间，以加速后续运行。
首先，使用 `actions/cache/save` 以特定键
`${{ runner.os }}-${{ runner.arch }}-tests-functional-cpu-stamp-${{ github.sha }}` 保存构建产物：
```yaml
CPU_Functional_Tests:
  name: CPU functional tests
  ...
  steps:
    - name: Save tests execution time
      uses: actions/cache/save@27d5ce7f107fe9357f9df03efb73ab90386fccae # v5.0.5
      if: github.ref_name == 'master'
      with:
        path: ${{ env.PARALLEL_TEST_CACHE }}
        key: ${{ runner.os }}-${{ runner.arch }}-tests-functional-cpu-stamp-${{ github.sha }}
    ...
```

然后它会出现在[仓库的缓存](https://github.com/openvinotoolkit/openvino/actions/caches)中：
![gha_cache_example](../../assets/CI_gha_cache_example.png)

后续运行可以使用 `actions/cache/restore` 从仓库缓存中下载该构建产物并使用：
```yaml
CPU_Functional_Tests:
  name: CPU functional tests
  ...
  steps:
    - name: Restore tests execution time
      uses: actions/cache/restore@27d5ce7f107fe9357f9df03efb73ab90386fccae # v5.0.5
      with:
        path: ${{ env.PARALLEL_TEST_CACHE }}
        key: ${{ runner.os }}-${{ runner.arch }}-tests-functional-cpu-stamp-${{ github.sha }}
        restore-keys: |
          ${{ runner.os }}-${{ runner.arch }}-tests-functional-cpu-stamp
    ...
```
`restore-keys` 键用于查找所需的缓存条目。`actions/cache` 会搜索
完全或部分匹配项，并将找到的缓存下载到提供的 `path`。

完整语法参考请参阅 [actions/cache 文档](https://github.com/actions/cache)。

## 共享驱动器缓存

此缓存用于存储依赖项和大型资源（如模型和数据集），
供不同工作流作业使用。

>**注意**：此缓存仅对 Linux [自托管运行器](./runners.md)启用。

该驱动器在自托管机器上可用。要在 [Docker 容器](./docker_images.md)内使用它，
请在作业配置的 `container` 的 `volumes` 键下添加挂载点：
```yaml
Build:
  ...
  needs: Docker
  runs-on: <self-hosted runner>
  container:
    image: ${{ fromJSON(needs.docker.outputs.images).ov_build.<image> }}
    volumes:
      - <shared drive path>:<shared drive path>
      - <secrets path on runner>:/secrets:ro
    options: -e SCCACHE_AZURE_BLOB_CONTAINER
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

在 `- <shared drive path>:<shared drive path>` 中，第一个路径是共享驱动器
在运行器上的位置，第二个是 Docker 容器内资源可用的路径。
请使用与目标运行器现有工作流相同的挂载点。

### 可用资源

* `pip` 缓存
  * 通过工作流级别定义的 `PIP_CACHE_PATH` 环境变量访问
  * 用于涉及 Python 的作业
* 测试用 onnx 模型
  * 存储在共享驱动器上的专用文件夹中
  * 用于 [`ubuntu_22.yml`](./../../../../.github/workflows/ubuntu_22.yml) 工作流中的 `ONNX Models tests` 作业
* Linux RISC-V 的 Conan 构建产物
  * 用于 [`linux_riscv.yml`](./../../../../.github/workflows/linux_riscv.yml) 工作流

如需添加新资源，请联系 CI 团队成员获取帮助。

### 远程存储编译器缓存

构建可以通过将 `CCACHE_SECONDARY_STORAGE`（或较新版本 ccache 的 `CCACHE_REMOTE_STORAGE`）设置为
`file://` URL，利用共享驱动器上 ccache 的远程存储来共享编译对象。每个构建配置在共享存储内使用自己的目录。
使用者：
* [`linux_riscv.yml`](./../../../../.github/workflows/linux_riscv.yml)
  和 [`linux_riscv_conan.yml`](./../../../../.github/workflows/linux_riscv_conan.yml) 中的 RISC-V GNU 和 Conan 构建。
* [`job_build_windows.yml`](./../../../../.github/workflows/job_build_windows.yml) 和
  [`windows_conditional_compilation.yml`](./../../../../.github/workflows/windows_conditional_compilation.yml) 中的 Windows 构建。

ccache 无法缓存使用独立 PDB（`/Zi`）的 MSVC 编译，因此 Windows Debug 构建
设置 `-DCMAKE_MSVC_DEBUG_INFORMATION_FORMAT=Embedded`（`/Z7`），将调试信息
存储在对象文件中。使用嵌入调试信息的构建链接时不使用 `/INCREMENTAL`，从而保持
PDB 文件不含过时的增量链接数据。关于静态库的处理方式，请参阅
[自定义编译的 CMake 选项](./../../cmake_options_for_custom_compilation.md)中的 `ENABLE_STATIC_LIBS_PDB`。

`CCACHE_MAXSIZE` 仅限制本地缓存。远程存储由
[`cleanup_caches.yml`](./../../../../.github/workflows/cleanup_caches.yml) 中的
`Cleanup_remote_ccache_*` 作业定期清理，使用
`ccache --trim-dir --trim-method mtime`（ccache 4.8 或更新版本），将最近最少使用的
条目逐出至 `CCACHE_REMOTE_MAX_SIZE`。构建必须在远程存储
URL 中保持 `update-mtime=true`，以便缓存命中时刷新用于逐出的修改时间。

## 通过 Azure Blob Storage 的云存储

此缓存用于在运行之间共享 OpenVINO 构建产物。
[`sccache`](https://github.com/mozilla/sccache) 工具可以缓存构建文件并将其上传到 [Azure Blob Storage](https://azure.microsoft.com/en-us/products/storage/blobs) 或从中下载。

>**注意**：此缓存仅对[自托管运行器](./runners.md)启用。

`sccache` 需要多项配置才能工作：
* 安装。请参阅 [sccache 安装](#sccache-安装)部分。
* [凭据环境变量：`SCCACHE_AZURE_BLOB_CONTAINER`、`SCCACHE_AZURE_CONNECTION_STRING`](#传递凭据环境变量)
  * 这些变量已在自托管运行器上设置好。
  * 可以通过 `container` 键下的 `options` 键将变量传递给 Docker 容器。
* [`SCCACHE_AZURE_KEY_PREFIX` 环境变量](#提供-sccache-前缀)，用于指定当前 OS/架构的缓存保存文件夹。
* [`CMAKE_CXX_COMPILER_LAUNCHER` 和 `CMAKE_C_COMPILER_LAUNCHER` 环境变量](#为-cc-文件启用-sccache)，用于为 C++/C 构建文件启用 `sccache` 缓存

### `sccache` 安装

安装通过社区提供的 `mozilla-actions/sccache-action` action 完成：
```yaml
- name: Install sccache
  uses: mozilla-actions/sccache-action@v0.0.3
  with:
    version: "v0.5.4"
```

此步骤必须放在工作流中构建步骤**之前**。

### 传递凭据环境变量

必须在环境中设置 `SCCACHE_AZURE_BLOB_CONTAINER` 和 `SCCACHE_AZURE_CONNECTION_STRING` 变量，
以启用 `sccache` 与 Azure Blob Storage 的通信。

这些变量已在自托管运行器上不使用 Docker 容器的作业环境中设置好，
无需进一步操作。


如果作业需要 [Docker 容器](./docker_images.md)，请通过 `container` 键下的 `options`
键传递变量，使容器内的 `sccache` 可以访问它们：
```yaml
Build:
  ...
  needs: Docker
  runs-on: <self-hosted runner>
  container:
    image: ${{ fromJSON(needs.docker.outputs.images).ov_build.<image> }}
    volumes:
      - <shared drive path>:<shared drive path>
      - <secrets path on runner>:/secrets:ro
    options: -e SCCACHE_AZURE_BLOB_CONTAINER
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

### 提供 `sccache` 前缀

远程存储上保存当前 OS/架构缓存的文件夹
通过作业 `env` 键下的 `SCCACHE_AZURE_KEY_PREFIX` 环境变量提供：
```yaml
Build:
  ...
  env:
    ...
    CMAKE_CXX_COMPILER_LAUNCHER: sccache
    CMAKE_C_COMPILER_LAUNCHER: sccache
    ...
    SCCACHE_AZURE_KEY_PREFIX: ${{ inputs.os }}_${{ inputs.arch }}_Release
```

### 为 C++/C 文件启用 `sccache`

要指示 CMake 使用缓存工具，请在作业的 `env` 键下设置 `CMAKE_CXX_COMPILER_LAUNCHER`
和 `CMAKE_C_COMPILER_LAUNCHER` 环境变量：
```yaml
Build:
  ...
  env:
    ...
    CMAKE_CXX_COMPILER_LAUNCHER: sccache
    CMAKE_C_COMPILER_LAUNCHER: sccache
    ...
    SCCACHE_AZURE_KEY_PREFIX: ${{ inputs.os }}_${{ inputs.arch }}_Release
```
你也可以在 CMake 配置命令中设置这些选项。