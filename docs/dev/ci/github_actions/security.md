# GitHub Actions 工作流安全最佳实践

我们应遵循几个简单步骤，确保工作流不易受到常见攻击。

## 调整 `GITHUB_TOKEN` 权限

使用 `permissions` 键确保 `GITHUB_TOKEN` 为每个作业配置最小权限。

从相对安全的权限开始：

```yaml
permissions: read-all
```

如需更多权限，尽量在作业级别声明，例如：

```yaml
jobs:
  stale:
    runs-on: ubuntu-latest

    # GITHUB_TOKEN 对 `stale` 作业仅拥有这些权限
    permissions:
      issues: write
      pull-requests: write

    steps:
      - uses: actions/stale@f7176fd3007623b69d27091f9b9d4ab7995f0a06

```

另请参阅 [GitHub 文档](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token)。

## 缩小环境变量作用域

环境变量应尽量在步骤级别声明（例如变量仅在该步骤使用）。仅当多个步骤使用变量时才放在作业级别，仅当大多数步骤使用变量时才放在工作流级别。

示例来自 [GitHub 官方文档](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/store-information-in-variables)：

```yaml
name: Greeting on variable day

on:
  workflow_dispatch

# 工作流级别变量。避免使用。
env:
  DAY_OF_WEEK: Monday

jobs:
  greeting_job:
    runs-on: ubuntu-latest
    # 作业级别变量
    env:
      Greeting: Hello
    steps:
      - name: "Say Hello Mona it's Monday"
        run: echo "$Greeting $First_Name. Today is $DAY_OF_WEEK!"
        # 步骤级别变量。优先采用此方式
        env:
          First_Name: Mona

```

## 避免使用 `pull_request_target`

**切勿**在工作流中使用 `pull_request_target` 触发事件。如需使用 `pull_request_target`，请先联系 OpenVINO GitHub Actions 任务组成员。另请参阅 [GitHub 博客文章](https://securitylab.github.com/resources/github-actions-preventing-pwn-requests/)。

## 正确处理密钥

**切勿**在 GitHub Actions 工作流中硬编码明文密钥。如需使用密钥，请先联系 OpenVINO GitHub Actions 任务组成员。

## 谨慎处理用户输入

大多数 GitHub 上下文变量来自用户输入，应视为不可信且可能恶意。可采用以下策略降低风险：
- 不使用内联脚本，而是创建 action 并将变量作为参数传入
- 将值放入步骤的环境变量，在脚本中使用该变量

更多细节见[这篇](https://securitylab.github.com/resources/github-actions-untrusted-input/)博客文章。

## 固定 GitHub Actions 版本

使用第三方 action 时，用提交哈希而非标签固定版本，以保护工作流免受潜在的供应链攻击。

例如，不要这样写：

```yaml
uses: actions/checkout@v4.2.2
```

而应这样写：

```yaml
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

## 延伸阅读
遵循 [GitHub 官方通用建议](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)