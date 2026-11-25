# OpenVINO Provider

这是一个自定义 GitHub Action，为指定修订版本提供预构建的 OpenVINO 构建产物——可以是特定提交、分支上最新的可用提交，或公开发布的包。它面向依赖 OpenVINO 并需要对其进行验证的第三方仓库的 GitHub Action 工作流，免去从源码重新构建 OpenVINO 的麻烦。

## 用法
一个简单示例：在工作流中通过独立 job 调用 OpenVINO provider：
```yaml
  openvino_download:
    name: Download prebuilt OpenVINO
    outputs:
      status: ${{ steps.openvino_download.outcome }}
      ov_wheel_source: ${{ steps.openvino_download.outputs.ov_wheel_source }}
      ov_version: ${{ steps.openvino_download.outputs.ov_version }}
      docker_tag: ${{ steps.get_docker_tag.outputs.docker_tag }}
    timeout-minutes: 10
    defaults:
      run:
        shell: bash
    runs-on: aks-linux-medium
    container:
      image: 'openvinogithubactions.azurecr.io/openvino_provider:0.1.0'
      volumes:
        - /mount:/mount
        - ${{ github.workspace }}:${{ github.workspace }}

    steps:
    - uses: openvinotoolkit/openvino/.github/actions/openvino_provider@master
      id: openvino_download
      with:
        platform: 'ubuntu22'
        revision: latest_available_commit
```
### 主要输入参数
* `revision` - 指定下载哪个 OpenVINO 版本的构建产物。可选值：
  * `latest_available_commit` - 返回通过 OpenVINO GHA 工作流构建的最新完整 _post-commit_ 构建产物。
  * `<specific commit hash>` / `HEAD` - 返回指定提交哈希或 HEAD 的 _post-commit_ 构建产物（如可用）。
  * `latest_nightly` - 从 [storage.openvinotoolkit.org](https://storage.openvinotoolkit.org/repositories/openvino/packages/nightly) 获取最新的 nightly 构建产物。
  * `<specific package version>`（如 `2025.1` 或 `2024.4.0rc2`）- 从 [storage.openvinotoolkit.org](https://storage.openvinotoolkit.org/repositories/openvino/packages) 获取任意其他正式版/预发布版/自定义包。


**注意**：OpenVINO provider 的_执行_仅支持 Linux GitHub runner；但它也能提供 Windows 构建产物——可通过 GitHub 存储传递到 Windows runner（参见 openvino_tokenizers 中的[示例](https://github.com/openvinotoolkit/openvino_tokenizers/blob/77f7abc0a900b2f189f397576a1f03aa9ffab383/.github/workflows/windows.yml#L98)）。要下载 _post-commit_ 构建产物，运行该 action 的 job 必须运行在 [Azure 托管的 Linux runner](./runners.md) 上，例如上例中的 `runs-on: aks-linux-medium`。


* `platform` - 指定下载哪个操作系统的构建产物。可选值见输入说明[此处](../../../../.github/actions/openvino_provider/action.yml)。
* `arch` - 指定目标架构（默认为 x86_64）。有效值见输入说明[此处](../../../../.github/actions/openvino_provider/action.yml)。
* `install_dir` - 若指定，则将构建产物下载到给定的本地路径；否则构建产物会安装到 GitHub 工作区根目录，然后上传到 GitHub 构建产物存储（当需要在多个不同 job 中复用构建产物，或将其传递到不兼容的 runner（如 Windows 或 GitHub 托管 runner，这些环境无法运行此 action）时，上传到 GitHub 存储很有用）。

### 主要输出
下载的构建产物信息通过 action 的输出提供：

* `ov_artifact_name` - 构建产物上传到 GitHub 存储时使用的名称（仅在未设置本地 install_dir 时适用）。
* `ov_wheel_source` - 作为 `pip install` 命令选项传入的字符串，用于指定 OpenVINO Python wheel 的位置（`--find-links` 或 `--extra-index-url`）。
* `ov_version` - 与下载的构建产物关联的 OpenVINO 版本（可用于在安装时显式指定版本，或用于报告目的）。

### 输出使用示例：
```yaml
  openvino_tokenizers_tests:
    name: OpenVINO tokenizers tests
    needs: [ openvino_download ]
    defaults:
      run:
        shell: bash
    runs-on: ubuntu-22.04
      steps:
      ...
      - name: Download OpenVINO package
        uses: akashchi/download-artifact@d59a9c15fec3fdb7c9adf09464124d00f9c11415 # main
        with:
          name: ${{ needs.openvino_download.outputs.ov_artifact_name }}
          path: ${{ env.INSTALL_DIR }}
          merge-multiple: true

      - name: Install OpenVINO Python wheel from pre-built artifacts
        run: |
          python3 -m pip install openvino==${{ needs.openvino_download.outputs.ov_version }} ${{ needs.openvino_download.outputs.ov_wheel_source }}
        working-directory: ${{ env.INSTALL_DIR }}
```

有关可用输入和输出的详细信息，请参阅 action 定义：
[.github/actions/openvino_provider/action.yml](../../../../.github/actions/openvino_provider/action.yml)。

另请参阅 openvino_tokenizers 仓库中的真实示例：
* [构建产物下载 job](https://github.com/openvinotoolkit/openvino_tokenizers/blob/77f7abc0a900b2f189f397576a1f03aa9ffab383/.github/workflows/linux.yml#L31-L55)
* [构建产物使用（Python）](https://github.com/openvinotoolkit/openvino_tokenizers/blob/77f7abc0a900b2f189f397576a1f03aa9ffab383/.github/workflows/linux.yml#L265-L276)