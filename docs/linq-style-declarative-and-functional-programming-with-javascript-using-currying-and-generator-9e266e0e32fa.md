# LINQ 风格的声明式和函数式 JavaScript 编程，使用 currying 和生成器函数

> 原文：<https://itnext.io/linq-style-declarative-and-functional-programming-with-javascript-using-currying-and-generator-9e266e0e32fa?source=collection_archive---------0----------------------->

![](img/d631259d529125f0b3d04726ac293aab.png)

[JJ 英](https://unsplash.com/@jjying?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的“不锈钢链条倾斜移位镜头照片”

LINQ 是 C#最好的特性之一，它提供了一种优雅的方式来编写易于阅读和理解的声明式和函数式代码。

有了 ES6 的一些特性，比如生成器函数和 currying 等技术，很容易探索用 JavaScript 采用 LINQ 风格编程的各种可能性。请注意，使用来自`Array`对象的`map`、`reduce`、`filter`等内置方法，即使使用 ES5 JavaScript 也可以编写 LINQ 风格的代码。像 [lodash](https://goo.gl/868bN1) ，[understand](https://goo.gl/YHcEXz)这样的多个库提供了处理集合和数组的综合方法，它们的编写将性能作为一个关键标准。

我在这篇文章后面的目标是展示如何使用这些特性来编写更简洁和可读的代码。

> 查看 [dotless](https://github.com/bhosale-ajay/dotless) ，这是一个 LINQ 风格的 JavaScript 编程实验库。尝试“npm i dotless”

# Currying

> Currying 是一种将带有多个参数的函数的求值转化为一系列函数的求值的技术，每个函数只有一个参数。— [维基百科](https://goo.gl/x9EmgC)

让我们来看看实际情况

正如您在这里看到的，Currying 变成了一种方便的技术，可以用来编写可读性和表达性更强的函数。

# 发电机功能

> 生成器函数提供了一种编写定制迭代器的简单方法。使用`yield`关键字，程序流返回给调用者，所以请注意，使用生成器函数时，执行不是连续的。详情请参考 [Mozilla 文档](https://goo.gl/MMgwwC)。

让我们来看看它的运行，下面的代码摘自[这篇文章](https://goo.gl/Hk2Hsr)来自 [Eric Elliott](https://medium.com/u/c359511de780?source=post_page-----9e266e0e32fa--------------------------------) ，请参考它以了解更多关于生成器功能和其他 ES6 特性的细节。

正如您从输出中看到的，程序流在来自 *Fibonacci* 函数的`while`循环和消耗由 *Fibonacci* 函数返回的生成器的`for`循环之间切换。

让我们结合 currying 和生成器函数来编写一些实用函数。

让我们使用这些实用函数来编写一些代码

像`whichAreOdd`和`takeFour`这样的变量在某些情况下可能过于简化，但它们可以帮助重构复杂的代码，以清楚地表达其意图。Currying 和 generator 使得代码可读性更好，同时也使得集合/迭代器的使用更加容易。

现在人们可以得出结论，生成器函数是返回数组/集合的函数的替代函数，但它们是根本不同的。生成器函数(一种状态机)返回迭代器(惰性执行)而不是实际的数组，程序流在生成器函数和消费者之间切换。

生成器函数可以返回无限序列，而消费者只能决定从该序列中只取 *n* 个项目。

让我们将*斐波那契*函数与上述效用函数结合起来。

[参考此代码笔](https://goo.gl/nu9tyX)使用上述代码。我用这种风格来解决从[代码](https://goo.gl/DuF3ao)出现以来的编程难题，参考这里的[库。有关详细信息，请查看](https://goo.gl/rxSVsi) [LINQ.ts](https://goo.gl/Pe82jG) 文件，该文件定义了编写表达性代码所需的大多数扩展。

感谢您的阅读，希望您觉得有用。