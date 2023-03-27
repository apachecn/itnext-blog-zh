# 使用递归和归约器

> 原文：<https://itnext.io/an-emoji-lovers-guide-to-functional-programming-part-6-5c1d441d36af?source=collection_archive---------2----------------------->

![](img/340a0bbf86d5ec4926929328322c8f60.png)

## 表情符号爱好者的函数式编程指南

## 功能组合、减压器和换能器模式的高级使用。

*用表情符号和 JavaScript 学习函数式编程。代码示例应该足够简单，不需要任何先验知识就可以理解，但是我可以想象它看起来有点奇怪。还有，JavaScript 实际上不允许表情符号作为 JavaScript 变量名。出于这个原因，这些代码示例不会在没有修改的情况下运行。*

*   [*用食物表情符号制作便便。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-1-241d8d4c9223)
*   [*把暴风云变成晴朗的云。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-2-800b438c7ce3)
*   [*用减速器造独角兽！*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-3-ef78e3156e)
*   [*管道操作员现在！*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-4-735c17ca4113)
*   [*为食肉动物过滤肉类。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-5-a6bc3324a839)
*   [*使用递归与归约。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-6-5c1d441d36af)

仍然有一些情况，你可能仍然需要一个`for`循环，或者你是这样想的。为什么不用递归？因为复杂？那又怎样？自从 JavaScript 有了尾递归，你就可以在调用一个反复调用自己的函数时防止内存溢出(**编辑:**尾递归被移除)。

不过，递归是一个复杂的话题，不一定是函数式编程的一部分。相反，我将解释我们如何使用归约器来模拟递归。

让我们先看看我们的老朋友，那个讨厌的`for`循环，然后尽快切换到一个功能性的 reducer 模式:

```
food = [🍔, 🍟, 🍪]
initialValue = 🍽️reducedValue = initialValuefor (let i = 0, l = food.length; i < l; i++) {
    reducedValue += food[i]
}💩 = reducedValue
```

看起来很合法，对吧？这一过程将食物减少为其他东西；在这种情况下，减少值为💩。我们现在做的是求和。把我们的食物和盘子加在一起，我们得到💩。

我们可以把这种缩减归结为一个叫做“缩减器”的`eat`函数:

```
eat = (combined, value) => (
    combined + value
)
```

Reducers 是接受两个值并返回一个新值的函数。第一个值是总计或累加器，第二个值是循环中的当前项。

