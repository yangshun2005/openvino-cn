# Agentic 工作流

OpenVINO CI 包含一组 **Agentic 工作流**——这类 GitHub Actions 工作流将控制权交给 AI 代理来执行调查或维护任务，而非运行固定的 shell 命令序列。目前共有三个，均专注于诊断 CI 失败：

* [`ci-doctor.md`](../../../../.github/workflows/ci-doctor.md) — **CI Doctor**，针对 Pull Request 的按需调查器。
* [`ci-doctor-mq.md`](../../../../.github/workflows/ci-doctor-mq.md) — **CI Doctor（合并队列）**，针对合并队列失败的自动调查器。
* [`ci-doctor-post-commit.md`](../../../../.github/workflows/ci-doctor-post-commit.md) — **CI Doctor（提交后）**，针对提交后（push）失败的自动调查器，仅收集并报告（从不重跑或重新排队流水线）。
* [`ci-doctor-remediation.md`](../../../../.github/workflows/ci-doctor-remediation.md) — **CI Doctor（每周修复）**，定时维护任务，以只读方式消费两个 Doctor 的知识库，最多开启 3 个草稿修复 Pull Request，并生成一个 Markdown 报告构建产物。

本文档说明它们是什么、如何构建与调用、各自遵循的算法，以及它们共享的 [`.github/workflows/shared`](../../../../.github/workflows/shared/agentic-workflows) 中的可复用任务。

## 目录

