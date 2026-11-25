# 如何准备一个高质量的 PR

   OpenVINO 是开源项目，你可以直接为其代码做贡献。
   为此，请遵循以下 Pull Request 创建指南，让你的
   改动获得最高的合并机会。


## 高质量 Pull Request 的通用规则

* 创建你自己的仓库 Fork，并用它来创建 PR。
  避免在主仓库中创建改动分支。
* 为你的工作选择合适的基线分支，并在其基础上创建你自己的分支。
* 为你的分支、提交和 Pull Request 起有意义的名称和描述。
  这有助于后续跟踪改动。如果你的改动涉及某个特定组件，
  可以在 PR 名称中加前缀标明，例如：``[DOCS] PR name``。
* 遵循 [OpenVINO 代码风格指南](https://github.com/openvinotoolkit/openvino/blob/master/docs/dev/coding_style.md)。
* 保持 PR 精简——每个 PR 只解决一个问题。删除所有
  与 PR 无关的改动。
* 如果 PR 解决了某个 Issue，请[将 Pull Request 关联到该 Issue](https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue#manually-linking-a-pull-request-to-an-issue-using-the-pull-request-sidebar)。
* 为你的贡献写文档！如果你的改动可能影响用户使用
  OpenVINO 的方式，请在合适的文章中提供相关信息。你可以自己撰写，
  也可以联系 OpenVINO 文档贡献者，共同
  编写合适的内容。
* 每个 PR 都要填写 [PR 模板](./.github/pull_request_template.md)中的 AI 部分，
  并遵循 [AI 使用政策](./AI_USAGE_POLICY.md)。
* 对于进行中的工作或需要提前查看测试结果的情况，请使用 Draft PR。


## 确保改动质量

你的 Pull Request 会由 OpenVINO™ 的预提交检查自动测试，如果
可以合并，状态会标记为"绿色"。如果有任何构建失败，状态为"红色"，
你需要修复控制台日志中列出的问题。对 PR 分支的任何改动都会
自动触发检查，因此无需重新创建 PR，只需等待
更新后的结果。

无论自动测试结果如何，你都应确保改动的质量：

* 在本地测试你的改动：
  * 务必仔细检查你的代码。
  * 在本地运行测试，发现并修复潜在问题（从构建产物目录
    执行测试二进制文件，例如 ``<source dir>/bin/intel64/Release/ieFuncTests``）
* 创建 PR 之前，确保你的分支已与目标分支的最新状态
  同步（例如 git fetch upstream && git
  merge upstream/master）。


## 分支策略

* "master" 分支用于开发，是每个新发布版本的基础。
* 每个 OpenVINO 发布版本都有自己的分支：``releases/<year>/<release number>``。
* 每年最终的发布版本被视为长期支持（LTS）版本，
  即该版本保持活跃。
* 仅接受活跃分支的贡献，包括：
  * 用于未来发布版本的 "master" 分支，
  * 最近发布的版本（用于修复），
  * LTS 版本（自发布之日起两年内）。


## Pull Request 活跃度与评审流程

### 何为活跃

当 Pull Request（PR）有实质性进展时，才被视为**活跃**，例如：

* 针对评审反馈更新代码
* 通过技术讨论解决未决问题
* 重大返工或设计澄清

简单的提醒、"ping" 消息或琐碎更新（例如无改动的变基）不算实质性进展。


### 响应预期

评审反馈给出后：

* 贡献者应在 **21 天**内响应
* 若无响应，PR 可能被视为**不活跃**

这有助于确保评审资源集中在正在积极推进的贡献上。


### 陈旧 Pull Request

如果 PR 变为不活跃：

* 维护者可能会为其添加 `stale` 标签
* 可能会添加提醒评论，要求更新

这表示该 PR 需要关注才能继续评审。


### 关闭不活跃的 Pull Request

如果在标记为 stale 后 **7 天内**仍无实质性进展：

* PR 可能被**维护者关闭**

因不活跃而关闭 PR：

* 是**管理操作**，并非拒绝
* **不影响**之后重新提交该贡献


### 继续或恢复工作

如果 PR 因不活跃被关闭：

* 欢迎贡献者重新打开（如可行），或
* 提交新 PR 并引用之前的工作

一旦有新的进展，维护者将恢复评审。


### 维护者接手

为避免丢失有价值的贡献：

* 如果 PR 不活跃但仍有价值，维护者可以：
  * 直接在 PR 基础上继续工作，或
  * 基于原始贡献创建后续 PR

这种情况下，将保留对原作者的署名。


### 社区评审

OpenVINO 鼓励社区参与评审流程。

* 所有 PR 都对外部贡献者开放评审
* 合并决策时可能会考虑社区的建设性反馈

社区贡献非常宝贵，有助于提升评审的可扩展性。


## 需要更多帮助？请查阅以下文章

* [如何创建 Fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo)
* [安装 Git](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)
* 如果你想添加新示例，请参阅 C++/C/Python OV 示例贡献指南，
  并在新文件顶部添加许可证声明（C++ 示例、Python 示例）。