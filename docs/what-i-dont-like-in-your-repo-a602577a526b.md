# 我不喜欢你的回购协议

> 原文：<https://itnext.io/what-i-dont-like-in-your-repo-a602577a526b?source=collection_archive---------1----------------------->

![](img/2776a11c74ce85158b424416a462224d.png)

大家好，我是奥列格，我对(可能是你的)回购大吼大叫。

这是我和一个朋友关于如何为任何规模和任何编程语言的任何社区做一个好的和有帮助的回购的对话的副本。

我们开始吧。

# README 什么也没说

但这是任何回购的关键部分！

这是你与潜在用户的第一次互动，也是你可能带给用户的第一印象。

在名称(也许还有一个标志)之后，是放置一些徽章的好地方，例如:

*   最新版本
*   CI 状态
*   链接到文档
*   代码质量
*   代码覆盖率
*   甚至聊天中的用户数量
*   或者干脆把它们都滚动到[https://shields.io/](https://shields.io/)

个人失败。不久前，我在围棋中做了一个简单、古怪且有点滑稽的项目，叫做[破坏](http://github.com/cristaloleg/sabotage)。我引用了一首歌，添加了一张图片，但是……没有提供任何关于它的信息。

这需要 10 分钟来做一个简单的介绍，并解释我分享的内容和它能做什么。

没有理由你或我应该跳过它。

# 自定义许可或根本没有许可

首先也是最重要的:做。不是。(重新)发明。执照。求你了。

当你打算创建一个新的闪亮的新许可证或使任何现有的更好，请问自己 17 次:这样做的意义是什么？

任何规模的公司在许可证方面都非常保守，因为这可能会毁掉他们的业务。所以，如果你的目标是大量的观众，这是一个愚蠢的方法。

有很多关于如何选择许可的指南，没有许可的使用它或者使用一个不受欢迎的或者有趣的(像 WTFPL)对用户来说是一个不好的信号。

请随意选择一个最受欢迎的:

*   麻省理工学院——当你想免费提供时
*   BSD 3——当你想要更多的权利时
*   Apache 2.0 —当它是一个商业产品时
*   GPLv3——这也是一个不错的选择

(这可能是一个固执己见的列表，但无论如何)

# 没有 Dockerfile 文件

现在已经是 2019 年了，集装箱已经赢得了这个世界。

对于任何人来说，发出一个`docker pull foo/bar`命令要比下载所有依赖项、配置路径、意识到某些东西可能不兼容甚至害怕完全破坏他们的系统简单得多。

未验证的项目有没有保证没有`rm -rf`？😈

添加一个简单的`Dockerfile`和所有需要的东西只需要 30 分钟。但是这将给你的用户一个安全快捷的方法来开始使用、验证或帮助改进你的工作。双赢的局面。

# 没有拉取请求的更改

这可能看起来很奇怪，但给我一秒钟。

当一个项目很小，只有 0 个或很少的用户时——这可能没问题。很容易跟踪最近几天发生的事情:修复、新特性等。但是规模变大了，哦…就变成噩梦了。

你已经将几个提交推送到 master 中，所以很可能你是在你的计算机上做的，没有人看到发生了什么，也没有任何反馈。您可能会破坏 API 向后兼容性，忘记添加或删除某些东西，甚至做无用的工作(哦，讨厌的一个)。

当你做一个拉请求时，一些随机的大师级的高级架构师可能会偶尔检查你的代码，并建议一些修改。听起来不太可能，但任何额外的眼睛可能会发现错误或架构错误。

不要隐藏你的工作，这难道不是开源的理由吗？

# 膨胀的依赖性

也许这只是我的看法，但我对依赖非常保守。

当我在锁文件中看到几十个 dep 时，我想到的第一个问题是:那么，我准备好修复它们中任何一个的任何故障了吗？

是的，今天有效，也许 1 周/月/年前有效，但是你能保证明天会发生什么吗？我不能。

# 没有样式或格式

不同的文件(有时甚至是函数)以不同的风格编写。

这给贡献者带来了麻烦，因为一个人喜欢空格，另一个人喜欢制表符。这只是一个最简单的例子。

那么结果会是什么呢:

*   一种风格的文件和另一种风格的完全不同
*   1，其中`{`位于一行的末尾，另一个`{`位于新行
*   1 函数在函数式中，右下方是纯过程式

他们谁是对的？——我不知道，但如果有效的话，这是可以接受的，但这也毫无理由地严重分散了读者的注意力。

简单的规则是:使用格式化程序和 linters: eslint，gofmt，rust fmt…哦，太多了！您可以随意配置它，但请记住，最受欢迎的往往是最自然的。

# 没有自动构建

如何验证用户可以构建您的代码？

答案很简单——构建系统。TravisCI，GitlabCI，CircleCI，这只是其中的几个。

将构建系统视为一个无声的伙伴，它将检查您的每一次提交，并将自动运行格式化程序/链接程序，以确保新代码具有良好的质量。听起来很神奇，不是吗？

和往常一样，添加一个简单的 yaml 文件，描述如何在几分钟内完成构建。

# 没有发布或 Git 标签

主树枝可能断了。

发生这种事。这是令人不快的事情，但它确实发生了。

一些最近的变化可能会被合并，不知何故，这导致了主机上的问题。需要多长时间给你修好？几分钟？一个小时？一天？直到你度假回来？谁知道‾\_(ツ)_/‾

但是当有一个 Git 标签指出一个项目是正确的并且能够被构建的时候，哦，这是一件好事，可以让你的用户的生活变得更好。

在 Github(类似于 Gitlab 或任何其他地方)上添加发布只需几秒钟，没有理由省略这一步。

# 没有测试

嗯，可能没问题。

当然，拥有正确的测试是一件好事，但可能你是在下班后的空闲时间或周末做这个项目(我很抱歉，我经常这样做，而不是为了休息)。

所以不要对自己太严格，可以随意分享自己的工作，分享知识。测试可以在以后增加，和家人朋友在一起的时间更重要，和精神或身体健康一样。

# 结论

还有很多其他的东西会让你的回购变得更好，但也许你会在评论中亲自提到它们？

推特:[https://twitter.com/oleg_kovalov/status/1105719270116388864](https://twitter.com/oleg_kovalov/status/1105719270116388864)

龙虾:【https://lobste.rs/s/6gixqw/what_i_don_t_like_your_repo 

https://news.ycombinator.com/item?id=19376264[HN](https://news.ycombinator.com/item?id=19376264)

Reddit:[https://www . Reddit . com/r/programming/comments/B0 isug/what _ I _ dont _ like _ in _ your _ repo/](https://www.reddit.com/r/programming/comments/b0isug/what_i_dont_like_in_your_repo/)

谢了。