* [什么是 Agentic 工作流](#what-is-an-agentic-workflow)
  * [gh-aw 框架](#the-gh-aw-framework)
  * [源文件与编译后的 lock 文件](#source-files-and-compiled-lock-files)
* [工作流一览](#workflows-at-a-glance)
* [CI Doctor（Pull Request，按需）](#ci-doctor-pull-request-on-demand)
* [CI Doctor — 合并队列（自动）](#ci-doctor--merge-queue-automatic)
* [CI Doctor — 提交后（自动）](#ci-doctor--post-commit-automatic)
* [共享可复用任务与提示词片段](#shared-reusable-jobs-and-prompt-fragments)
  * [共享任务与步骤](#shared-jobs-and-steps)
  * [共享提示词片段](#shared-prompt-fragments)
* [设置与基础设施](#setup-and-infrastructure)
  * [日志预下载与缓存](#log-pre-download-and-caching)
  * [Repo-memory 与模式数据库](#repo-memory-and-the-pattern-database)
  * [模式匹配与复发检测](#pattern-matching-and-recurrence-detection)
  * [自动重跑集成](#automated-re-run-integration)
  * [密钥与权限](#secrets-and-permissions)
* [维护工作流](#maintaining-the-workflows)
* [另见](#see-also)

## 什么是 Agentic 工作流

传统工作流描述*确切*要运行什么：检出代码、安装依赖、运行脚本。**Agentic 工作流**则用自然语言描述*目标*和*协议*，将执行委托给 AI 代理。代理决定调用哪些工具——读取日志、搜索仓库、查询 GitHub API——以完成任务，并受工作流中写入的护栏约束。

在 OpenVINO 中，所有 Agentic 工作流都是 **CI 失败诊断器**：读取失败流水线的日志，定位根因，分类失败，并生成可操作的报告（PR 评论或 Microsoft Teams 通知）。

### gh-aw 框架

这些工作流使用 [GitHub Agentic Workflows（`gh-aw`）](https://github.github.com/gh-aw/introduction/overview/) 编写。`gh-aw` 工作流是一个 Markdown 文件，包含 YAML frontmatter 块和自然语言正文：

* **Frontmatter** 的配置方式与普通 GitHub Actions 工作流相同——`on:` 触发器、`permissions:`、`concurrency:`，外加 `gh-aw` 特有键：
  * `engine:` — 代理运行时（`copilot`）和 `model:`（`claude-sonnet-5`）。
  * `network:` — 由代理周围的防火墙代理执行的网络策略。
  * `tools:` — 代理可使用的工具服务器（例如 GitHub MCP 服务器工具集和 `repo-memory` 存储）。
  * `safe-outputs:` — 代理唯一允许产生的副作用。代理不能直接发表评论或发送通知；它发出结构化的*安全输出*，由受信任的非代理任务执行。
  * `imports:` — 从 [`.github/workflows/shared`](../../../../.github/workflows/shared/agentic-workflows) 引入的共享片段：步骤/任务片段*以及*参数化提示词片段（见[共享可复用任务与提示词片段](#shared-reusable-jobs-and-prompt-fragments)）。
* **正文**是代理的提示词：任务、调查协议、输出格式和护栏（例如"最多读取 10 个源文件"或"绝不写入知识库"）。两个自动 Doctor 的大部分提示词从共享片段导入；工作流文件本身的正文只包含工作流特有的补充内容。

### 源文件与编译后的 lock 文件

`gh-aw` 不直接运行 `.md` 文件。它将每个 `.md` **编译**为扩展名为 `.lock.yml` 的标准 GitHub Actions 工作流：

| 源文件（编辑此文件） | 编译产物（生成） |
| --- | --- |
| [`ci-doctor.md`](../../../../.github/workflows/ci-doctor.md) | [`ci-doctor.lock.yml`](../../../../.github/workflows/ci-doctor.lock.yml) |
| [`ci-doctor-mq.md`](../../../../.github/workflows/ci-doctor-mq.md) | [`ci-doctor-mq.lock.yml`](../../../../.github/workflows/ci-doctor-mq.lock.yml) |
| [`ci-doctor-post-commit.md`](../../../../.github/workflows/ci-doctor-post-commit.md) | [`ci-doctor-post-commit.lock.yml`](../../../../.github/workflows/ci-doctor-post-commit.lock.yml) |

GitHub Actions 实际执行的是 `.lock.yml` 文件。它是**自动生成的——不要手动编辑**。修改 `.md` 源文件（或任何导入的共享文件）后，重新生成 lock 文件：

```bash
gh aw compile
```

详见[维护工作流](#maintaining-the-workflows)。

## 工作流一览

| | **CI Doctor** | **CI Doctor — 合并队列** | **CI Doctor — 提交后** |
| --- | --- | --- | --- |
| 源文件 | [`ci-doctor.md`](../../../../.github/workflows/ci-doctor.md) | [`ci-doctor-mq.md`](../../../../.github/workflows/ci-doctor-mq.md) | [`ci-doctor-post-commit.md`](../../../../.github/workflows/ci-doctor-post-commit.md) |
| 触发方式 | 按需：PR 上的 `/ci-doctor` 评论 | 自动：受监控工作流完成 `merge_group` 运行且结果为 `failure` | 自动：受监控工作流完成 `push` 运行且结果为 `failure` |
| 范围 | PR head 提交上的**所有**失败流水线 | **单个**失败的合并队列运行 | **单个**失败的提交后运行 |
| 知识库 | 读取 MQ 模式数据库（只读） | **写入** MQ 模式数据库（调查 + 模式） | **写入**自己的提交后模式数据库（调查 + 模式） |
| 输出 | 一条汇总 PR 评论 | Microsoft Teams 通知（+ 可选 PR 评论、重跑、重新排队） | 仅 Microsoft Teams 通知 |
| 修复 | 仅报告 | 可重跑失败任务、将 PR 重新加入合并队列、升级复发失败 | 仅报告（从不重跑或重新排队） |
| 引擎 / 模型 | `copilot` / `claude-sonnet-5` | `copilot` / `claude-sonnet-5` | `copilot` / `claude-sonnet-5` |

## CI Doctor（Pull Request，按需）

**是什么** 一个按需助手，调查 Pull Request 上*所有*当前失败的 CI 流水线，并在单条评论中汇总。

**如何调用** 用户在 Pull Request 上发布 `/ci-doctor` 斜杠命令评论。frontmatter 中的 `if:` 守卫将调用限制为明确的维护者允许列表，因此任意用户无法触发代理运行：

```yaml
on:
  slash_command:
    name: ci-doctor
    events: [pull_request_comment]

if: ${{ contains(fromJSON('["akashchi","mryzhov","akladiev","ababushk","dorloff","as-suvorov"]'), github.actor) }}
```

**功能 / 优势**

* 将 PR head 提交上*所有*失败流水线聚合到**一条**评论中，作者无需逐个打开失败的运行。
* 对每个失败报告类别、失败任务、关键日志摘录、根因说明和具体补救措施。
* 交叉引用合并队列知识库（只读），标记已知的复发问题，并优先采用此前解决过该问题的补救措施。
* 有界且安全：只产生一条 PR 评论（`safe-outputs.add-comment: max: 1`），源码检查有上限（≤ 10 个文件、≤ 5 次搜索、优先 PR diff）。

**算法。** 正文定义五阶段协议：

1. **初步分诊** — 读取预下载的失败流水线摘要；若无失败（或命令未在 PR 上运行），以 `noop` 停止。
2. **深度日志分析** — 对每个失败流水线，从预定位的错误*提示*开始，先读取每个提示周围约 50 行上下文，再加载完整日志。
3. **历史上下文** — 将每个失败与合并队列模式数据库匹配，检测已知复发问题（严格只读）。
4. **根因调查** — 将失败归入七个类别之一并定位，受源码检查护栏约束。
5. **报告** — 通过 `add_comment` 安全输出发布一条汇总评论，包含可见的概览表，以及每个流水线诊断、补救和日志证据的独立默认折叠 `<details>` 区块；否则 `noop`/`missing_data`。读者只需展开需要的流水线，无需滚动浏览所有日志摘录。

## CI Doctor — 合并队列（自动）

**是什么** 一个自动、常驻的调查器，响应合并队列 CI 失败，构建持久化的失败模式知识库，并在同一失败持续复发时升级。

**如何触发** 它在固定监控工作流列表（Linux/Windows/Android/ARM/RISC-V/… 验证流水线）的 `workflow_run: completed` 事件上运行，且仅当运行为**合并队列**运行且**失败**时继续：

```yaml
on:
  workflow_run:
    workflows: ["Linux (Ubuntu 22.04, Python 3.11)", "Windows (VS 2022, Python 3.11, Release)", ...]
    types: [completed]
    branches: ["gh-readonly-queue/**"]

if: ${{ github.event_name == 'workflow_dispatch' || (github.event.workflow_run.conclusion == 'failure' && github.event.workflow_run.event == 'merge_group') }}
```

`gh aw compile` 要求正向的 `workflow_run.branches` 范围（检查 `CTR-021`）。合并队列运行始终在本仓库中 GitHub 创建的 `gh-readonly-queue/<base>/pr-<N>-<sha>` 分支上执行，包括来自 Fork 的 PR，因此 `gh-readonly-queue/**` 仍覆盖所有合并队列失败。

还提供 `workflow_dispatch` 入口（带 `run_id` 或 `link` 输入）用于手动测试。

**功能 / 优势**

* **根因分析**单个失败的合并队列运行：类别、失败任务、关键错误、说明、置信度。
* **知识构建**：每次调查写入持久化的跨运行知识库，使复发失败随时间累积统计（`count`、`first_seen`、`last_seen`）。
* **升级**：当同一失败签名在 **12 小时内出现 3 次或以上**时，发送第二条更高可见性的 Teams 告警。
* **自动修复**：当失败为瞬时性时，代理调用单个 `remediate_transient_failure` 安全输出；该任务解析 PR 的实时合并队列状态，自行决定仅重跑失败任务（PR 仍在队列）或将 PR 重新加入合并队列（PR 已掉出）。代理从不推理队列状态。
* **重跑模式提案**：对于有已验证 `rerun_search_string` 候选的瞬时失败，代理调用 `propose_rerun_pattern`，向 `master` 开启一个**人工审查 Pull Request**，将其加入 `errors_to_look_for.json`。工作流重跑器仅在该 PR 合并后使用该字符串——见[自动重跑集成](#automated-re-run-integration)。

**算法** 正文定义七阶段协议：

1. **触发检测与分诊** — 验证运行为失败的 `merge_group` 运行；否则退出。
2. **深度日志分析** — 提示优先的日志读取，与 CI Doctor 相同。
3. **历史上下文** — 在先前调查中搜索相似签名。
4. **根因调查** — 归入七个类别之一，受源码检查护栏约束。
5. **模式存储** — 写入调查记录，更新仅追加的调查索引，读-改-写每签名模式文件（均按 JSON Schema 校验）。见 [Repo-memory 与模式数据库](#repo-memory-and-the-pattern-database)。
   * **5.5 复发检查** — 若最近 12 小时内 ≥ 3 次命中，准备复发失败升级。
6. **报告** — 撰写调查报告。
7. **输出校验** — 在发出前规范化安全输出载荷（所有看似数字的字段必须为字符串）。

运行始终以调用恰好一个（或有效组合）安全输出结束：`notify_teams`、`notify_teams_recurring`、`add_comment`、`remediate_transient_failure`、`propose_rerun_pattern`、`noop` 或 `missing_data`。

## CI Doctor — 提交后（自动）

**是什么** 一个自动、常驻的调查器，响应**提交后**（push）CI 失败，构建自己的持久化失败模式知识库，并将每个失败报告到 Microsoft Teams。与合并队列 Doctor 不同，它**从不重跑或重新排队**任何流水线——唯一目的是收集问题并报告。

**如何触发** 它在与合并队列 Doctor 相同的固定监控工作流列表的 `workflow_run: completed` 事件上运行，且仅当运行为**push**（提交后）运行且**失败**时继续：

```yaml
on:
  workflow_run:
    workflows: ["Linux (Ubuntu 22.04, Python 3.11)", "Windows (VS 2022, Python 3.11, Release)", ...]
    types: [completed]

if: ${{ github.event_name == 'workflow_dispatch' || (github.event.workflow_run.conclusion == 'failure' && github.event.workflow_run.event == 'push') }}
```

还提供 `workflow_dispatch` 入口（带 `run_id` 或 `link` 输入）用于手动测试。

**功能 / 优势**

* **根因分析**单个失败的提交后运行：类别、失败任务、关键错误、说明、置信度——与合并队列分析相同。
* **知识构建**：每次调查写入自己的持久化跨运行知识库（在 `memory/ci-doctor-post-commit` 分支的 `post-commit/` 子目录下），使复发提交后失败随时间累积统计（`count`、`first_seen`、`last_seen`），与合并队列数据分开保存。
* **仅报告**：只发出一个 `notify_teams` 安全输出，`source: "post_commit"`，别无其他——无 PR 评论、无重跑、无重新排队、无复发升级。

**算法** 正文遵循与合并队列 Doctor 相同的阶段，减去修复和复发升级：触发检测与分诊（验证失败的 `push` 运行）、提示优先日志分析、历史上下文、受源码检查护栏约束的根因调查、模式存储（调查记录 + 仅追加索引 + 每签名模式文件，均经 Schema 校验）、报告和输出校验。其模式 Schema 去掉 `rerun_search_string` 字段（无重跑集成），并跟踪 `affected_commits` 而非 `affected_prs`。运行始终以调用恰好一个 `notify_teams`（`source: "post_commit"`）、`noop` 或 `missing_data` 结束。

## CI Doctor — 每周修复（自动）

**是什么** 一个每周维护工作流，*消费*两个自动 Doctor 构建的知识库，而非诊断新失败。它审查过去一周记录的复发失败，为每个 Issue 提出具体修复，为可代码修复的开启草稿 Pull Request，并上传覆盖所有已识别 Issue 的 Markdown 报告。对知识库严格**只读**，从不重跑或重新排队任何流水线。

**如何触发** 每周 `schedule`（模糊 `weekly on monday`）和通过 `workflow_dispatch` 按需（可选 `days` 回溯输入，默认 `7`）。无 `workflow_run` 触发，无参与者允许列表——它是定时维护任务。

**功能 / 优势**

* **每周综合**：不逐个失败处理，而是查看两个 Doctor 整周累积的模式，按 `signature_hash` 分组，并按复现 `count`、新近度和广度排序。
* **可操作修复**：为每个 Issue 决定具体修复，并分类为可代码修复与否。
* **自动修复**：对影响最大的可代码修复 Issue，开启**最多 3 个草稿 Pull Request**（每个 Issue 一个 PR），每个以 `akashchi` 为审查者和指派者，并带 `agentic-workflows` + `automated-fix` 标签，由工作流自动应用。由于修复常涉及 `.github/` CI 基础设施，`create-pull-request` 配置为 `protected-files: allowed`。
* **持久记录**：每个已识别 Issue——无论是否可代码修复——写入 Mark