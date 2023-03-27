# 强制提交模板

> 原文：<https://itnext.io/enforcing-commit-templates-8cf3dbfe2510?source=collection_archive---------2----------------------->

![](img/a5a0da4bb28c82fc1bdf96c2850365c5.png)

照片由[像素](https://www.pexels.com/photo/adult-american-football-athlete-audience-209954/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)的[皮克斯拜](https://www.pexels.com/@pixabay?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)拍摄

很多时候，公司会有一个标准的提交消息模板。类似“发行号:短信，详情”之类的。就像编码风格一样，这些有助于使历史清晰易读。但是就像编码风格一样，如果不强制执行，它们就没有任何意义。

典型的标准提交消息可能是:

> 您的 git 提交应该以一个简短的单行开始，以:
> 
> DE:针对缺陷，后跟缺陷号
> 
> 美国:对于新功能的用户故事，后跟故事编号

例如，一个好的提交应该是这样的:

> DE175:修复缺陷简短描述
> 
> 下面是对修复缺陷所做的变更的更长的描述。

您可以(也应该)在您的 git 服务器上执行这些规则。使用[预接收钩](https://help.github.com/en/enterprise/2.16/admin/developer-workflow/creating-a-pre-receive-hook-script)可以做到这一点。设置预接收挂钩相当容易。预接收挂钩将进行检查，以确保所有提交到服务器的提交都通过您拥有的任何提交消息规则。

如果他们没有呢？

git 服务器将拒绝来自客户端的推送。由客户端决定修复任何提交消息，然后再次尝试推送。

现在，如果违规行为在最前面，这并不太难。一个简单的 [commit amend](https://www.atlassian.com/git/tutorials/rewriting-history) 命令就可以解决这个问题。

```
git commit --amend
```

这将允许您快速轻松地修改上次提交。

但是如果违规的提交不是在头部，如果有不止一个违规的提交呢？

现在，用户需要执行一个[交互式 rebase](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History) 。

```
git rebase -i
```

这可能是一个冗长乏味的过程。如果提交非常旧，并且在头部和违规提交之间发生了合并，那么修复提交就更加困难。有时，我不得不放弃整个变更历史，简单地做一个[合并挤压](https://git-scm.com/docs/git-merge)来修复和删除任何违规的提交。显然，这并不理想，因为我已经丢失了违规分支上的所有提交。

但这些只是解决我的烂摊子的方法。

如果我一开始就阻止自己制造混乱呢？

![](img/ad83d7a718471ad588488b7b59a1ee55.png)

照片由 [Pixabay](https://www.pexels.com/@pixabay?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 从[像素](https://www.pexels.com/photo/board-game-business-challenge-chess-277052/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)拍摄

# Git 客户端挂钩

就像我们前面看到的服务器端 git 挂钩强制提交消息一样，我们可以在本地使用类似的机制来确保我们不会创建任何错误的提交。

Git 挂钩是在 git 执行的特定阶段执行的特殊程序。我们可以使用几种不同的挂钩，但出于我们的目的，我们将重点关注:

*   **commit-msg:** 用于检查消息文件后拒绝提交。

钩子本身必须是一个名为 commit-msg 的可执行文件。为了让事情变得更快，让我们只使用一个简单的 bash 脚本。

跑步时:

```
git commit
```

您实际上正在运行一个名为 git-commit 的独立程序。然后，这个程序将使用提交消息的内容创建一个临时文件。这个文件然后被传递到我们的 **commit-msg** 脚本中。如果脚本返回非零状态代码，则提交被视为无效并被拒绝。

下面是一个小的示例**提交消息**脚本

```
#!/bin/bashMSG_FILE=$1
FILE_CONTENT="$(cat $MSG_FILE)"# Initialize constants here
export REGEX='^(DE[0-9]+:|US[0-9]+|Merge) .+'
export ERROR_MSG="Commit message format must match regex \"${REGEX}\""if [[ $FILE_CONTENT =~ $REGEX ]]; then
 echo "Good commit!"
else
  echo "Bad commit \"$FILE_CONTENT\""
 echo $ERROR_MSG
 exit 1
fiexit 0
```

注意正则表达式，我们也允许以“Merge”开头的行。这是为了使默认的合并提交消息也有效。

## 自动前缀

我们可以通过让 git 根据分支名称自动为我们的任何提交加上适当的缺陷或故事的前缀来进一步发展这种策略。这样，我们就不必为跟踪者记住我们正在做的事情的细节，而只需专注于编写好的提交消息。

为此，我们需要使用另一个 git 挂钩:

> 准备提交消息
> 
> 这个钩子由 [git-commit](https://git-scm.com/docs/git-commit) 在准备默认日志消息之后、编辑器启动之前调用。
> 
> 它需要一到三个参数。
> 
> 第一个是包含提交日志消息的文件的名称。
> 
> 第二个是提交消息的来源，可以是:
> 
> `message`(如果给出了`-m`或`-F`选项)；
> 
> `template`(如果给出了`-t`选项或设置了配置选项`commit.template`)；
> 
> `merge`(如果提交是合并或者存在`.git/MERGE_MSG`文件)；
> 
> `squash`(如果存在`.git/SQUASH_MSG`文件)；
> 
> 或`commit`，后跟提交 SHA-1(如果给出了`-c`、`-C`或`--amend`选项)。
> 
> 如果退出状态不为零，`git commit`将中止。

[(Git Docs)](https://git-scm.com/docs/githooks)

这个脚本有点复杂，但请继续关注我。基本前提是，我们将修改传入的提交，并从当前分支名称中为问题编号添加前缀。

为此，我们遵循以下分支命名方案:

> $ { type }/$ { userid }/$ { issue # } _ $ { message }

例如:

> 缺陷/efenglu/de 253 _ bad 注释

同样，我们需要一个小的 bash 脚本:

```
#!/bin/bashMSG_FILE=$1
MSG_TYPE=$2
FILE_CONTENT="$(cat $MSG_FILE)"export REGEX='^(DE[0-9]+:|US[0-9]+:|Merge) .+'
# If commit is already good, take it
if [[ $FILE_CONTENT =~ $REGEX ]]; then
 exit 0
fi# Skip files
skip_list=`git rev-parse --git-dir`"/hooks/pre-commit.skip"
if [[ -f $skip_list ]]; then
  if grep -E "^$BRANCH$" $skip_list; then
    exit 0
  fi
fi# Get curent branch
BRANCH="$(git rev-parse --abbrev-ref HEAD)"# Create prefix based off parsing branch name
STORY=$(echo ${BRANCH} | awk 'match($0, /US[0-9]+|DE[0-9]+/) {print toupper(substr($0, RSTART, 1))  substr($0, RSTART + 1, RLENGTH - 1)}')# If unable to parse story information from branch abort with zero
# Our other commit hook will catch bad commits for exit 0
if [[ ! $STORY ]]; then
  exit 0
fi# Add story to commit in attempt to make it good
echo "Prepending branch information \"$STORY\"..."
echo "Message Type: $MSG_TYPE"
echo "$STORY: $FILE_CONTENT" > $MSG_FILEexit 0
```

现在当我从分支**缺陷/efenglu/de253** 运行时:

```
git commit -m "Sample message."
```

日志将包含:

```
DE253: Sample message.
```

## 控制脚本

现在，您很可能从具有潜在不同规则的多个不同存储库中检出。例如，我使用我们的公司存储库和公共 github。我不希望在我的公共 github 库上强制执行公司规则。

为此，我在提交挂钩的开头添加了一个复选标记:

```
#!/bin/bashif [[ $(git remote -v | grep github.com) ]]; then
  echo "Commit Rules NOT enforced"
  exit 0
else
  echo "Commit Rules enforced"  
fi
```

## 文件名检查

还有其他几个钩子可以扩展，并提供各种其他功能。我们还想强制文件名区分大小写。因为我们的开发人员使用不同的文件系统，有些不区分大小写(OSX)，有些不区分大小写(Linux)。有时 git 会因为文件名冲突而无法签出文件。

这对我们的开发人员产生了许多奇怪的错误，并且很难解决。

尽管我们将检查添加到了 git 存储库中，但我们也希望进行客户端检查。

*   **预提交:**验证将要提交的内容

```
#!/bin/bashif git rev-parse --verify HEAD >/dev/null 2>&1
then
 against=HEAD
else
 # Initial commit: diff against an empty tree object
 against=4b825dc642cb6eb9a060e54bf8d69288fbee4904
fi# Redirect output to stderr.
exec 1>&2TF=$(mktemp)
trap "rm -f $TF" 0 1 2 3 15
checkstdin() {
    sort -f | uniq -di > $TF
    cat $TF
    test -s $TF || return 0   # if $TF is empty, we are good
    echo "non-unique (after case folding) names found!" 1>&2
    cat $TF 1>&2
    return 1
}git ls-files | checkstdin || {
    echo "ERROR - file name collision, check case of names, stopping commit" 1>&2
    exit 1
}
```

![](img/47dea018f133d50d5d26b7c162adb20e.png)

照片由 [Pixabay](https://www.pexels.com/@pixabay?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 从[像素](https://www.pexels.com/photo/referee-raising-both-hands-163294/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)拍摄

# 设置

不幸的是，要让客户端使用客户端挂钩，它们必须由客户端来设置。

幸运的是，这是一个非常简单的一次性过程。只需克隆包含您的共享 git 客户端脚本的 repo。

然后启用 git 中的脚本:

```
git config --global core.hooksPath ~/git/git-hooks
```

# 示例代码

检查我的仓库，上面有完整的代码示例。

[](https://github.com/efenglu/example-git-client-hooks) [## efenglu/example-git-client-hooks

### 使用客户端钩子来执行提交编码标准的例子

github.com](https://github.com/efenglu/example-git-client-hooks)  [## 学习如何提高你的 Git 技能

### Git 挂钩的介绍性指南和资源。了解如何使用提交前挂钩、提交后挂钩、接收后挂钩…

githooks.com](https://githooks.com) [](https://git-scm.com/docs/githooks) [## Git - githooks 文档

### 钩子接收将要更新当前分支的尖端的提交。它可以用一个…

git-scm.com](https://git-scm.com/docs/githooks)