# Git:更改作者

> 原文：<https://itnext.io/git-change-author-709783e69626?source=collection_archive---------1----------------------->

## 用你的私人邮箱去犯？

我通常致力于我的工作和我的私人 git 项目，我使用两个不同的电子邮件地址。但有时我会把电子邮件弄乱，用我的私人电子邮件地址来完成工作项目，反之亦然。

因此，如果可以更改最后一次提交的作者，这是很方便的，尽管我的用例并不是唯一更改提交作者有用的情况。

![](img/416690b7486255f81fe6c61ce3491cb7.png)

# 更改上次提交的作者

如果您只想更改最后一次提交的作者，那就简单多了。一般来说，如果你想改变最后一次提交的内容，通常会更容易。这可以通过古老的修正案来实现:

```
git commit --amend --author="Mohammad-Ali A'RÂBI <[mohammad-ali@aerabi.com](mailto:mohammad-ali@aerabi.com)>"
```

执行完这个命令后，会打开一个编辑器，这样您也有机会更改提交消息。

注意，在进行修改之前不要有任何更改阶段，否则，这些更改也会添加到提交中。

此外，请注意，现在您必须使用租赁来推动[,因为您已经在历史上做了一些整理工作:](https://medium.com/itnext/git-force-vs-force-with-lease-9d0e753e8c41)

```
git push --force-with-lease
```

# 在旧提交中更改作者

现在，让我们假设我们想要改变提交的作者，这是两次提交之后(因此，第三次提交，从最后一次开始计数)。为了找到我们想要编辑的提交，让我们做一个 oneliner git 日志:

```
git log --oneline
```

输出如下所示:

```
dd16fac (**HEAD -> master**) Add GitHub page 
b8473cf Update README
deba6a0 Add REAME
```

这里，`Add GitHub page`是最后一次提交，我们要编辑`Add README`。为此，我们使用交互式重新设定基准:

```
git rebase -i HEAD~3
```

`HEAD~3`表明我们想回顾过去改变历史到什么程度。在本例中，是 3 次提交。通过运行此命令，将打开一个编辑器，其中包含以下内容:

```
pick deba6a0 Add README 
pick b8473cf Update README
pick dd16fac Add GitHub page
```

请注意，该顺序与 git 日志中的顺序不同。您想要更改的提交现在显示在第一行。要改变它的作者，你需要改变它前面的动词，从`pick`变成`edit`。

```
**edit** deba6a0 Add README 
pick b8473cf Update README
pick dd16fac Add GitHub page
```

然后保存文件并退出。您将返回命令行，并有机会修改您选择的提交:

```
git commit --amend --author="Mohammad-Ali A'RÂBI <[mohammad-ali@aerabi.com](mailto:mohammad-ali@aerabi.com)>"
```

这和以前一样。现在让交互式 rebaser 知道您已经完成了这里的工作，并希望继续:

```
git rebase --continue
```

因为您刚刚更改了作者，所以不会有冲突，您的 rebase 将成功完成。现在，和以前一样，你必须用力推租约。

# 结论

本教程的重点是更改作者的电子邮件地址，但是也可以使用相同的说明来更改作者的姓名或执行任何类型的修改。

今天的主要要点如下:

> 提交前请检查您的电子邮件地址。简单多了。

您还可以添加检查作者姓名和电子邮件地址的 CI 作业:最好在将提交的作者合并到 master 之前修复它们。一般来说，你越早解决问题，就越容易解决。

*   阅读我的[“16 个 Git 技巧和窍门”](https://medium.com/@aerabi/16-git-tips-and-tricks-bf08d0602d3b)
*   [订阅](https://medium.com/subscribe/@aerabi)git 上的每周内容