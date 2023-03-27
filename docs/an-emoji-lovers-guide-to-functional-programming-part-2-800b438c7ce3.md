# 把暴风雨的云变成晴朗的云

> 原文：<https://itnext.io/an-emoji-lovers-guide-to-functional-programming-part-2-800b438c7ce3?source=collection_archive---------0----------------------->

![](img/340a0bbf86d5ec4926929328322c8f60.png)

## 表情符号爱好者的函数式编程指南

## 数组映射、数组归约和使用管道的函数组合的高级用法

*用表情符号和 JavaScript 学习函数式编程。代码示例应该足够简单，不需要任何先验知识就可以理解，但是我可以想象它看起来有点奇怪。还有，JavaScript 实际上不允许表情符号作为 JavaScript 变量名。出于这个原因，这些代码示例不会在没有修改的情况下运行。*

*   [*用食物表情符号制作便便。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-1-241d8d4c9223)
*   [*把暴风云变成晴朗的云。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-2-800b438c7ce3)
*   [*用减速器建造独角兽！*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-3-ef78e3156e)
*   [*现在是管道操作员！*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-4-735c17ca4113)
*   [*为食肉动物过滤肉类。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-5-a6bc3324a839)
*   [*用归约器递归。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-6-5c1d441d36af)

我们从上次的`add`和`subtract`方法开始:

```
add = additive => item => item + additive
subtract = subtractor => item => item - subtractor
```

我们谈到了使用多种`map`语句来将我们的乌云变成晴朗的云:

```
[⛅, 🌦️] = (
    [🌩️, ⛈️️]
    .map(subtract(⚡))
    .map(add(☀️))
)
```

但是你知道吗，我们可以通过加减法去掉一个`map`:

```
addSun = add(☀️)
removeLightning = subtract(⚡)[⛅, 🌦️] = (
    [🌩️, ⛈️️]
    .map(item => (
        removeLightning(
            addSun(item)
        )
    )
)
```

