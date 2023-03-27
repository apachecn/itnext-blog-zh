# Git 二等分——如何跟踪一个中断的提交

> 原文：<https://itnext.io/git-bisect-how-to-track-down-a-broken-commit-4fc82ae98ed6?source=collection_archive---------4----------------------->

![](img/866aff31ba903788f2f8ab9dfc58915e.png)

照片由[尼克·费因斯](https://unsplash.com/@jannerboy62?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/roads?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

时不时——时有发生:*有人*折断了主枝。根据项目规模的不同，回归测试可能会非常困难和令人疲惫。在几个合并的提交之后，人们注意到最后 50 个提交中的一个确实引入了回归，并且上游主分支被破坏。

如果你手动地*发现这个引入回归的提交，例如一个接一个地测试提交，这是困难和令人疲惫的。如果你在一个已知的工作配置和不工作的配置之间有五次提交，这可能是你可以做的。您只需一个接一个地检查提交，检查错误是否已经存在。但是如果有，你能做什么呢..在工作和非工作提交之间是 50、100 还是 500 次提交？*

> git 平分救援！

**Git**

我想现在每个开发人员都应该知道 git 了。如果没有:我写了两篇关于 git 的文章，您可能想先看看:

[](/become-a-git-terminal-pro-ab6d1955606f) [## 成为 Git 终端 Pro

### 一些有用的提示，可以提高您在命令行中使用 git 的效率。

itnext.io](/become-a-git-terminal-pro-ab6d1955606f) 

和

[](/how-to-undo-a-commit-in-git-2c7d49deabe0) [## 如何在 git 中撤销提交

### 许多开发人员在 git 中努力撤销提交。这有助于您撤销提交！

itnext.io](/how-to-undo-a-commit-in-git-2c7d49deabe0) 

简而言之，引用我自己的话(是的——我知道)

> Git 是一个版本控制系统，可以让你在开发软件的时候跟踪代码的变化。当你在团队中工作时，它是有用的，但当你独自工作时，它也是有用的——git 是必须的。

# **Git 平分**

为了跟踪一个*失败的*提交，git 给了我们一个非常方便的工具叫做`git bisect`。Git 二等分执行一次二分搜索法来查找中断的提交。好吧，那么什么是二分搜索法呢？

# **二分搜索法**

你可能还记得很久很久以前的研究..二分搜索法是一种算法，它接受一个列表，反复地把它分成两半，直到你缩小问题的范围。

> 二分搜索法是一种有效的算法，用于从排序的项目列表中查找项目。它的工作原理是重复地将列表中可能包含该项目的部分分成两半，直到您将可能的位置缩小到只有一个。

一个简单的例子是猜谜游戏。让我们假设你想到一个 0 到 100 之间的数字。所以我问你:数字是否高于 50？没有吗？好的:现在我把可能的范围从 0 到 100 分成两半。从 0 到 50 因为我知道它不在 50 到 100 的范围内，因为你说的数字不高于 50。好的:这个数字是否高于 25？什么事？好的:现在我再把 0 到 50 的范围分成两半。一个范围从 0 到 25，另一半从 25 到 50——我知道它一定在第二个范围内。我继续这样做，直到找到正确的数字。那是二分搜索法。如果你想了解更多:[这篇](https://www.khanacademy.org/computing/computer-science/algorithms/binary-search/a/binary-search)是一篇相当有帮助的文章。

# **Git 平分**

好吧——回到`git bisect`。我们如何在 git 中做同样的事情？基本原理是这样的:首先你要告诉 git 你想用`git bisect start`做一个二等分。然后你挑一个知道*不好的*跟`git bisect bad <commit>`犯。如果您将`<commit>`留空，git 会将您当前的`HEAD`视为错误提交。如果我们将它与上面提到的猜谜游戏进行比较，这将是一个更高的结果:100。

我们不需要选择一个零，或者我们范围的起点。这是**最后一次已知工作的**提交。所以找到提交散列并键入类似于`git bisect good eb02f47e20`的内容。一旦您指定了一个好的和一个坏的提交，git 二等分应该打印如下内容:

```
Bisecting: 493 revisions left to test after this (roughly 9 steps)
[c0b028f205a78ecc748e69d62450b4a804784da9] mb/google/cyan: Adjust ACPI interrupt triggering for audio codecs
```

好的，我们现在做的是:我们将一个提交(这是在我们的已知错误的提交之后的 493 个提交)作为一个好的提交。Git 二分现在开始二进制搜索，以找到破坏树的提交。

现在您可以再次构建您的项目—测试它，看看它是否工作。如果是:键入`git bisect good`如果不是:`git bisect bad`。通过这种方式，您可以告诉 git 二等分要么继续在一个提交中搜索，要么在提交的另一半中搜索。

在这个过程中，git 二分总是会告诉您还有多少可能的提交，以及(大致)还需要采取多少步骤。

```
Bisecting: 7 revisions left to test after this (roughly 3 steps)
[a38fee31b59acd8e3f07ec89d4328e98b6979611] nb/intel/sandybridge: Rename raminit_ivy.c
```

# **为什么？**

所以问题是——为什么使用 git 二等分而不是一个接一个地检查您的提交？你可能已经注意到:Git 平分*总是*大约需要 9 个步骤。因此，不要一个人完成 200 次提交，你可以完成 9 次提交，测试一下，然后就有答案了！

黑客快乐！