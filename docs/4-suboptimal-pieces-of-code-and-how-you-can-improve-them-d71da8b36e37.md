# 4 段次优代码以及如何改进它们

> 原文：<https://itnext.io/4-suboptimal-pieces-of-code-and-how-you-can-improve-them-d71da8b36e37?source=collection_archive---------1----------------------->

![](img/2ec7f891160c30dec0c4ce5d518572cd.png)

卢卡·布拉沃在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

## 通过这些例子了解如何提高代码质量

代码质量很重要。事实上，代码的质量会影响代码库的安全性、可靠性。好的代码可以在五分钟内修复一个 bug，而不是五个小时。写好代码不仅仅影响你。如果你在团队中工作，你的代码也会对你的同事产生直接影响。

然而，事实证明，编写高质量的代码是说起来容易做起来难。尽管很多代码都完成了它们的工作，但它们并没有那么好。大多数时候代码是不可读或不可维护的。或者更糟，两者都有。

这就是为什么我们将在本文中讨论四个次优代码的例子，并向您展示如何改进每个例子。

# 1.重复的 if 语句

我经常看到的一种形式的次优代码是看起来差不多的多个 if 语句。这种情况在处理表单时经常发生。

让我们看看下面的例子:

有一件事可能会引起你的注意，那就是所有这些 if 语句基本上都是一样的。唯一不同的是它检查和更新的变量。那不是解决这个问题的最佳方法。

当我们需要对另外三个字段进行同样的检查时会发生什么？我们要不要再添加三个几乎相同的 if 语句？这段代码的问题在于它不可维护。当一个 if 语句改变时，我们可能必须改变所有的 if 语句。这增加了引入 bug 的变化，因为在调整所有 if 语句时，有一个 if 语句可能会被遗忘。最重要的是，代码也不太可读，因为它太长了。

## 我们如何改进这段代码？

理想情况下，我们希望我们的代码遵循 DRY 原则(不要重复自己)。为了不重复，我们必须去掉所有多余的和不必要的假设语句。

我们可以通过使用循环来做到这一点。这样我们只需要使用一个 if 语句。

让我们看看如何改进我们的代码:

我们实现了一个遍历所有字段的循环。我们现在只需要一个 if 语句，而不是必须为每个字段都编写 if 语句。这使得我们的代码更容易维护。当需要检查一个额外的字段时，需要做的就是向*字段*数组添加一个新值。这样，您可以添加一个额外的字段，而不必复制粘贴任何代码。

另一个好处是我们的代码可读性更好。不再有重复的代码，我们需要更少的代码行来实现相同的结果。

# 2.魔法无处不在

下面的例子包含了太多的魔法。如果你正在看这段代码，你可能需要花点时间来理解它的意思。

第一行的那些 0 和 6 是什么意思？我们实际上返回的是什么？第一次看这段代码的时候，没有那么大的意义。

## 我们如何改进这段代码？

如果我们去掉所有神奇的字符串和数字，我们可以让这段代码可读性更好。

改进后的代码可能如下所示:

看到我们如何消除所有的神奇字符串和整数了吗？代码变得更加易读易懂。我们还添加了一个额外的功能，检查它是否在周末，这也提高了可读性。最重要的是，改变数字或字符串的值要容易得多，因为它不是重复的。

更改幻数或字符串的值很容易出错，因为同一个值经常会在应用程序的不同位置被多次使用。如果您忘记更改这些值中的一个，您很可能会引入一个新的 bug。

# 3.环境变量

我经常看到的一个错误是变量是基于某个参数设置的，让我们以请求 URL 为例。

某段代码可能如下所示:

API 的基本 URL 是基于当前 URL 确定的。如果当前 URL 包含*测试*，我们使用 API 的测试环境。

如果这种代码对您来说很熟悉，那么您应该知道有一种更好的方法来解决这样的问题。

## 我们如何改进这段代码？

当您使用包含所有环境变量的文件时，会使您的代码更加整洁。你可能听说过*。env* 文件。这个文件包含所有的环境变量。每个环境都有自己的*。env* 文件。这样我们就可以去掉之前使用的不必要的难看的 if 语句。

除了你的代码更干净以外，它也更容易维护。如果创建了一个新环境，您需要做的就是创建一个新的*。env* 文件。

如果你没有使用环境变量，你需要遍历所有的代码来检查基于某种环境变量设置的变量。每次创建新环境时，您都必须这样做，这是浪费时间。

# 4.不可读的代码

变量名对代码的可读性有很大的影响。我怎么强调变量名的重要性都不为过。

命名的问题是，当你写代码的时候，你正在做命名的决定，但是那是那些决定影响最小的时候。

在查看该代码示例时，您可能会问自己一些问题。这个*列表*变量里有什么？这个方法返回什么？光看名字是回答不了这些问题的。你必须深入研究代码才能找到答案。

## 我们如何改进这段代码？

通过给所有变量起一个有意义的名字，这段代码会得到显著的改进。写代码的时候要记住，选择好的名字需要时间，但是省的时间比花的时间多。

为了向您展示有意义的名称和不好的名称之间的区别，请看改进后的示例:

同样的代码通过这种方式变得更加易读，对吗？