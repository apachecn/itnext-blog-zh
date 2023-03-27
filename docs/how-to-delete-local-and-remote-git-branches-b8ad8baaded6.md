# 如何删除本地和远程 Git 分支

> 原文：<https://itnext.io/how-to-delete-local-and-remote-git-branches-b8ad8baaded6?source=collection_archive---------7----------------------->

![](img/cd6493ee6e443dca6b8f43d7da2a1e1d.png)

如果您以前曾经使用 Git 对 Angular 代码进行版本控制，那么您很有可能遇到过想要删除一个远程分支或多个分支的情况。这对于开发人员来说是很常见的，尤其是在大型项目中，所以我想写一篇文章来探讨这个主题。

在本文中，我们将了解:

*   如何删除 Git 存储库中的本地分支
*   如何在 Git 中删除远程分支
*   如何删除所有已经合并的 Git 分支
*   如何删除所有本地分支，而不是远程分支
*   如何删除除 master 之外的所有本地 Git 分支

在讨论如何删除远程分支之前，我们首先来看看如何删除本地 Git 存储库中的分支。

> ***注:*** *版本控制系统是现代 web 开发中不可或缺的工具，可以帮助你解决与众多任务相关的诸多问题。Git 是当今最流行的版本控制系统之一。*

在我们继续学习如何[在 Git](https://www.techiediaries.com/delete-local-remote-git-branches/) 中删除本地和远程分支之前，让我们定义一下什么是 Git 分支以及删除分支的副作用。Git 中的分支是指向提交的指针。如果删除一个分支，它会删除指向提交的指针。这意味着，如果您删除了一个尚未合并的分支，并且任何其他分支或标记都无法访问这些提交，Git 垃圾收集将最终删除这些无法访问的提交。

# 删除本地分支

让我们从学习如何删除本地分支开始。

1.  首先，使用`git branch -a`命令显示所有分支(本地和远程)。
2.  接下来，您可以删除本地分支，使用`git branch -d`命令，后跟您想要删除的分支的名称。

```
$ git branch -a # *master # b1 # remote/origin/master # remote/origin/b1 
$ git branch -d b1 # Deleted branch b1.
```

> ***注:*** *你也可以用* `*-D*` *旗来代替* `*--delete --force*` *命令* `*-d*` *。这将使您能够删除本地分支，而不管其合并状态如何。*

# 删除远程分支

与本地分支不同，您不能使用`git branch`命令删除远程分支。但是，您需要使用`git push --delete`命令，后跟您想要删除的分支的名称。您还需要在`git push`后面指定`remote`名称(在本例中为`origin`)。

```
$ git branch -a# *master
# b1
# remote/origin/master
# remote/origin/b1$ git push origin --delete b1
# [...]
# - [deleted] b1
```

# 如何删除所有未合并的 Git 分支？

现在我们已经看到了如何删除 Git 存储库中的本地和远程分支，让我们假设您有多个 Git 分支。如何删除已经合并的分支？理想情况下，您可以一次完成所有这些操作，而不是一个分支一个分支地删除它们。

> ***注意*** *:合并是使用* `*git merge*` *命令执行的，它简单地表示集成来自另一个分支的变更。*

首先，您需要使用以下命令获取远程存储库中合并的所有分支:

```
$ git branch --merged
```

如果您有一个合并的分支，您可以使用以下命令简单地删除合并的本地分支:

```
$ git branch -d branch-name
```

如果您想从远程存储库中删除它，请使用以下命令:

```
$ git push --delete origin branch-name
```

# 删除所有本地分支，而不是远程分支

您可以使用以下 bash 命令删除不在远程存储库上的所有本地分支:

```
$ git branch -r | egrep -v -f /dev/fd/0  <(git branch -vv | grep origin) | xargs git branch -d
```

让我们来分解这个命令:

1.  首先，我们使用`git branch -r`命令获取所有远程分支
2.  接下来，我们使用`egrep -v -f /dev/fd/0 <(git branch -vv | grep origin)`命令获取不在远程的本地分支，
3.  最后，我们使用`xargs git branch -d`命令删除分支。

> [*grep*](https://en.wikipedia.org/wiki/Grep) *是一个命令行实用程序，用于在纯文本数据集中搜索匹配正则表达式的行。它的名字来源于 ed 命令 g/re/p(全局搜索一个正则表达式并打印匹配行)，效果相同。grep 最初是为 Unix 操作系统开发的，但后来可用于所有类似 Unix 的系统和其他一些系统，如 OS-9。*
> 
> [*xargs*](https://en.wikipedia.org/wiki/Xargs) *(“扩展参数”的缩写)是 Unix 和大多数类 Unix 操作系统上的一个命令，用于从标准输入构建和执行命令。它将标准输入转换成命令的参数。*

# 删除除 Master 之外的所有本地 Git 分支

如果您已经完成了本地 Git 分支，那么移除它们以释放空间通常是一个好的做法。您可以简单地运行以下命令:

```
$ git branch | grep -v "master" | xargs git branch -D
```

我们使用`grep -v "master"`命令搜索分支(不包括主分支)，然后使用`git branch -D`命令删除它们。

# 结论

在本文中，我们看到了如何从 Git 存储库中删除远程和本地分支。我们了解到:

*   如何删除 Git 存储库中的本地分支
*   如何在 Git 中删除远程分支
*   如何删除所有已经合并的 Git 分支
*   如何删除所有本地分支，而不是远程分支
*   如何删除除 master 之外的所有本地 Git 分支

感谢您阅读我的文章，希望对您有所帮助。这篇文章最初发表在[技术刊物](https://www.techiediaries.com)上。