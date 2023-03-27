# 关于排版中地图和集合的思考

> 原文：<https://itnext.io/thoughts-on-map-and-set-in-typescript-efd43c3bf2ad?source=collection_archive---------1----------------------->

![](img/492fc2edcd77879015bc2c77833c1e08.png)

在 [Unsplash](https://unsplash.com/photos/BYGLQ32Wjx8?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上由[米米·蒂安](https://unsplash.com/@mimithian?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)拍摄的照片

用 ES6 [映射](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)和[设置](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)引入到 JavaScript 中。它们最终为您的 web 应用程序带来了更复杂的数据结构。正因为如此，他们是一个非常受欢迎的语言补充。长久以来，阵列被迫解决存在的每一个问题(然而，我看不出这种情况会很快发生)。但是，在我看来，他们在执行方面确实没有达到目标。

# 地图和布景快速介绍

让我们从头开始:什么是地图和集合？映射是一种保存键值对的数据结构。在这方面它有点像一个物体，但有一些主要的区别:

*   密钥不仅限于字符串、数字和符号
*   map 是一个集合，意味着它有一个大小、一个顺序，并且可以被迭代。
*   映射被设计用来处理可选键，而不是必需键。这在使用 TypeScript 和确保类型安全时更为重要

集合是保证所有值都是唯一的集合。最好将它描述为一个键和值总是相同的映射。集合在检查值是否存在以及计算两个集合之间的差异或重叠方面非常有用。

# 哪里出了问题

基于以上几点，人们会觉得地图和布景都是语言的惊人补充。如果不是因为他们的界面如此斯巴达化，他们会的。让我们从地图开始。对于这个论点，我将介绍一个地图的用例示例:

包含计数器动态集合的映射

比方说，我想增加某个键的计数器，当这个键没有被预置时，它应该变成 1。在 ES6 地图上，这条线周围的某个地方:

😢 😢 😢

当您意识到 map 的全部目的是处理键值对时，这需要大量的代码！例如，在 [immutable.js](https://github.com/immutable-js/immutable-js) 中，这将通过一个更新调用来编写:

这就是使用 map 时键-值对应该有多简单

> 注意:我确实喜欢 immutable.js，但并不是说你应该在“普通的”地图和集合上使用它。不可变和可变数据结构是不同的概念，不仅仅是可互换的。

另一个大吝啬鬼的例子是在地图和集合上缺少变换方法。没有`map`来转换集合中的每个值，也没有`filter`从映射或集合中排除值。假设我们想要删除所有为零的计数器。对于 ES6 映射，我们需要一个带有新映射的 for 循环:

在函数式编程和 lambda 时代，for 循环中的 if 语句

而使用简单的过滤方法，这可以用一行和一个箭头函数来完成。对我来说，这是一个谜，为什么数组得到了所有这些方法，他们没有把它们放在集合和地图上。

如果它只是一个数组…

我想举的关于地图的最后一个例子是缺少一个`merge`功能。映射的一个常见任务是将两个映射合并在一起，并以某种方式解决冲突的键(通常用后一个或给定的自定义合并覆盖)。对于 ES6 映射，这将再次需要 for 循环和一些 if 语句。人们可能希望用`map.merge(otherMap)`简单地做到这一点。

上述所有要点也适用于 Set，因为它是一个非常类似于 map 的数据结构。他们这里的主要额外吝啬鬼是缺少一个`merge`、`except`和`both`。在编写关注多组值之间的差异重叠的代码时，Set 通常非常出色，但在 JavaScript 中却不是这样。

最后:记住映射和集合都是有序集合？除了将其转换为数组、对数组进行排序并重新创建地图或地图集之外，没有其他方法可以对它们进行排序。为什么要显式地实现一个有序地图，却不允许用户订购他们的地图呢？

# 如何正确地做它

在所有的抱怨之后，是时候谈谈如何正确地做这件事了。首先:在设计标准库时，保持接口的一致性。如果我能以某种方式映射、过滤和减少一种类型的集合，我希望所有其他类型的行为都相似。集合、映射和数组都应该有可比较的接口。理想情况下，它们应该是一个 [ICollection](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.icollection-1?view=netcore-3.1) 接口，强制所有集合保持一致(对于内置和定制实现)。

还有一点就是思考那些集合为什么存在。如果 collection soul 的目的是，比方说，处理键-值对，我希望有一切可以用它们的键来创建、删除和修改值。当一个集合主要用来处理唯一值的集合时，我希望所有的东西都能真正使用这个功能，而不必实现像`merge`和`except`这样的标准库函数。尤其是在像 JavaScript 这样被设计成易于使用的高级脚本语言中。

在本文的最后，我提供了一个代码片段，实现了大部分提到的特性。其实大部分真的很容易。这只是一个痛苦，你必须考虑这些基本功能，而你的大脑应该专注于你试图解决的问题。

# 结论

你对 JavaScript 中的 Map 和 Set 有什么看法？你经常使用它们吗？或者你和我一样，仍然在使用第三方库作为数据结构？在我看来，新的 ES6 数据结构还不够好，不够实用。我将用我提到的大多数特性的示例实现来签署这一篇。只是为了证明第一次做对并不是什么大事。