我们为什么要这么做？我们可以保持我们的两个`map`声明，每个人都会很高兴！通常，这就是你要做的，但是在某些情况下，你可能需要不同的方式来使用`compose`或者`pipe`来链接。它们执行完全相同的功能，除了`pipe`以类似于[波兰符号](https://en.wikipedia.org/wiki/Polish_notation)的相反顺序接受参数:

```
compose = (second, first, item) => second(first(item))pipe = (item, first, second) => second(first(item))
```

通常这些返回函数不是直接取`item`:

```
compose = (second, first) => item => second(first(item))pipe = (first, second) => item => second(first(item))
```

我们将使用`pipe`作为例子，因为它通过从左向右阅读降低了复杂性。

让我们假设我们需要微优化我们的代码，两个地图太多了。如果我们想把它放到一张地图上，我们需要组合我们的函数。正如我们在前面的例子中看到的，这可能会很快变得很糟糕！编写一个简化的`pipe`函数，我们可以完成我们的目标并保持代码整洁:

```
pipe = (first, second) => item => second(first(item))[⛅, 🌦️] = [🌩️, ⛈️️].map(pipe(subtract(⚡), add(☀️)))
```

既然我们已经弄清楚了`pipe`的基础，我们需要写一个真正的。这意味着我们必须能够给`pipe`无限多的函数，让它按顺序调用。

从功能上来说，使用`reduce`很简单，但是如果不了解基础知识，这很难推理。让我们从程序上来看:

```
item = nullupdateItem = change => {
    item = change(item)
}pipe = (...functions) => {
    [item] = functions.splice(0, 1)

    for(let i = 0, l = functions.length; i < l; i++) {
        updateItem(functions[i])
    }

    return item
}⛅ = pipe(☁️, subtract(⚡), add(☀️))
```

在这个例子中，没有闭包。我们首先直接传入云表情符号，然后添加我们的其他功能作为单独的参数。

我们在程序`pipe`中做的第一件事是删除第一个参数。那将是我们的`item`。我们将循环其余的函数，一个接一个地调用它们，并将`item`更新为最新值，直到我们完成了返回修改后的`item`的循环。相当混乱。

这段代码有很多变异。我们正在创建`item`，然后每次在代码中的其他地方循环时都改变`item`的值，而`updateItem`正在产生副作用。

这里还有一个问题:`splice`。这个函数使数组变异；取出一组值，并将剩余的值留在原始数组中。当然，当代码很小的时候，它现在是有意义的，但是当你添加更多的代码时，弄清楚`item`和`functions`数组的状态将变得更加困难。

函数式编程不再需要突变和副作用。通过移除它们，我们最终写出了更易于单元测试和多线程的纯函数。眼下，出现 bug 的可能性很高。对于纯函数，这种可能性大大降低。

下面是同样的`pipe`使用不变性、函数组合和`reduce`。

```
pipeReducer = (item, change) => change(item)pipe = (...functions) => {
    [startingItem] = functions

    return (
        functions
        .slice(1)
        .reduce(pipeReducer, startingItem)
    )
}⛅ = pipe(🌩️, subtract(⚡), add(☀️))
```

注意我们是如何使用`slice`(不可变版本的`splice`)的，这样我们就不必改变我们的函数数组。我们做的第一件事就是用解构的方法从`functions`中抽出`startingItem`。接下来，我们再次使用`slice`，这一次除了第一项之外获取所有内容。然后，我们可以在较小的数组上运行`reduce`,并对一个又一个返回值调用`change(item)`。

当`reduce`像`map`一样遍历数组中的每个值时，每次迭代都会获得前一个值和当前值。主要区别是它最终返回任何类型的单个值。同样，如果你不给`reduce`一个初始值，它将默认使用数组中的第一个值。这意味着我们可以进一步简化:

```
pipeReducer = (item, change) => change(item)pipe = (...functions) => functions.reduce(pipeReducer)⛅ = pipe(🌩️, subtract(⚡), add(☀️))
```

现在看起来真的很棒！就像`add`和`subtract`一样，我们希望使用闭包，这样`pipe`就可以生成新的函数。写管道有几种方法。一种是显式传递我们的起始项作为`reduce`的初始值:

```
pipe = (...functions) => startingItem => (
    functions
    .reduce(pipeReducer, startingItem)
)
```

另一种方法是将我们的`startingItem`添加到一个数组中，并将`concat`添加到我们的流水线函数中:

```
pipe = (...functions) => startingItem => (
    [startingItem]
    .concat(functions)
    .reduce(pipeReducer)
)
```

你选择哪一个都没关系。对于纯函数，只要相同的输入总是得到相同的输出，您总是可以尽可能地重构内部函数。使用这两种方法中的任何一种，我们最终都会得到相同的结果:

```
changeStormyToSunny = pipe(subtract(⚡), add(☀️))⛅ = changeStormyToSunny(☁️)
```

作为一个主要的好处，我们突然有了可重复使用的天气变化代码！使用我们关于闭包、函数生成器和`pipe`的知识，我们可以将 double `map`重构为一个单一的、可读的`map`。

```
add = additive => item => item + additive
subtract = subtractor => item => item - subtractorpipeReducer = (item, change) => change(item)
pipe = (...functions) => startingItem => (
    functions
    .reduce(pipeReducer, startingItem)
)changeStormyToSunny = pipe(subtract(⚡), add(☀️))[⛅, 🌦️] = [🌩️, ⛈️️].map(changeStormyToSunny)
```

**感受简单！**

## [点击这里进入第三部分！](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-3-ef78e3156e)

# 更多阅读

如果您对与函数式编程相关的更多主题感兴趣，您应该看看我的其他文章:

*   [安全重构旧代码:第 1 部分](/how-to-safely-refactor-old-code-part-1-a1a853263fec)
*   使用 Redux 的秘密:createNamespaceReducer
*   [在 React 组件中使用 Redux 还原剂](https://medium.com/@Sawtaytoes/using-redux-reducers-in-react-components-4e92985dd9cb)
*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)
*   [RxJS 和可观察的 Flic 按钮](https://medium.com/flicblog/flic-buttons-and-the-observable-customization-using-rxjs-2214bc53d407)