# 有趣的 JavaScript:使用函数式编程安全地处理未定义/空值。

> 原文：<https://itnext.io/fun-javascript-safely-dealing-with-undefined-null-values-using-functional-programming-39dcbc61eb0c?source=collection_archive---------3----------------------->

![](img/5b3bbda247da1bcdac311f2ff64697c3.png)

来源:https://www.carecreations.basf.com/

让我们假设有一个我们想要应用运算的数字，如果这个数字是未定义的/空的，我们想要避免一个错误或意想不到的结果。该操作将*添加*。

一种必不可少的方法是:

```
const number = // getting number from somewhere (DB/network/etc)let newNumber;
if (number) {
    newNumber = add(number, 5);
}// Another 50 lines below you might find something like this:if (number) {
    newNumber = add(newNumber, 3);
}
```

如果缺少数字的话，可以打印数字到控制台或者一条消息吗？

```
const revealNumber = (number) =>
    console.log(number ?? 'There was no number to reveal');revealNumber(newNumber);
```

我们都知道真正的生产代码并不像这些例子那样简单，所以实际上，我们的代码库在尝试使用某个值进行某些操作之前，会被大量的样板文件检查弄得乱七八糟。我是一个函数式编程爱好者，虽然我从来没有告诉任何人它比其他范式(比如 OOP)更好。相反，我鼓励开发人员学习这种范式，然后找到将其与 OOP 相结合的方法，取两者之长，创建一个强大的组合。

以我的拙见，JavaScript 是最好的语言之一，你可以做 OOP，也可以做函数式编程，这些风格都不会让语言感到尴尬，就像我在 C#或 Java 中使用某些函数式技术时，函数式编程有时会感到尴尬。让我们看看函数式开发人员是如何处理“可能缺失但可能存在”的问题，即可能单子。如果你是 Java 开发人员，你可能已经熟悉了*Optional<T>，如果你是 C#开发人员，你可以看看我的文章[https://it next . io/fun-cs harp-handling-with-null-values-in-a-a-safe-and-elegant-way-4 ffdf 86 a 38 C 8](/fun-csharp-dealing-with-null-values-in-a-safe-and-elegant-way-4ffdf86a38c8)，这里我讲的是我的 LeanSharp 库([https://github.com/ericrey85/LeanSharp/](https://github.com/ericrey85/LeanSharp/))。*

Maybe Monad 是一个函数式编程概念，你可以在网上读到很多，它遵循三个数学定律，左等同、右等同和结合性。我在这里不是要谈论这个，如果你想更深入地研究它的数学方面，你可以在网上找到很多。出于本文的目的，可以将 Maybe Monad 看作一个值的容器，它将允许您对该值执行安全的操作。它如何工作的基本原理是:“如果值在那里，我将执行您的操作，但是如果它不在，什么也不会发生(我不会因为一个错误而让您失望)”。一个非常简单的定义如下:

```
const isNil = (o) => o == undefined;
const notNil = (o) => !isNil(o);const Maybe = (value) => ({
    map: (fn) => notNil(value) ? Maybe(fn(value)) : Maybe(),
    bind: (fn) => notNil(value) ? fn(value) : Maybe(),
    getOrElse: (defaultValue) => value ?? defaultValue
});
```

这是什么？让我们看看…我们有一个工厂函数(也许)，它将一个值作为参数，并使用三种方法返回一个对象:

1- *map* :它接受一个函数，如果该值存在，则对该值应用该函数，并在同一个 Maybe 容器中返回一个新值(尽管是一个新实例)。如果值不存在，它将返回一个可能的包装*未定义的*。

2- *绑定*:类似于*映射，*但是应用的函数返回另一个 Maybe，而不是一个简单的值。

3- *getOrElse* :说了这么多，做了这么多，它允许我们访问包含的值，或者返回提供的默认值，如果我们在那一点上有一个 *Maybe(undefined)* 。

让我们看看如何使用这个单子。

```
const result = Maybe(number)
    .map((*x*) => *x* + 3)
    .bind((*x*) => Maybe(*x* + 5))
    .getOrElse('There was no number to reveal');console.log(result);
```

如果上面的代码对你来说有点奇怪，如果你从来没有做过函数式编程，我想是这样的，让我们用一个数组和它的函数做一些类似的事情。

```
const result = [number]
    .filter(notNil)
    .map((*x*) => *x* + 3)
    .flatMap((*x*) => [*x* + 5])[0] ?? 'There was no number to reveal';
```

