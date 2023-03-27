# 使用多种配置设置 Git

> 原文：<https://itnext.io/setup-git-with-multiple-configs-9b4111d6928c?source=collection_archive---------2----------------------->

![](img/b66a8db70046d535d724ac267e5b1246.png)

图片由 [suju](https://pixabay.com/users/suju-165106/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=2372148) 来自 [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=2372148)

我有几个公共回购和公司回购，我每天都在工作。我需要一种方法让 git 根据我的 git 存储库的来源使用不同的配置。

例如，公司存储库应该使用我的公司电子邮件和公司用户名。公共 GitHub 库应该使用我的个人邮箱和名字。

您可以通过在每个存储库中设置用户名和电子邮件来做到这一点:

```
git config user.name "My Name"
git config user.email "my@email.com"
```

但是这很容易出错。首先，我已经将我的全球用户名和电子邮件设置为我的公司用户名和电子邮件，因为我的大部分工作都是在那里完成的。第二，这要求我记住每当我克隆一个 repo 来覆盖全局配置时，如果我需要的话。

我最终得到的是一套工具，用于在出现问题时进行特定于上下文的配置和特定于上下文的验证。

# 第一步。)将 GitHub 克隆移动到新文件夹

以前，我将所有回购(包括公司回购和公开回购)放在一个文件夹中:

*   ~/git

现在我需要两个独立的文件夹。一个用于工作，一个用于娱乐。

我决定:

*   ~/git:工作存储库
*   ~/github:公共 github 存储库

创建新文件夹后，我需要将所有公共回购文件移动到新的 github 文件夹中。

我创建了一个简单的 shell 脚本，将所有将远程源设置为 github 的 repos 移动到我的新 github 文件夹中。

```
#!/usr/bin/env bashfor d in */; do
 REMOTE=$(cd $d; git config --get remote.origin.url)
 if [[ $REMOTE == *"github.com"* ]]
 then
   mv $d ~/github/.
 fi
done
```

# 第二步。)设置上下文特定的 Git 配置

从 Git 2.13 开始，全球增加了一个新特性*。gitconfig* 文件语法: [includeIf](https://git-scm.com/docs/git-config)

**includeIf** 允许您进行条件配置逻辑。到目前为止，它只支持文件路径上的条件逻辑，因此需要移动存储库。

我创建了两个 sub git 配置。每个环境一个，我把它放在各自的文件夹里。然后我设置了全局*。gitconfig* 有条件地包含它们:

```
[includeIf "gitdir:~/git/"]
  path = ~/git/.gitconfig[includeIf "gitdir:~/github/"]
  path = ~/github/.gitconfig
```

最后，我为每个环境设置了特定的 git 配置:

## 公司:

```
[user]
 name = userid
 email = corporate@email.com
[core]
 hooksPath = ~/git/.git-hooks
```

## 个人 Github:

```
[user]
 name = My Name
 email = personal@email.com
[core]
 hooksPath = ~/github/.git-hooks
```

您会注意到我在每个页面中设置了用户名和电子邮件。我还为每一个设置了特定的 git-hooks。我们一会儿会讲到这个。

您仍然可以在最初的 *~/中配置许多其他真正的全局配置。gitconfig* 如编辑器、颜色、别名等。

## 延伸阅读:

*   [堆栈溢出发布](https://stackoverflow.com/questions/34597186/use-a-different-user-email-and-user-name-for-git-config-based-upon-remote-clone)
*   [Git 文档](https://git-scm.com/docs/git-config)

# 第三步。)设置特定于环境的 Git 挂钩

配置的好坏取决于对它的验证。因此，我觉得有必要设置一个客户端 git-hook 来检查正在创建的提交是否为环境正确配置。

为此，我在每个环境的 git-hooks 文件夹中创建了一个文件:

## 预提交

```
#!/bin/bash
#
# An example hook script to verify what is about to be committed.
# Called by "git commit" with no arguments.  The hook should
# exit with non-zero status after issuing an appropriate message if
# it wants to stop the commit.
#
# To enable this hook, rename this file to "pre-commit".if git rev-parse --verify HEAD >/dev/null 2>&1
then
 against=HEAD
else
 # Initial commit: diff against an empty tree object
 against=4b825dc642cb6eb9a060e54bf8d69288fbee4904
fi# Redirect output to stderr.
exec 1>&2REMOTE=`git config --get remote.origin.url`
USERNAME=`git config --get user.name`
EMAIL=`git config --get user.email`checkEmailUsername() {
  if [[ "$EMAIL" != "$1" ]]
  then
    echo "Invalid email: $EMAIL"
    echo "git config user.email $1"
    exit 1
  fiif [[ "$USERNAME" != "$2" ]]
  then
    echo "Invalid username: $USERNAME"
    echo "git config user.name \"$2\""
    exit 1
  fi
}if [[ $REMOTE == *"git.corporateRepo.com"* ]]
then
  checkEmailUsername myCorporate@email.com corporateUsername
fiif [[ $REMOTE == *"github.com"* ]]
then
  checkEmailUsername [m](mailto:englundGiant@gmail.com)yPersonal@email.com "Erik Englund"
fi
```

确保文件是可执行的。

如果用户名或电子邮件对于给定的遥控器不正确，这将导致任何提交中止。这避免了以后出现问题时令人讨厌的 rebases。

## 为什么两个 git-hooks 脚本是一样的？

是的，你可以侥幸逃脱。但是对我来说，我的公司已经有了几个客户端 git-hooks 来执行提交消息和不区分大小写的文件名。因此，我需要两个独立的 git-hook 文件夹。然后，我可以在公司存储库上实施公司标准，并在公共存储库中做我想做的任何事情。

## 延伸阅读:

*   [设置验证邮件挂钩](https://www.raphael-brugier.com/blog/git-verify-email-hook/)
*   [饭桶挂钩](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)

# 结论

现在我要记住做的就是在一个地方克隆我的公司库，在另一个地方克隆我的公共 GitHub 库。这不是完全证明，但它是非常接近的。