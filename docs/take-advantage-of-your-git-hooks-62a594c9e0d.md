# 利用你的 git 钩子

> 原文：<https://itnext.io/take-advantage-of-your-git-hooks-62a594c9e0d?source=collection_archive---------3----------------------->

## 如果您是 javascript 开发人员，您可以在 git 存储库的 hooks 文件夹中放些什么呢

![](img/ef587df239fe94c3aa163259b6516437.png)

弗朗切斯科·温加罗在 Unsplash[拍摄的照片](https://unsplash.com?utm_source=medium&utm_medium=referral)

什么是 git 挂钩？如果你已经开发了很多年，却没有听说过 git 挂钩，也没有真正使用过它们，我不会感到惊讶。

Git 挂钩基本上是 bash 文件，可以在 git 命令前/后运行。

为什么这很有帮助？老实说，有很多不同的钩子你可能永远不会使用。但是我发现了一些有用的方法。尤其是当我在一个大型开发团队中工作时。

下面是您可以使用的所有 git 挂钩的列表

```
applypatch-msg
post-update
pre-push.sample
prepare-commit-msg
commit-msg
pre-applypatch
pre-rebase
update
fsmonitor-watchman
pre-commit
pre-receive
```

你可以在你隐藏的。本地存储库中的 git 文件夹。

路径:

```
.git/hooks/
```

如果你现在看那里，你应该会看到一堆样本文件，你可以用它们来开始编写你自己的钩子。

我想把重点放在我发现非常有用的一个钩子上。这个挂钩是提交前挂钩。我将向你们展示我在预提交钩子里放了什么，希望最后你们也会想做同样的事情。

# 较美丽

![](img/dd99d8755669ef11f6df1103262c4181.png)

[https://prettier.io/](https://prettier.io/)

什么更漂亮？([摘自他们的网站](https://prettier.io/))

> 固执己见的代码格式化程序
> 
> 支持多种语言
> 
> 与大多数编辑器集成
> 
> 选择不多

那么我为什么要用它呢？我了解到每个人对代码应该如何格式化都有自己的看法。如果有一件事是我讨厌的，那就是浪费我的时间和我的同事争论格式问题。老实说，有些事情你一开始不会喜欢。但是如果你有机会的话，你会惊讶地发现这种格式对你来说是多么的有用。

为什么使用 git 仓库这么好？你发现自己花在拉取请求和评论无关紧要的事情上的时间减少了。它也使得拉取请求看起来更加清晰。

因此，下面是在您的系统上安装 pretty 需要做的事情(pretty 支持多种语言，但它的主要语言是 javascript，所以我将向您展示如何使用 javascript)。

```
yarn add prettier --dev --exactornpm install --save-dev --save-exact prettier
```

我也更喜欢在我的 git 钩子上使用 quit-quick，因为它更快。下面是如何安装它。

```
yarn add pretty-quick --devornpm install --save-dev pretty-quick
```

现在导航到您的。git/hooks 文件

```
cd {repo-directory}.git/hooks
```

在下面的命令中创建一个名为 pre-commit 和 paste 的文件。

在`pretty-quick`之前的一行只是允许输出被打印到控制台，这样你就可以看到发生了什么。

现在，如果你在你的仓库中，你应该看到每次提交时运行得更漂亮。

请注意，如果你在一个大的存储库中，并且你在整体上运行得更好，你将有很多 git 的变化。所以如果我是你，我不会在整个代码库上运行它，除非你的整个团队都同意。

# 林挺

好吧，所以如果有什么东西与漂亮相伴，那就是林挺。我甚至无法解释为什么林挺是编写 javascript 代码的开发团队中的一员。在林挺发现了如此多的虫子。现在，重要的是你要找到一套适合你的团队的规则。

我的建议是遵循 airbnb 的风格指南。他们有一个很棒的规则集，并且很好地解释了为什么他们选择了不同的规则集。

这里有一个林挺规则的链接。

要将它添加到您的 git 挂钩中，您需要首先将它安装到您的项目中。

```
npm install eslint --save-dev
```

如果你在全局范围内安装它，你就不需要在钩子中添加对 node_modules 文件夹的引用。

我喜欢在钩子文件中运行的另一个工具是`eslint-friendly-formatter`

```
npm install eslint-friendly-formatter --save-dev
```

如果我是你，我也会全局安装它，这样当你把它放在你的 git 钩子中时，你就不必引用节点模块文件夹了。

现在，在预提交文件中，确保将这一行添加到末尾。

# 结论

您的整个文件现在应该如下所示:

现在，如果您签入代码，您应该会看到预提交钩子正在运行。

让我知道它是如何为你工作的！也让我知道你使用的其他提交钩子以及它们是如何帮助你的！

编码快乐！

***Avery Duffin*** *是* [*【开发者午餐】*](https://www.developerswholunch.com) *的所有者和创始人，这是一个播客和博客，旨在帮助那些学习编程并寻求在软件开发领域建立职业生涯的人。他目前在*[*rai focus*](https://www.rainfocus.com/)*担任软件工程师，居住在美国犹他州。你可以在*[*Twitter*](https://twitter.com/DuffinAvery)*和*[*linkedIn*](https://www.linkedin.com/in/avery-duffin-69317228/)*上和他联系。*