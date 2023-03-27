# 迭代器和生成器一起工作非常好

> 原文：<https://itnext.io/iterators-and-generators-work-great-together-80b196aaee2d?source=collection_archive---------4----------------------->

![](img/53c094ec0de876cc2f45206b38d774ab.png)

迭代器和生成器都是有趣的 JavaScript 特性。当你把它们结合在一起使用时更是如此。在这篇博文中，让我们重温一下对生成器和迭代器的理解，看看如何将它们结合起来编写优雅的 JavaScript 代码。

# 迭代程序

简单地说，迭代器是一个符号，你可以用它来使任何对象*可迭代*。可迭代对象可以使用`for...of`循环进行迭代。首先，让我们回顾一下这些符号是什么。

## 标志

符号是一种保证唯一的原语。它的主要用途之一是用作字典集合(地图或普通对象)的键。此外，它总是独一无二的这一事实使它成为其他有趣用例的良好候选。现在让我们探讨一下我所说的真正独特的符号是什么意思:

上面的代码将打印出来:

```
*// String Unique is not equal to Symbol(Unique)**// Symbol(Unique) is not equal to Symbol(Unique)*
```

首先`if`将值为`Unique`的字符串`notSymbol`与描述值为`Unique`的符号实例进行比较。如你所见，这两个变量不相等。一个更有趣的例子是第二个`if`，我们在比较两个看起来完全相同的符号。但是，即使两个符号具有相同的描述值，它们也不相等。描述仅用于提高代码的可读性。

## 定义迭代器和可迭代对象

JavaScript 提供了`Symbol.iterator`静态符号，我们可以用它来使任何对象可迭代。让我们来看看这个例子:

首先让我们定义一个我们将要迭代的类:

`PetShelter`是一个简单的类，包含一个名为`pets`的对象数组。在我们的`pets`属性下面，我们使用`Symbol.iterator`作为键添加了另一个属性。我们给它分配一个函数，该函数返回一个新的`PetIterator`实例。

现在让我们看看`PetIterator`的定义是什么样的:

`PetIterator`只是另一个只包含一个方法的类——`next`。每个迭代器都需要有`next`方法。同样，这个方法必须返回一个包含两个属性的对象:`done`和`value`。您可能已经猜到了这些属性的用途:

*   `done`是一个布尔属性，表示我们是否已经到达迭代的末尾。
*   `value`用于返回当前迭代项的值。

## 一起使用它们

现在我们有了 iterable 和 iterator，让我们看看如何一起使用它们:

很简单，对吧？`for...of`循环自动创建并使用了由`PetShelter`类提供的迭代器的一个新实例。

上述代码会将以下内容打印到控制台:

```
*// { pet: { type: 'cat', name: 'Emma' } }**// { pet: { type: 'dog', name: 'Bubbles' } }**// { pet: { type: 'dog', name: 'Hank' } }**// { pet: { type: 'cat', name: 'Leo' } }*
```

# 发电机

生成器是 EcmaScript 2015 规范中引入的新语法功能之一。就像`async`一样，通过在客户端认为合适的时候暂停和恢复程序，生成器可以用来控制程序的执行流。让我们看一个如何定义生成器的例子:

正如您所看到的，生成器只是普通的函数，有一些小的不同:

*   为了定义一个生成器，你必须在关键字`function`后加上`*`。
*   `yield`可以在生成器内部使用来暂停执行。另外，`yield`关键字之后的任何值都被传递回调用生成器的客户机。

现在让我们看看如何使用我们的发电机:

下面是上面代码的输出:

```
*// Hi**// What's your name?**// I'm done here*
```

这里有几件事需要注意:

*   调用`generatorSample()`并不运行生成器函数，而是返回生成器的一个新实例。
*   一旦我们的生成器内部的代码遇到`yield`，它就会停止。我们使用生成器的`next`方法继续执行。

下面是`next`对象的结构:

```
{value: 'Hi' *//yielded value,*done: **false** *//whether the generator is done executing or not*}
```

## 使用生成器创建迭代器

看着眼熟？希望现在你开始看到迭代器和生成器之间的联系。坦率地说，ES 2015 引入生成器的主要原因之一是让创建迭代器的过程变得更加容易。事实上，要将我们的`PetShelter`变成一个 iterable，我们所要做的就是让[ `Symbol.iterator` ]函数返回一个生成器的新实例。现在，让我们来看看如何实现这一点:

就像前面的例子一样，上面的代码将产生以下输出:

```
*// { pet: { type: 'cat', name: 'Emma' } }**// { pet: { type: 'dog', name: 'Bubbles' } }**// { pet: { type: 'dog', name: 'Hank' } }**// { pet: { type: 'cat', name: 'Leo' } }*
```

如您所见，我们不再需要为我们的`PetIterator`定义一个单独的类。我们要定义的只是一个生成器，它使用一个`for`循环来迭代`pets`属性。接下来，我们调整了`[Symbol.iterator]`函数来返回一个新的`PetIterator`生成器实例。

我们能够使用生成器来代替迭代器，因为生成器已经有了一个名为`next`的方法，它返回一个具有`done`和`value`属性的对象。

这个帖子到此为止！在这里，我们简要回顾了迭代器和生成器的基础知识，以及如何使用它们轻松创建可迭代对象。我希望这篇文章对你有用。

感谢您的阅读！

*原载于 2019 年 7 月 24 日 https://isamatov.com**的* [*。*](https://isamatov.com/iterators-and-generators-work-perfect-together/)