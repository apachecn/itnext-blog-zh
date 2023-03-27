# 16 个 Git 提示和技巧

> 原文：<https://itnext.io/16-git-tips-and-tricks-bf08d0602d3b?source=collection_archive---------1----------------------->

## Git 16 岁生日的时候

![](img/6bc7a2f17d751809bdb717e00ca185aa.png)

在过去的 6 年里，我一直在使用 git，根据我的经验，这里有一些技巧，可以让你成为一个更有效的 git 用户:

# 0x00。不要强行推动

你可能经历了一次不成功的经历。例如，当您改变提交的基数或挤压提交时，可能会发生这种情况。在这种情况下，可以强行推动:

```
git push --force
```

ide 有一个“强制按下”按钮，有时甚至会鼓励这样做。但是，我会说:

> 不要强行推动，除非你想打破什么东西。

危险性较小的替代方案被称为“租借强制”。如果您进行了重新调整或修改，请使用以下替代方法:

```
git push --force-with-lease
```

了解更多:[Git Force vs Force with Lease](/git-force-vs-force-with-lease-9d0e753e8c41?source=user_profile---------0----------------------------)

# 0x01。使用别名

人们使用`--force`而不是`--force-with-lease`的原因之一是他们必须键入的字符数量。这就是别名发挥作用的地方:

```
git config --global alias.enforce "push --force-with-lease"
```

在这里，我创建了一个新的别名`enforce`，它的行为类似于`push --force-with-lease`:

```
git enforce
```

现在`git push --force`比`git enforce`要输入更多的字符。

> 愿力量与租约同在。

# 0x02。设置上游

推送时，您可以指定要推送至哪个远程分支:

```
git push origin master
```

> 奖励点:您也可以使用 enforce 指定分支。

```
git enforce origin the-feature-branch
```

但是，提到远程分支机构的名称几次后会变得很烦人。上游是一个默认的远程分支，当您没有指定它时，您希望将它推入。设置它就像在 push 命令中写入`-u`一样简单:

```
git push -u origin the-feature-branch
```

从下一次开始，您可以省略远程分支。

> 如果您想将多个数据推送到一个远程分支，那么设置上游更经济。

# 0x03。正确命名分支

处理许多不同的特性和错误，很快就会弄不清哪个分支是哪个分支。特别是，如果你命名一个分支`the-new-feature`或者`bug-fix`。

> 为您的分支使用一个描述性的名称。

更糟糕的是，另一个开发人员可能也在修复一个 bug，或者正在开发与您类似的功能。所以，你们可能有相同的分支名称，并且在不知道的情况下破坏了彼此的工作。

> 在你的分行名称前加上你的首字母。

我见过的被广泛使用的标准如下:

```
git checkout -b jto/add-noob-saibot
```

首字母是:名的 1 个字母，姓的 2 个字母。在这种情况下，约翰·托拜厄斯变成了`jto`。然后是斜线，然后是特征描述。更好的是，将问题/票据/任务 ID 添加到组合中:

```
git checkout -b jto/666-add-noob-saibot
```

# 0x04。清理本地分支机构

另一件让你很难在本地分支之间进行搜索的事情是有太多无用的分支。

> 合并后删除本地分支。

我太懒或太健忘，不会做那件事。因此，我偶尔运行这个神奇的命令来删除每个非主分支:

```
git branch | grep -v master | xargs git branch -D
```

但是还有另一个神奇的设置，如果从远程存储库中删除了本地分支，那么在获取时也会删除它们:

```
git config fetch.prune true
```

所以这取决于你。你想要控制还是自动化？

# 0x05。清理本地存储的远程分支

人们可能不知道这一点，但是每次您提取或获取时，所有远程分支的副本都会被下载并存储在本地。当远程分支被删除时，这些分支不会被自动删除。

运行以下命令，列出本地 git repo 上存储的所有远程分支:

```
git branch -r
```

其中一些可能在远程回购中不存在。要找出是哪些，请运行以下命令:

```
git remote prune origin --dry-run
```

要从本地删除这些分支，请运行以下命令:

```
git remote prune origin
```

# 0x06。隐藏消息

一个普通的 git 用户知道`git stash`是做什么的:它将您未提交的更改存储在一个堆栈中。当您想要提取或者切换带有未提交变更的分支时，这是特别需要的。通常情况下，git 会指示你在拉取或结帐之前先把东西藏起来。

但这并不是存储的唯一用例。在做某件事的过程中，你可能会说，“哦，我需要稍后再完成这件事；我们就暂时把它藏在这里吧”。然后你把零钱藏起来，永远忘记它。

通过执行`git stash`，它使用最后一次提交的消息来保存您的更改。但这通常是不相关的。要用合适的服装信息隐藏更改，请执行以下操作:

```
git stash push -m "Change the ID into a string"
```

# 0x07。重定基础而不是合并

维护 git 分支通常有两种策略:rebase vs merge。当合并把你的 git 历史变成一个讨厌的蜘蛛网时，重定基础保持主分支的干净和线性。但是，出于任何原因，这是 git 的默认行为。

我建议你研究一下重定基础和合并，如果你还没有做的话，然后选择重定基础而不是合并。

要在拉动时重设基础，请执行以下操作:

```
git pull --rebase
```

例如，要用主文件的最新更改更新您的特征分支，请执行以下操作:

```
git pull --rebase origin master
```

在 Github 的 pull 请求中，merge 按钮可以变成 rebase 按钮。在 GitLab 上，当将一个功能分支合并到主功能时，您可以更改项目的设置来进行“快进”(这意味着它将重新建立基础)。

> 你不应该合并。

*注意。*重定一个本地分支后，你要`enforce`它(参考 0x01)。

# 0x08。将重设基础设为默认值