我们已经将数字“提升”到一个容器中，在本例中是一个数组，我们已经排除了未定义/空的值(通过使用*过滤器*),这允许我们对其执行安全操作。我们 Maybe 的 *bind* 函数就像数组的 *flatMap* (这个函数也被称为*链*)。如果你还没有注意到，***【0】？？没有数字显示最后一行中的'*** 相当于 Maybe 的 *getOrElse* 函数。显然最后这个例子并不完全相同，但是我想给你们看一些相似的，非常相似的东西，这样你们就可以和你们可能已经熟悉的东西有一个比较点。

在结束本文之前，让我们看一个更具声明性/功能性的方法。如果你熟悉反应式编程(RxJS)，你已经知道*管道*功能。如果你不是，我现在就给你看，它是一个函数，接受一个函数数组，然后返回另一个函数。这个返回的函数取一个初始值，然后用它作为应用所有以前提供的函数的起点。这可能对你来说没什么意义，所以让我们来看看它的作用。

```
const pipe = (...*fn*) => (*n*) => *fn*.reduce((*acc*, *fn*) => fn(*acc*), *n*);
```

这样想想，如果你有两个函数， *f* ，和 *g* ，一个初始值*x；*你可以这样做:

```
const finalResult = f(g(x));
```

你也可以把它分解成这样:

```
const firstResult = g(x);
const finalResult = f(firstResult);
```

通过使用管道，您可以像这样完成相同的任务:

```
const getFinalResult = pipe(g, f);
const finalResult = getFinalResult(x);
```

我希望它现在更有意义，*管道*，有效地允许函数组合。函数式编程中使用的另一种技术称为 *currying* ，它可以被认为是一个接受另一个具有 n 个参数的函数，并将其转换为 *n 个*函数，每个函数接受一个单一参数并返回另一个遵循相同模式的函数，直到最后一个函数返回结果。让我们来看一个 curried 函数:

```
const add = (a) => (b) => a + b;
const increment = add(1);
const incrementedValue = increment(10); // 11
```

在这里，我们不会使用函数来处理我们的函数，相反，我们将手动处理它们，你可以在像 ramda 这样的库中找到这样的函数。请记住，在生产代码中，您将希望使用一个函数来为您处理函数(而不是手动处理)，尽管您应该知道，如果将*处理*函数和默认参数结合在一起，它们不会很好地发挥作用。如果您使用这样的函数，它看起来会像这样:

```
const add = curry((a, b) => a + b);
```

Currying 是一种允许函数重用的技术(就像上面看到的*增量*，以及使用类似*管道*函数的函数组合(或者它的兄弟*组合*)。现在，让我们为可以从 Maybe 对象解耦的 *map* 、 *bind* 和 *getOrElse* 提供可约束的实现:

```
const map = (*fn*) => (*o*) => *o*.map(*fn*);
const bind = (*fn*) => (*o*) => *o*.bind(*fn*);
const getOrElse = (*defaultValue*) => 
    (*o*) => *o*.getOrElse(*defaultValue*);
```

在每种情况下，我们都使用了 duck 类型并将调用委托给传入的对象。我们现在可以像这样重写代码:

```
const getFinalResult = pipe(
    Maybe,
    map(add(3)),
    bind((*x*) => Maybe(*x* + 5)),
    getOrElse('There was no number to reveal')
);console.log(getFinalResult(number));
```

注意到上面使用的 *add* 函数是 curried 版本。*管道*函数从上到下(或从左到右)阅读，如果你觉得从下到上(或从右到左)阅读更舒服，那么你想使用的函数是*撰写，*它与*管道*的功能相同，但从最底部/最右侧的函数开始。你可以这样实现*合成*(基本上是用 *reduceRight* 代替 *reduce* ):

```
const compose = (...*fn*) => (*n*) => *fn*.reduceRight(
    (*acc*, *fn*) => fn(*acc*), *n* );const getFinalResult = compose(
    getOrElse('There was no number to reveal'),
    bind((*x*) => Maybe(*x* + 5)),
    map(add(3)),
    Maybe
);
```

有几个 JavaScript 库为您提供了编写函数式程序所需的函数式结构，其中最著名的有 ramda、underscorejs、lodash 和 folktale。我鼓励您看一看它们，并在可行和可能的时候使用它们(而不是使用您自己的实现)。

我总是听到一些开发人员说“这种编程方式令人困惑”，或者“你不会用这样的方法编写一个完整的系统”。我开始理解那些类型的感觉，它们来自于对它的不熟悉，来自于对我们已经知道的东西的偏见。我曾用本文中展示的技术和其他单子编写过大型程序，比如要么单子，要么 IO 单子，等等。我逐渐意识到，这些程序比我用纯命令式方法编写的程序有更低的错误密度。我鼓励你走出你的舒适区，学习一些你还不知道或不理解的东西，我向你保证，这只会让你成为一个更好的开发人员。