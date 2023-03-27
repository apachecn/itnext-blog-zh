# 如何更快地查看拉式请求:实用技巧

> 原文：<https://itnext.io/how-to-review-pull-requests-more-quickly-practical-tips-2305b7bc70c1?source=collection_archive---------5----------------------->

![](img/2d54af65519bd0db51ba0aa8d264523e.png)

# 介绍

我一直在研究加快公关审查的方法，因为我作为一名技术主管审查了很多公关。这篇文章讨论了我发现的帮助我做到这一点的东西。

要明确的是，我并不是暗示你应该加快你的公关审查。作为开发人员，这是你最重要的职责之一。当您审查对您的代码库有重大影响的 pull 请求时，您应该花时间来确保代码写得很好，并且涵盖了所有的边缘情况。这篇文章简单地讨论了如何聪明地加速这个过程。

我们开始吧！

# 马上开始复习

尽快开始你的公关评论。起初，这可能看起来违反直觉，但尽快开始你的公关审查有很多好处，包括你的公关审查最终会花更少的时间。

## 最小化上下文切换

开发人员通常会花 5 到 10 分钟的时间自我审查他们提交的 PR，他们的注意力仍然集中在它上面。当你开始回顾、评论并与开发人员接触时，他们会很快做出反应并更好地回答你的问题，因为问题仍然在他们的脑海中记忆犹新。

相反，如果你在审查 PR 之前等待几个小时，开发人员很可能会开始处理不同的任务。切换回公关环境需要时间，所以他们需要更长的时间来理解和回应你的评论。

当您在收到 PRs 后很快定期审查它们时，这也会鼓励您团队中的其他开发人员编写更小的 PRs。看到更小的 PRs 不需要那么长的时间来审查，将开始更小的 PRs 和更快的发货到生产的连续循环。

## 一开始可能会让你慢下来

最初，快速回顾 PRs 可能会让你慢下来一点，但这没关系，因为你提高了整个团队的生产力。如果 PRs 开始占用你大量的时间，考虑在你站立的时候提及你已经花了时间来回顾它。

综上所述，如果你正在一个活跃的编码会话中，并且在“区域”中，那么立即推迟审查 PRs 是好的。当你频繁地切换上下文时，你会损害你和你的团队的生产力。所以等到你在自己的工作中达到一个很好的停止点也没关系。

# 提问

我过去常常以不需要任何帮助就能解决问题为荣。然而，如果你想加快你的复习过程，多问一些澄清性的问题。理解别人的代码需要付出很多努力，要求澄清并不可耻。

其他开发人员也会将提问视为团队工作的一个正常部分，这可以改善团队沟通和协作。

# 从大部件开始

首先，尝试找出对应用程序设计有重大影响的代码。看看重要的数据模型、类和高级抽象的变化。这将有助于您理解较小的变化，并通常加快您的代码审查。

如果你发现一些大的错误，立即发送评论，不要等着看整件事。你甚至可以暂停任何进一步的审查，要求先解决这些问题。

我发现的另一个有用的技巧是先阅读验收测试。它帮助您理解代码应该做什么，并看到边缘情况。

# 尽量不要太死板

同样，这并不意味着你应该牺牲代码的质量。这只是意味着在小细节上花费时间可能不是最有效的利用你的时间。牢记[收益递减](https://www.britannica.com/topic/diminishing-returns)定律。

## 现在批准，以后修复

有时批准一份 PR 是没问题的，即使你还留下了一些未解决的意见:

*   您确信开发人员会处理您的其余意见。
*   剩下的变化很小，如果没有解决，PR 仍然是可以接受的。

如果您的团队分布在几个时区，这种做法尤其方便。这将帮助你们所有人节省来回交谈的时间。

## 您并不总是需要手动测试

如果您对开发人员的能力有信心，您可能不需要手动测试他们的代码。例如，你可以问他们是否处理过一个特殊的边缘案例，或者让他们为你录制一个截屏。我喜欢在提交 PRs 时使用[织机](https://www.loom.com/)快速录制演示，如果这样更容易解释的话。

一般来说:如果一个拉请求有助于你的应用程序的整体质量，并提供新的功能，你应该倾向于批准它。

# 使用更好的工具

在从事任何需要大量脑力劳动的任务之前，我喜欢先花些时间确保我使用了最适合这项工作的工具。

在我的研究中，我发现一些工具可以让我以更快的速度和更好的理解来审查拉式请求。

## 代码流

[Codestream](https://marketplace.visualstudio.com/items?itemName=CodeStream.codestream) 是一个 VSCode 扩展，允许你从你的编辑器中查看 PRs。与只显示行差异的 web Git 界面不同，Codestream 允许您在编辑器中查看整个文件，因此您可以更好地了解 PR 变化如何影响您的应用程序。

Codestream 还最大限度地减少了上下文切换，因为您不必在编辑器和查看 PRs 的 web 界面之间切换。

## 更好的拉取请求

[Better pull request](https://chrome.google.com/webstore/detail/better-pull-request-for-g/nfhdjopbhlggibjlimhdbogflgmbiahc?hl=en) 是一个小型的 Chrome 扩展，为 Github PR reviews 的 web 界面增加了文件树功能。用文件树来审查拉请求减少了浏览变更所需的精神负担。

我相信还有其他很棒的工具，所以自己做研究，发挥创造力吧！

# 结论

许多人认为审查 PRs 是一件容易的事情。在现实中，它和编写代码一样具有挑战性；你经常在你的头脑中解决与你正在审查的代码相同的问题。

希望这篇文章能给你一些可行的建议，告诉你如何更快地开始复习 PRs。编码快乐！

*原载于 2021 年 12 月 5 日*[*【https://isamatov.com】*](https://isamatov.com/review-pull-requests-faster/)*。*