# 如何对提交进行签署

我们要求 Pull Request 中的每个提交都带有以下格式的签署提交信息。

```
This is a commit message.

Signed-off-by: Author Name <author-email@example.com>
```

## 如何对新提交进行签署

在本地 Git 环境中，可以通过手动方式（以文本形式）或使用 `git commit` 命令的 **-s** 标志为提交添加签署信息，例如：

`git commit -s -m "My commit message"`

为避免每次提交都手动添加该标志，建议通过以下步骤设置 Git Hook：

1.	进入 `<openvino repository root>/.git/hooks` 文件夹。
2.	打开 `prepare-commit-msg.sample` 文件并粘贴以下内容：

```
#!/bin/sh

COMMIT_MSG_FILE=$1
COMMIT_SOURCE=$2
SHA1=$3

NAME=$(git config user.name)
EMAIL=$(git config user.email)

if [ -z "$NAME" ]; then
    echo "empty git config user.name"
    exit 1
fi

if [ -z "$EMAIL" ]; then
    echo "empty git config user.email"
    exit 1
fi

git interpret-trailers --if-exists doNothing --trailer \
    "Signed-off-by: $NAME <$EMAIL>" \
    --in-place "$1"
```

3.	将文件保存为 `prepare-commit-msg`（去掉 .sample 扩展名）。
4.	使文件可执行（在 Linux / Git Bash 上：`chmod +x <openvino repository root>/.git/hooks/prepare-commit-msg`）。

**注意**：无论采用哪种签署方式，请先确保已在 Git 中配置用户名和邮箱地址：

```
git config user.name 'FIRST_NAME LAST_NAME'
git config user.email 'MY_EMAIL@example.com'
```

### 对基于 Web 的提交进行签署

要启用通过 GitHub Web 界面所做提交的自动签署，请确保在你的 OpenVINO 仓库 Fork 的 Settings 菜单中勾选了
[Require contributors to sign off on web-based commits](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/managing-the-commit-signoff-policy-for-your-repository#enabling-or-disabling-compulsory-commit-signoffs-for-your-repository)
设置。

## 如何对历史中的旧提交进行签署

如果忘记为最近一次提交添加签署，可以修正该提交并强制推送到 GitHub：

```
git commit --amend --signoff
```

如需对更早的提交进行签署，请使用交互式变基，编辑未签署的提交，并对每个提交执行
`git commit --amend --signoff`。但请注意，如果其他人已基于该分支中的提交开始工作，此操作将重写历史，可能给协作者带来问题。