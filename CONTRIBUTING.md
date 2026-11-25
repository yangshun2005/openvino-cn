# 为 OpenVINO 做贡献

## 如何为 OpenVINO 项目做贡献

OpenVINO™ 始终在寻找改进的机会，你的贡献在这一过程中起着重要作用。你可以通过多种方式让产品变得更好。

# 目录
1. [贡献形式](#贡献形式)
2. [技术指南](#技术指南)


## 贡献形式

另请参阅：[AI 使用政策](./AI_USAGE_POLICY.md)。
如果你在任何贡献活动（代码、文档、Issue、讨论或 PR 沟通）中使用 AI 辅助，请阅读并遵守该政策。

### 提供反馈

   * **报告 Bug / Issue**
     如果你在 OpenVINO 或其组件中遇到异常行为，可以在 GitHub Issue 跟踪器中
     [创建新 Issue](https://github.com/openvinotoolkit/openvino/issues)。

   * **提出新功能 / 改进建议**
     如果你对改进 OpenVINO 有建议或想分享想法，可以发起新的
     [GitHub Discussion](https://github.com/openvinotoolkit/openvino/discussions)。
     如果你的想法已经比较明确，也可以创建
     [功能请求 Issue](https://github.com/openvinotoolkit/openvino/issues/new?assignees=octocat&labels=enhancement%2Cfeature&projects=&template=feature_request.yml&title=%5BFeature+Request%5D%3A+)
     两种情况下，请提供详细描述，包括用例、收益和潜在挑战。
     如果你的观点与产品愿景特别契合，将被纳入开发路线图。
     用户反馈对 OpenVINO 的开发至关重要，即使你的输入未立即被优先处理，
     也可能在后续被采用，或由社区自行推进，而不受官方路线图限制。


### 贡献代码变更

   * **修复 Bug 或开发新功能**
     如果你想帮助改进 OpenVINO，请从
     [GitHub Issue 跟踪器](https://github.com/openvinotoolkit/openvino/issues) 中选择一个 Issue，
     并[创建 Pull Request](./CONTRIBUTING_PR.md) 来处理它。如果你想从简单的任务开始，
     请查看[首次贡献章节](#3-开始处理你的-good-first-issue)。
     如果你想开发的功能较复杂或报告者描述不够清晰，
     最好先与 OpenVINO 开发者[讨论](https://github.com/openvinotoolkit/openvino/discussions)。
     创建新 PR 前，请确认没有其他人已在处理。如果有人在做，你仍可在与其协调后提供帮助。

     重要的是，开始工作前务必确认该变更尚未被实现！
     你可以用最新的 master 分支构建 OpenVINO，确认是否仍需要你的改动。另外，不要处理仅影响旧的非 LTS 版本（如 2022.2）的 Issue。

   * **开发新的设备插件**
     由于计算设备市场不断演进，OpenVINO 始终愿意扩展对新硬件的支持。如果你想在目前不支持的设备上运行推理，
     可在[插件开发者指南](https://docs.openvino.ai/2026/documentation/openvino-extensibility/openvino-plugin-library.html)中了解如何为其开发新插件。


### 改进文档

   * **OpenVINO 开发者文档**完全包含在本仓库的
     [./docs/dev](https://github.com/openvinotoolkit/openvino/tree/master/docs/dev) 文件夹中。

   * **用户文档**由多个来源构建并发布在
     [docs.openvino.ai](https://docs.openvino.ai/)，推荐在该处阅读这些文档。本仓库中维护的文件仅用于编辑目的。

   * 帮助改进文档最简单的方式是审阅现有文章并提供反馈。无论你发现错误、看到改进文本的可能，
     还是认为应补充更多信息，都可以联系任意文档贡献者讨论潜在修改。

     你也可以直接创建 Pull Request，遵循[编辑指南](./CONTRIBUTING_DOCS.md)。


### 推广和支持 OpenVINO

   * **推广 OpenVINO**
     文章、教程、博客、演示、视频以及任何其他参与 OpenVINO 社区的方式都是受欢迎的贡献。如果你在各种社交平台上讨论
     或介绍 OpenVINO，就是在提升 AI 爱好者对产品的认知，让更多人发现该工具包。如果你在制作此类社区内容时需要帮助，
     欢迎联系 OpenVINO 开发者。

   * **帮助其他社区成员**
     如果你是经验丰富的 OpenVINO 用户并希望提供帮助，可以随时与社区分享你的专业知识。查看 GitHub Discussions 和
     Issues，看看能否帮助他人。

## 技术指南

本节列出搭建环境、在本地构建 OpenVINO 以及为特定组件运行测试所需的全部步骤。当你刚挑选了一个 Good First Issue 并想知道如何开始时，这里是理想的起点。

请记住我们随时提供帮助——**如有不清楚之处，不要犹豫，直接询问开发团队**。这类问题有助于我们持续改进文档。

### 1. 前置条件

可以从以下链接开始：
- [什么是 OpenVINO？](https://github.com/openvinotoolkit/openvino#what-is-openvino-toolkit)
- [OpenVINO 架构](https://github.com/openvinotoolkit/openvino/blob/master/src/docs/architecture.md)
- [用户文档](https://docs.openvino.ai/)
- [关于为 OpenVINO 做贡献的博客文章](https://medium.com/openvino-toolkit/how-to-contribute-to-an-ai-open-source-project-c741f48e009e)
- [挑选一个 Good First Issue](https://github.com/orgs/openvinotoolkit/projects/3)
- 查看 [Intel DevHub Discord 服务器](https://discord.gg/7pVRxUwdWG) —— 参与讨论、提问并与 OpenVINO 开发者交流

### 2. 构建项目

要构建项目，请遵循[针对你所使用操作系统的构建说明](https://github.com/openvinotoolkit/openvino/blob/master/docs/dev/build.md)。

### 3. 熟悉你将处理的组件

选择与你的 Good First Issue 相关的组件。你可以运行测试确认其工作正常。

##### API
- [C API](https://github.com/openvinotoolkit/openvino/tree/master/src/bindings/c)
- [Core](https://github.com/openvinotoolkit/openvino/tree/master/src/core)
- [Python API](https://github.com/openvinotoolkit/openvino/tree/master/src/bindings/python)
- [Node.js API](https://github.com/openvinotoolkit/openvino/tree/master/src/bindings/js/node)

##### 前端
- [IR 前端](https://github.com/openvinotoolkit/openvino/tree/master/src/frontends/ir)
- [ONNX 前端](https://github.com/openvinotoolkit/openvino/tree/master/src/frontends/onnx)
- [PaddlePaddle 前端](https://github.com/openvinotoolkit/openvino/tree/master/src/frontends/paddle)
- [PyTorch 前端](https://github.com/openvinotoolkit/openvino/tree/master/src/frontends/pytorch)
- [TensorFlow 前端](https://github.com/openvinotoolkit/openvino/tree/master/src/frontends/tensorflow)

##### 插件
- [Auto 插件](https://github.com/openvinotoolkit/openvino/blob/master/src/plugins/auto)
- [CPU 插件](https://github.com/openvinotoolkit/openvino/blob/master/src/plugins/intel_cpu)
- [GPU 插件](https://github.com/openvinotoolkit/openvino/blob/master/src/plugins/intel_gpu)
- [NPU 插件](https://github.com/openvinotoolkit/openvino/blob/master/src/plugins/intel_npu)
- [Hetero 插件](https://github.com/openvinotoolkit/openvino/blob/master/src/plugins/hetero)
- [模板插件](https://github.com/openvinotoolkit/openvino/tree/master/src/plugins/template)

##### 工具
- [基准测试工具](https://github.com/openvinotoolkit/openvino/tree/master/tools/benchmark_tool)
- [OpenVINO 模型转换器](https://github.com/openvinotoolkit/openvino/tree/master/tools/ovc)

##### 其他
- [文档](https://github.com/openvinotoolkit/openvino/blob/master/CONTRIBUTING_DOCS.md)

### 3. 开始处理你的 Good First Issue

要开始贡献，请从 [Good First Issues 看板](https://github.com/orgs/openvinotoolkit/projects/3) 中挑选一个任务。

要认领 Issue，只需在所选 Issue 中留下包含 `.take` 命令的评论。
根据 Issue 描述并在本地构建 OpenVINO 来完成任务。

你可以随时向“联系人”部分标记的用户寻求帮助！
访问 [Intel DevHub Discord 服务器](https://discord.gg/7pVRxUwdWG)，在专门支持 Good First Issue 的频道中提问。

### 4. 提交包含变更的 PR

遵循我们的 [Good Pull Request 指南](https://github.com/openvinotoolkit/openvino/blob/master/CONTRIBUTING_PR.md)。请记得[将 Pull Request 关联到](https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue#manually-linking-a-pull-request-to-an-issue-using-the-pull-request-sidebar)它所处理的 Issue。

### 5. 等待评审

我们会尽快评审你的 Pull Request 并提供反馈。当你的变更通过自动测试验证并获得维护者批准后，即可合并。

## 许可证

向 OpenVINO 项目做贡献，即表示你同意你的贡献将按 [LICENSE](./LICENSE) 文件中声明的条款进行许可。