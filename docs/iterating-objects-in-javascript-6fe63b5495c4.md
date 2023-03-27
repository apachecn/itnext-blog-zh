# 在 JavaScript 中迭代对象

> 原文：<https://itnext.io/iterating-objects-in-javascript-6fe63b5495c4?source=collection_archive---------4----------------------->

![](img/d8621302d7515082000594c645bffee2.png)

你有没有想过如何在 JS 对象(不是数组)中迭代属性？嗯，我有。

这可能不是最常见的情况，但有时您将数据结构保存为对象而不是数组，并且需要遍历该结构的每个属性。第一次遇到这种情况，我上网搜了一下，结果犯了一个错误。在帮助人们学习 JS 和构建他们的代码时，我已经看到这个错误很多次了；因此，我决定写这篇文章。

# 不仅仅是一个“for … in ”?

虽然使用一个`for … in`确实会遍历你的对象属性，但问题是它实际上会比你预期的多做一点。你的目标可能是这样的:

虽然这可能经常正确工作，但事实是这种方法存在隐患。`for … in`将遍历你的对象和它的原型。你不知道什么是原型吗？这也是了解 JS 特别有意思的地方。使用来自 [MDN](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes) 的词语:

> *原型是 JavaScript 对象相互继承特性的机制*

所发生的是，如果你使用`for ... in`，你也从你的对象(“父对象”)迭代通过原型链的属性。

即使当你创建和开发你自己的数据结构时，这可能不是问题，但是当使用插件和维护大的存储库时，这可能会变得更加复杂。

想象一下这样一个场景，你有一个带有原型的物体。你从某处得到它。你可以检查，但不一定看到它包含这两个属性。你只会在`for … in`遍历你没有预料到的属性时看到这一点！

可以看出，尽管我们从未为`objectFromSomewhere`定义过`a`、`b`和`c`，但它拥有来自`firstObject`的这些，这实际上是它的原型。你甚至可以看到它先是迭代了`d`和`e`(它自己的属性)，后来又去了原型的属性。所以，如果你正在使用`for … in`并且只想遍历你的对象的属性，那就有问题了。

有一些方法可以防止这种情况发生，但是当你处理第三方库和大代码库时，用`for … in`进行迭代是有风险的。

# 如何防止这种情况？

嗯，其实很简单。解决这个问题的方法也不止一种。但是让我们为您提供一个快速安全的解决方案。在这里我给你介绍`[Object.keys()](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)`和`[Object.values()](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Object/values)`。顾名思义，第一个函数将以数组的形式为您提供对象的键(您可以按照自己的意愿进行迭代)。请记住，这些方法也会迭代函数(您必须手动检查这一点)。

后一个将为您提供对象的值，也是以数组的形式。

要同时迭代键和值，还可以使用`[Object.entries()](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Object/entries)`，它返回一个用`[key, value]`数组填充的数组。

就这么简单！这样你就保证了对迭代的控制。

# 如果我被困在“for … in”中怎么办？

出于任何原因，如果您需要坚持`for … in`，您将需要在每个循环中添加一个新的检查。你可以用`hasOwnProperty`问一个对象这个属性是自己的还是来自他的原型。

今天就到这里了，伙计们！尽管很简单，但这种知识可以让你在开发过程中省去一些麻烦。

附注:如果你想更深入一点，我真的推荐阅读《你不知道的 JS》第五章(实际上是整本书)