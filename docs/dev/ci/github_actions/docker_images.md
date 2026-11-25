# Docker 镜像

OpenVINO GHA CI 中的大多数工作流使用[带动态生成运行器的自托管机器](./runners.md)来执行任务。

为避免运行器和机器损坏，工作流使用各种 Docker 镜像，为自托管机器提供额外一层保护。

每个任务通过 `container` 键指定 Docker 镜像。语法参考见 [GHA 文档](https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container)。

以下是 [`ubuntu_22.yml`](./../../../../.github/workflows/ubuntu_22.yml) 工作流中 `Build` 任务的示例：
```yaml
Build:
  ...
  needs: [Docker, Smart_CI]
  uses: ./.github/workflows/job_build_linux.yml
  with:
    runner: aks-linux-16-cores-32gb
    image: ${{ fromJSON(needs.docker.outputs.images).ov_build.ubuntu_22_04_x64 }}
  ...
```

镜像引用由 `Docker` 任务通过 `handle_docker` action 生成。可复用的构建工作流将该镜像应用到其 `container` 配置，并定义挂载。

此外，你可以使用 `volumes` 键让缓存在 Docker 容器中可用。详情参见[缓存](./caches.md)页面。

## 可用的 Docker 镜像

工作流中的任务根据各自需求使用 Docker 镜像。

