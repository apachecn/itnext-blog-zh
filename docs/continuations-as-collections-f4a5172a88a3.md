# 作为集合的延续

> 原文：<https://itnext.io/continuations-as-collections-f4a5172a88a3?source=collection_archive---------5----------------------->

![](img/8ba0e7e800ce56b5da8263d6d854ef6d.png)

照片由[杰西卡·鲁斯切洛](https://unsplash.com/@jruscello?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/collection?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

## 为延续创建收藏库

上周我发表了一篇关于 TypeScript 中的[延续的文章。在那篇文章中，我提到延续(不同于承诺)就像是随着时间的推移而开始存在的价值集合。在本文中，我想进一步探讨这个问题，看看您的传统数组与延续有多少共同点。](/continuations-in-typescript-db18402010bc)

实现 continuation 的集合库的最大限制因素是我们不知道 continuation 将要输出的值，甚至不知道有多少。因此，并不是所有被广泛接受为标准集合操作符的方法都可以实现延续。然而，延续的异步特性也允许定义新的独特而有趣的方法。

[](/continuations-in-typescript-db18402010bc) [## 打字稿中的延续

### 提取打字稿中的常见模式

itnext.io](/continuations-in-typescript-db18402010bc) 

# 快速回顾

延续是对随着时间推移而产生价值的事物的抽象。本质上，它只不过是一个“东西”,只要你给它一个回调，它就会调用这个回调。这意味着延续与承诺非常相似，主要区别在于承诺总是只产生一个值，而延续可以产生许多值。

键盘延续的一个例子

通过将(回调的)eventlistener 的概念建模为包含值的数据结构，我们获得了向其添加方法的能力。其中最重要的是`map`将一个延续从一种类型转换成另一种类型的延续，以及`then`将多个延续链接成一个更大的延续。

`keyboard`、`map`、`then`用法举例

延续的接口

# 延续的集合组合子

有多种方法可以将两个集合合并为一个集合。最简单的方法是用两个集合的所有元素创建一个集合。这个操作通常被称为`concat`，它首先给出`a`的所有元素，然后给出`b`的所有元素。我们可以做一个函数，取两个延拓，做一个延拓，输出所有的值。只有排序是按照它们输出的顺序，而不是所有的`a`先输出，然后是`b`输出。因为这个微小的差别，我决定把它叫做`merge`而不是`concat`以避免混淆。

`intersect`操作员是我们的下一个操作员。这个函数确保得到的集合只包含那些在两个输入中都存在的值。我们可以通过跟踪输出来为延续实现这一点，并且只在另一个延续已经输出了一个值的情况下才输出这个值。注意，这个函数只输出已经输出的值，当输出发生时，不输出仍然必须(但是将会)由另一个延续输出的值。

`except`仅输出其他延续尚未输出的值。它的实现看起来与`intersect`非常相似，唯一的区别是颠倒的条件。请注意，这只是防止输出已经由另一个延续输出的值，而不是另一个延续将在特性中输出的值。

函数式编程中另一个常见的集合组合器是`zip`函数。`zip` takes 将两个集合组合成一个集合对。该函数也可用于延续:

# 具有延续的子集合

我们还可以创建包含原始延续值子集的延续。这些功能中最常见的是`filter`、`skip`和`take`，所有这些功能在之前的文章中都已经介绍过了。

`filter`和`skip`用于延续

有了上面例子中的类似结构，我们可以实现更多的选择方法，比如`[take_while](https://gist.github.com/WimJongeneel/33adf25ff848187faad6170f4b98ed7d)`、`[take_until](https://gist.github.com/WimJongeneel/0e3c86a7d54a9727d5871487aadf4086)`、`[skip_while](https://gist.github.com/WimJongeneel/e3380b8d69e0be2de1e7c33be10e1ef7)`和`[skip_until](https://gist.github.com/WimJongeneel/e191d995514dee46ff6c9a4bf67798c8)`。`first`可以用`take(1)`创建，因为只输出一次的延续和输出多次的延续是一个类型。我们不能实现`last`、`take_last`或任何其他选择，因为我们永远不知道一个值是否是输出的最后一个值。

`distinct`是另一个操作符，它将给出一个不需要谓词的子集合。`distinct`可用于赋予`intersect`和`except`函数更集合论风格的行为(从不输出一个值超过一次)。

# 延期付款

我们将关注的集合的最后一个属性是它们在分析函数中的使用能力。因为 continuations 是异步的，我们不能只计算它们(最终)包含的所有值的平均值或总和。我们可以做的一件事是将数字的延续转换成输出新值的延续。例如，参见下面的`sum`功能。每次`c`输出时，新的延续将输出`c`已经输出的所有值的新总和。

每次内部`cont`输出时，输出总和的新值的`cont`

分析数字集合的另一种方法是使用滚动欠款。滚动重置是对该值的最后一个`n`值的重置。这种想法非常适合异步填充的集合，因为当内部延续输出一个新值时，我们可以输出滚动更新的下一个值。

具有动态步长的滚动平均值

# 结论

如你所见，continuation 可以被看作是集合，包括我们通常用它们做的所有有趣的事情。推理和实现这些延续的方法是一个很好的方法来提高你对它们的理解。