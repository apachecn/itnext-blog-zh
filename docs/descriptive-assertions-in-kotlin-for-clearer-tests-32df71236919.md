# Kotlin 中用于更清晰测试的描述性断言

> 原文：<https://itnext.io/descriptive-assertions-in-kotlin-for-clearer-tests-32df71236919?source=collection_archive---------4----------------------->

我已经写了关于科特林的[模仿。在那篇文章中，我用](https://hceris.com/mock-verification-in-kotlin/)[心房](https://github.com/robstoll/atrium)来写我的断言。此后我给了 [Strikt](https://github.com/robfletcher/strikt) 一个尝试，这是另一个很酷的小库。与此同时，我在工作中使用了 AssertJ ,所以我最近有机会尝试了很多！

我觉得有两个小技巧值得分享:

*   数据类的断言
*   自定义断言

# [数据类的断言](https://kotlinlang.org/docs/reference/data-classes.html)

我们的函数接收并返回数据类。这意味着我们的测试通常会期望一个这样的类的特定实例。对于断言，我们首先使用`isEqualTo`来比较整个实例。

随着类变得越来越复杂，这种方法就成了一个问题。也许它们包含其他实体，或者有列表或地图。生成一个合适的实例来让`isEqualTo`开心最终会是一项繁重的工作。

相反，我们只想检查一些属性。我更喜欢在一个测试中避免多重断言，但是在这种情况下，我认为这是不可避免的。这是我为 *AssertJ* 使用的解决方案，随后是为 *Strikt* 使用的解决方案

我真的很喜欢 *Strikt* 解决方案的紧凑性。公平地说，我们可以用`apply`来压缩*资产*资产。但是我更喜欢第二个。

错误消息是什么？拥有不同断言的一个缺点是，您会得到一个缺乏上下文的错误消息:

如果不仔细看测试，谁能理解呢？幸运的是，我们的解决方案提供了更有意义的信息:

好多了，不是吗？

# 自定义断言

让断言说得更多的一个方法是根据我们的需要扩展它们。例如，我最近一直在玩 [Arrow](https://arrow-kt.io/) (我相信它本身可以成为博客帖子的无尽来源)。我尽可能避免使用异常，而是使用数据类型。或者 *Monad* ，好像我真的不知道自己在说什么。

无论如何，我有一个包含我想要测试的功能的存储库。

`fun find(id: Int): Either<Int, RecipeDetails>`

我正在调用这个方法，并且想要断言我得到了一个有效的返回(`Either.Right`)。然后，我想检查输出的一些属性:

这段代码有点不尽人意。我必须检查该值是否是一个`Right`值，转换它，然后在开始断言之前获取实际内容。对我们来说幸运的是， *Strikt* 允许您编写[定制断言](https://strikt.io/wiki/custom-assertions/)，非常适合这种情况。在敲了一会儿键盘后，我找到了这个助手:

我是这样使用的:

变化不大。然而，它增加了这个小片段的可读性，并使其背后的意图更加清晰。我喜欢用心良苦的代码。

对[选项数据类型](https://arrow-kt.io/docs/arrow/core/option/)也可以这样做:

# 为什么呢？

我们完成了什么？在我看来，有两件事:

*   测试将更好地讲述测试的内容和原因。
*   当他们失败时，就更容易找出原因。

*原载于 2019 年 9 月 28 日*[*https://hceris.com*](https://hceris.com/descriptive-assertions-in-kotlin/)*。*