当您执行`git pull`时，默认行为是获取变更，然后将它们合并到您当前的分支中。要更改默认拉取行为，请执行以下操作:

```
git config pull.rebase true
```

这会让每一个`git pull`都变成一个`git pull --rebase`。

# 0x09。拉取时自动存储

现在使用`git pull`rebase 代替合并，您可以享受其他优势，如自动存储:

```
git config rebase.autoStash true
```

这意味着如果你正在做某件事，你不需要在拉之前把它们藏起来；git 会帮你做的。它将隐藏更改，进行拉取(rebase)，然后将隐藏的更改应用到结果中。

# 0x0A。壁球

挤压基本上意味着将几个提交组合在一起。在将 pull/merge 请求合并回 master 之前，压缩其中的所有提交是一个很好的做法。然后，您有机会编写一个很好的提交消息，将所有事情都考虑进去，并创建一个更好的主分支，每个拉/合并请求或问题提交一次。

但是，挤压有点棘手。首先，您需要选择想要挤压的提交数量。假设是 5。然后执行以下操作:

```
git rebase -i HEAD~5
```

这个命令在最后 5 次提交时启动了一个叫做交互式 rebase 的东西。将启动一个编辑器(如`nano`)，其内容如下:

```
pick f72e7cb Feat: Resurrect flatMap
pick 1a18b1c Feat: Add list transformations
pick 3420af4 3.0.0
pick f55a9ee Docs: Add marble diagrams to README
pick 0433f9d Update README.md
```

保持第一行不变，用`s`(用于挤压)替换后面各行中的每个`pick`:

```
pick f72e7cb Feat: Resurrect flatMap
**s** 1a18b1c Feat: Add list transformations
**s** 3420af4 3.0.0
**s** f55a9ee Docs: Add marble diagrams to README
**s** 0433f9d Update README.md
```

保存并退出。然后，编辑器再次启动，显示所有 5 次提交的提交消息。随意删除全部，重写整个消息。保存。退出。还有 tada！就这样。

> 壁球。

*注意。*压扁后需要`enforce`而不是`push`。

# 0x0B。重置基础前挤压

假设您有一个比主分支提前 13 次提交、落后 3 次提交的功能分支。这就是 rebasing 所做的:它删除您的 13 个提交，将来自 master 的 3 个提交添加到您的分支，然后开始一个接一个地应用所有的 13 个提交。所以，如果有冲突，很有可能你需要修复 13 次。我是凭经验说话的。

将你的 13 次提交压缩为 1 次，使得重置基础更加容易。特别是，如果您后来的提交恢复了先前的提交或者在某种意义上取消了它们。

# 0x0C。改进

你把所有的提交都压扁，推到远程 repo，然后“妈的，少了个分号”。在这种情况下，请修改。

```
git commit --amend
```

你压扁了，准备推，但是“妈的，我忘了在提交消息里提问题 ID”:修改。

你不喜欢挤压，但仍然想保持你所有的贡献在一个承诺:修改。

你写了宪法法律，忘了在里面尊重人的尊严:修改。

Amend 会将更改添加到最后一次提交中，并允许您更改提交消息。

# 0x0D。重写历史

交互式重置基础不仅用于挤压。你可以用它改写(即改变信息)一个旧的提交，你可以改变提交顺序或混合；例如，挤压最近的提交，在它之前有 2 个提交，保持中间的提交不变。

换句话说，只需找到提交并将`pick`更改为`reword`:

```
pick f72e7cb Feat: Resurrect flatMap
pick 1a18b1c Feat: Add list transformations
**reword** 3420af4 3.0.0
pick f55a9ee Docs: Add marble diagrams to README
pick 0433f9d Update README.md
```

要更改顺序，只需更改选择中的顺序:

```
pick f72e7cb Feat: Resurrect flatMap
**pick 3420af4 3.0.0
pick 1a18b1c Feat: Add list transformations**
pick f55a9ee Docs: Add marble diagrams to README
pick 0433f9d Update README.md
```

诸如此类。

> 历史是由最后一个开发者书写的。

# 0x0E。自动挤压

挤压，并使用交互式 rebase 工具作为一个整体，是一个非常乏味的工作，特别是如果你想改变顺序和挤压一次。这就是自动挤压介入的地方。

假设您在一个特性分支中处理了 3 个不同的特性(顺便说一下，这是一个糟糕的实践)，并且想要将它们作为 3 个不同的提交合并到主分支中。但是后来发现第一次提交有问题。在这种情况下，这是最后一次提交，一旦可以修改，但它有点棘手的提交之前。

假设我们的提交如下:

*   `Feat: Add feature 1`
*   `Feat: Add feature 2`
*   `Feat: Add feature 3`

为功能 1 制作补丁，并使用以下消息提交它:

```
git commit -m "squash! Feat: Add feature 1"
```

然后，使用自动挤压标志进行交互式重设基础:

```
git rebase -i HEAD~4 --autosquash
```

编辑器将自动显示正确的作业:

```
pick f72e7cb Feat: Add feature 1
squash 1a18b1c squash! Feat: Add feature 1
pick 3420af4 Feat: Add feature 2
pick f55a9ee Feat: Add feature 3
```

# 0x0F。自动自动挤压

通过设置以下配置，使自动挤压成为交互式重定基础的默认行为:

```
git config rebase.autosquash true
```

# 进一步阅读

*   [在 Twitter 上关注作者](https://twitter.com/MohammadAliEN)以获得未来工作的通知。
*   [Git 提交消息的十诫](/ten-commandments-of-git-commit-messages-94bd6dcf6e0e)
*   [13 个处理数组和元组的便捷 RxJS 操作符](/13-handy-rxjs-operators-ab5a9a1db60)