由于自托管机器托管在 [Microsoft Azure](https://azure.microsoft.com/en-us)，建议使用托管在 [Azure 容器注册表 (ACR)](https://azure.microsoft.com/en-us/products/container-registry) 实例中的 Docker 镜像。

OpenVINO GHA CI 使用的 ACR 为 `openvinogithubactions.azurecr.io`。

使用自有容器注册表的优缺点如下：
* 优点：
  * 镜像拉取无限制：Docker Hub 对镜像拉取有[限制](https://docs.docker.com/docker-hub/download-rate-limit/)
  * 拉取速度快
* 缺点：
  * 使用前需向注册表填充所需镜像
    * 注册表不会自动镜像 Docker Hub 上的镜像
    * 必须手动将所需镜像添加到注册表

可用 Docker 镜像的命名模式为：`openvinogithubactions.azurecr.io/<image-name>:<image-tag>` 或 `openvinogithubactions.azurecr.io/library/<image-name>:<image-tag>`

OpenVINO ACR 上的大多数镜像要么是 Docker Hub 镜像的镜像副本，要么基于它们构建。

示例：
* `openvinogithubactions.azurecr.io/library/ubuntu:22.04` 对应 Docker Hub 的 `ubuntu:22.04`
* `openvinogithubactions.azurecr.io/nvidia/cuda:11.8.0-runtime-ubuntu20.04` 对应 Docker Hub 的 `nvidia/cuda:11.8.0-runtime-ubuntu20.04`

镜像镜像通过 [ACR 构件缓存规则](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-artifact-cache)自动完成，无需从公共 DockerHub 手动上传到 OpenVINO ACR 实例。

## 自定义 Docker 镜像与 handle_docker action

为优化工作流中安装依赖所需的时间，并便于在本地复现工作流步骤，我们为不同类型的验证创建了自定义 Docker 镜像并在工作流中使用。这些镜像的 Dockerfile 存储在 OpenVINO 仓库的 [`.github/dockerfiles`](./../../../../.github/dockerfiles) 文件夹中。Dockerfile 组织如下：

* [`.github/dockerfiles/ov_build`](./../../../../.github/dockerfiles/ov_build) - 包含不同操作系统构建工作流的环境配置（例如安装 [install_build_dependencies.sh](./../../../../install_build_dependencies.sh)）
* [`.github/dockerfiles/ov_test`](./../../../../.github/dockerfiles/ov_test) - 包含不同操作系统测试工作流的环境配置（例如安装 [scripts/install_dependencies/install_openvino_dependencies.sh](./../../../../scripts/install_dependencies/install_openvino_dependencies.sh)）

对这些 Dockerfile 的更改会通过自定义 action [handle_docker](./../../../../.github/actions/handle_docker) 在预提交中自动检查和应用。该 action 在工作流中于实际验证开始前，在名为 Docker 的独立任务中执行。该 action 检查 Pull Request 是否更改了 Dockerfile 或环境配置脚本，若有则触发受影响 Docker 镜像的构建，并用更新后的镜像进行验证。镜像以更改它们的 Pull Request ID 作为标签，必须通过修改 [.github/dockerfiles/docker_tag](./../../../../.github/dockerfiles/docker_tag) 中的 PR ID 来在 git 中更新该标签。一旦你更改了影响 Docker 环境的内容，该 action 会提示你执行此操作。

**重要**：如果你添加了新的环境配置脚本供 Dockerfile 使用，请将其路径添加到 [.github/labeler.yml](./../../../../.github/labeler.yml) 的 `category: docker_env` 键下（这将确保 handle_docker action 检测到该脚本的更改并自动应用），并将该脚本路径从 [.dockerignore](./../../../../.dockerignore) 中排除（这将确保 Docker 本身能检测到脚本文件）。

该 action 接受所需构建的镜像列表作为输入，并输出完全限定的 Docker 镜像引用，供工作流任务使用。

### 在工作流任务中使用自定义镜像

* 确保你的工作流中调用了 Docker 任务。将后续工作流中要使用的 Dockerfile 文件夹路径（一个或多个）传给 `handle_docker` action 的 `images` 参数。示例：
```yaml
  Docker:
    needs: Smart_CI
    runs-on: aks-linux-4-cores-16gb-docker-build
    container:
      image: openvinogithubactions.azurecr.io/docker_build:0.2
      volumes:
        - /mount:/mount
    outputs:
      images: "${{ steps.handle_docker.outputs.images }}"
    steps:
      - name: Checkout
        uses: actions/checkout@692973e3d937129bcbf40652eb9f2f61becf3332 # v4.1.7

      - uses: ./.github/actions/handle_docker
        id: handle_docker
        with:
          images: |
            ov_build/ubuntu_22_04_x64
            ov_build/ubuntu_22_04_x64_nvidia
            ov_test/ubuntu_22_04_x64
          registry: 'openvinogithubactions.azurecr.io'
          dockerfiles_root_dir: '.github/dockerfiles'
          changed_components: ${{ needs.smart_ci.outputs.changed_components }}
```
* 将 "Docker" 添加到将使用所需自定义镜像执行的任务的 `needs:` 块中，并将该任务的 `container.image` 键设置为指向从 `handle_docker` 输出中获取的 Docker 镜像，如下所示：
```yaml
  Build:
    needs: [Smart_CI, Docker]
    ...
    runs-on: aks-linux-16-cores-32gb
    container:
      image: ${{ fromJSON(needs.docker.outputs.images).ov_build.ubuntu_22_04_x64 }}
```
**注意**，使用自定义 Docker 镜像需要基于 Azure 的运行器（aks-***），因为只有此类运行器才能访问我们的 Azure 容器注册表。


## 如何选择镜像

Docker 镜像的选择取决于任务性质和所测试的配置。请查看 [.github/dockerfiles](./../../../../.github/dockerfiles) 中的镜像。通常，需要编译的任务选择 **ov_build** 镜像，其他任务选择 **ov_test** 镜像，如上一节所述。

如果没有现有镜像满足需求，且你需要一个安装了自有依赖的独立 Docker 镜像，可在 `.github/dockerfiles/<validation_type>/<platform>` 文件夹下添加新的 Dockerfile，并按上述方式在工作流中引用。

如果流水线不需要安装任何系统依赖，使用 DockerHub 上的普通镜像可能就足够了。此时，请使用我们私有 Azure 容器注册表中该 DockerHub 镜像的镜像副本。例如，如果你需要使用 `ubuntu:22.04` 镜像，本地拉取方式如下：
```
docker pull ubuntu:22.04
...
Status: Downloaded newer image for ubuntu:22.04
docker.io/library/ubuntu:22.04
```
注意最后一行以 `docker.io` 开头，将 `docker.io` 替换为 `openvinogithubactions.azurecr.io`，并在工作流中使用生成的镜像引用：
```yaml
  SomeJob:
    ...
    container:
      image: openvinogithubactions.azurecr.io/library/ubuntu:22.04
```
该镜像会自动从 DockerHub 镜像到我们的 Azure 容器注册表。


## 本地复现

自定义 Docker 镜像也可在本地构建，以在相同环境中复现 CI 结果：
1. 在本地机器上安装 [Docker](https://docs.docker.com/engine/install)（Linux 是最佳选择）。
2. 如果你在运行 Docker 构建的主机上使用代理，请将[代理](https://docs.docker.com/engine/cli/proxy/#configure-the-docker-client)添加到 `~/.docker/config.json`。
3. 克隆 openvino 仓库，进入仓库根目录，为所需镜像执行 `docker build`，例如：
```
cd _git/openvino
docker build -f .github/dockerfiles/ov_test/ubuntu_22_04_x64/Dockerfile -t my_local_ov_test_build .
```
4. 在构建的镜像中运行交互式 shell 会话：
```
docker run --entrypoint bash -it my_local_ov_test_build
```

有关 Docker 环境的任何帮助，请联系 CI 团队成员。