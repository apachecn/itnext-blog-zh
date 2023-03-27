# Git 重写提交

> 原文：<https://itnext.io/git-reword-commit-82f0773fb22b?source=collection_archive---------3----------------------->

## 如何更改提交消息？

有时您想要更改提交的消息，例如，更新信息、修复打字错误或修复引用。如果是这样的话，这一块是给你的。

![](img/467b39a44d3bd2b57107c0eb9bd42649.png)

通常有两种方法可以改变提交的消息，一种更简单，只适用于最后一次提交，另一种适用于任何提交，但更复杂。

> 就像这个行业的其他事情一样，你越早解决它就越容易。

# 更改上次提交的消息

更改最后一次提交的消息相当容易:

```
git commit --amend
```

通过运行该命令，将打开一个文本编辑器(例如 VIM 或 Nano)，您可以更改提交消息，保存并退出。就是这样。验证提交消息是否已更改:

```
git log
```

## 注意:确保没有上演任何东西

在运行 commit 命令之前，请确保没有为提交准备任何内容，否则，其他更改将添加到最后一次提交中。要检查您是否有一些分阶段的更改，请运行:

```
git status
```

如果你真的有一些阶段性的改变，先取消它们:

```
git restore --staged <file>
```

## 注意:用力推配合租赁

如果您更改消息的提交已经被推送，您现在必须使用 [force 和 lease](https://medium.com/itnext/git-force-vs-force-with-lease-9d0e753e8c41) 再次推送，就像您在历史中所做的一些内务处理一样:

```
git push --force-with-lease
```

# 更改任何提交的消息

这个方法适用于任何提交，无论是最后一次提交还是更早的提交。让我们假设您想要更改第三个旧提交的提交消息。做一个日志，以确保它在那里:

```
git log --oneline -3
```

在这个日志命令中:

*   `--online`每次提交时使输出联机
*   `-3`使其仅显示最近 3 次提交

输出将类似于以下内容:

```
dd16fac (**HEAD -> master**) Add GitHub page 
b8473cf Update README
deba6a0 Add REAME
```

这里，第一个列表是最后一个提交。第三行显示了第三次提交，我们希望更改它的消息。为此，对最后 3 次提交进行交互式重新排序:

```
git rebase -i HEAD~3
```

`HEAD~3`表明我们想回顾过去改变历史到什么程度。在本例中，是 3 次提交。通过运行此命令，将打开一个编辑器，其中包含以下内容:

```
pick deba6a0 Add README 
pick b8473cf Update README
pick dd16fac Add GitHub page
```

请注意，该顺序与 git 日志中的顺序不同。您想要更改的提交现在显示在第一行。要改变它的信息，你需要改变它前面的动词，从`pick`变成`reword`。

```
**reword** deba6a0 Add README 
pick b8473cf Update README
pick dd16fac Add GitHub page
```

然后保存文件并退出。然后，将打开另一个编辑器，显示提交消息。对提交消息进行更改，保存并退出。答对了。

要确保提交消息被更改，请执行 git 日志:

```
git log -3
```

# 遗言

我每周在 git、GitHub 和 GitLab 上写文章。

*   [订阅](https://medium.com/subscribe/@aerabi)my Medium publishes，以便在新的 Git 周刊发布时获得通知。
*   关注 Twitter 上的[我，获取 git 上的每周文章和每日推文。](https://twitter.com/MohammadAliEN)