数组有一个`reduce`方法，它接受一个缩减器和一个初始状态。这是一个类似的概念，你可能在流行的州立图书馆 [Redux](https://redux.js.org/) 中见过。

```
💩 = [🍔, 🍟, 🍪].reduce(eat, 🍽️)
```

第一个值是我们的`eat`函数，第二个值是我们的初始状态:🍽️.

这是一个非常简单的缩减器，但是我们还可以做更多的事情，比如创建新的数组或者把一个数组变成一个对象，数字，字符串，你能想到的！

下面是从`reduce`创建一个`filter`方法的样子:

```
filterReducer = (
    condition,
) => (
    combined,
    value,
) => (
    condition(value)
    ? combined.concat(value)
    : combined
)
```

如果`condition(value)`是`true`，那么用那个值创建一个新数组；否则，返回前一个数组。我们在这里所做的是创建类似于`Array.prototype.filter`的东西。它在第一次被调用时接受一个条件，然后返回一个 reducer。

让我们看看它的实际效果吧！：

```
handFilterReducer = (
    filterReducer(value => (
        value
        .includes(✋)
    ))
)[🤷, 💁, 🙋] = (
    [🤷, 💁, 🙋, 🙍]
    .reduce(handFilterReducer)
)
```

`handFilterReducer`检查数值是否包含手牌。然后，我们将它传递到我们的缩减器中，并过滤掉没有手的人。

首先，没有初始状态。当您有一个数组时，它将第一个数组值与初始状态相结合，但是如果您没有传入一个数组，那么`reduce`将第一个数组值与第二个数组值相结合。

这种方法使用了我们在第一部分的[中讨论过的叫做合成的东西。这在函数式编程中非常重要，因为它允许你通过组合函数来创建新的函数。](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-1-241d8d4c9223)

因为我们将`filterReducer`开发为一个返回函数的函数，所以我们能够像在第一部分中编写`add`来创建`addLightning`一样编写`filterReducer`来创建`handFilterReducer`。

像`filterReducer`这样的缩减器对于状态管理很有用。如果你的值对这个缩减器没有任何作用，那么这个缩减器就返回之前的状态。

让我们看一个状态管理的例子:

```
countReducer = (
    total,
    value,
) => (
    value
    ? total + 1
    : total
)4️⃣ = (
    [🤷, 💁, 🙋, 🙍]
    .reduce(countReducer, 0)
)
```

现在我们的`total`是数组项数的和。**挺没用的对吧？**

让我们来看一个真实的例子！：

```
partitionReducer = (
    getPartitionName,
) => (
    paritions,
    value,
) => ({
    ...partitions
    [getPartitionName(value)]: {
        ...partitions[getPartitionName(value)]
        value,
    }
})
```

尽管看起来很复杂，但这是一个将我们的状态分割或拆分成多个其他状态的缩减器。它需要一个初始函数，就像我们的`filterReducer`；在这种情况下，它是一个函数，它获得了我们将要用来划分状态的名称。

综上所述，`partitionReducer`拿了我们的状态，又创造了一个新的。我们的状态由一个数组对象组成，我们给它多少数组，`partitionReducer`就可以维护多少数组。

这是它在使用中的样子:

```
bunchPartitionReducer = (
    partitionReducer(value => (
        value
        .isBunch
        ? 'multiple'
        : 'single'
    ))
)initialState = {
    single: [],
    multiple: [],
}{
    single: [🍈, 🍋],
    multiple: [🍇, 🍒],
} = (
    [🍇, 🍈, 🍒, 🍋]
    .reduce(
        bunchPartitionReducer,
        initialState,
    )
)
```

再次注意构图。我们创作了`partitionReducer`来创造`bunchPartitionReducer`。我们用`initialState`初始化我们的 reducer，并传递给它一个水果数组。有些水果是成串的，比如🍇和🍒而另外两个是单身:🍈和🍋。

我们从一个包含两个数组的对象开始，最终我们得到了一个包含两个由`value.isBunch`分隔的数组的对象。

如果你在这一点上迷失了，不要担心，从这里开始只会变得更糟；实际上不，我们结束了。这是给你一个减速器的快速概述。它们功能强大得令人难以置信，并允许许多功能概念，例如我们在第 3 部分和第 4 部分中讨论过的管道化。

在我们谈到的所有函数数组方法中，它们都可以用`reduce`来编写。这个系列已经介绍了基础知识，但是`reduce`是其中最强大的。这就是为什么这些例子如此疯狂。

换能器模式，另一种形式的功能流水线，特别依赖于减速器。实际上，它接受一个缩减器并返回一个缩减器。我们实际上已经创建了两个与传感器非常相似的函数:`filterReducer`和`partitionReducer`。这些可组合的函数允许复杂的函数管道，你将在像 RxJS 这样的库中看到。随便翻翻 [RxJS 的运营商列表](https://rxjs-dev.firebaseapp.com/api)，你就明白我的意思了。

如果您只能在工具箱中选择一个工具来进行函数式编程，请选择 reducer。您可以根据需要创建所有其他工具。

# 递归

您可能已经注意到，我们的`reduce`方法类似于递归函数，因为它依赖于前一个值来计算下一个值。因为你不需要自己管理递归，所以归约器是推理递归的一种更简单的方法。你一直不知道就明白了！

**感受递归！**

# 更多阅读

如果您对与函数式编程相关的更多主题感兴趣，您应该看看我的其他文章:

*   [防止现有系统发生重大变化](/how-to-safely-refactor-old-code-part-1-a1a853263fec)
*   [使用 Redux 的秘密:createNamespaceReducer](https://medium.com/@Sawtaytoes/the-secret-to-using-redux-createnamespacereducer-d3fed2ccca4a)
*   [在 React 组件中使用 Redux 还原剂](https://medium.com/@Sawtaytoes/using-redux-reducers-in-react-components-4e92985dd9cb)
*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)
*   [RxJS 和可观察的 Flic 按钮](https://medium.com/flicblog/flic-buttons-and-the-observable-customization-using-rxjs-2214bc53d407)