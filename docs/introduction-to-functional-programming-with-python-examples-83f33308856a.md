# 使用 Python 示例介绍函数式编程

> 原文：<https://itnext.io/introduction-to-functional-programming-with-python-examples-83f33308856a?source=collection_archive---------4----------------------->

![](img/d8130fd86810a8e628a3956125bd169c.png)

[Duomly —在线编程课程](https://www.duomly.com/)

本文最初发表于:[https://www . blog . duomly . com/the-most-important-aspects-of-functional-programming-with-python-examples/](https://www.blog.duomly.com/the-most-important-aspects-of-functional-programming-with-python-examples/)

**函数式编程**是一个有趣的编程概念，最近得到了很多关注。本文介绍了函数式编程的一些最重要的方面，并提供了几个 Python 中的例子。

函数式编程是一种声明式编程范式，其中函数表示对象之间的关系，就像数学中一样。因此，函数不仅仅是普通的例程。

这种编程范式可以用多种语言实现。有几种函数式编程语言，如 Closure、Erlang 或 Haskel。除了其他范例之外，许多语言还支持函数式编程:C++、C#、F#、Java、Python、JavaScript 等。

在本文中，您将找到与函数式编程相关的几个重要原则和概念的解释:

*   纯函数，
*   匿名函数，
*   递归函数，
*   一流的功能，
*   不可变的数据类型。

# 纯函数

纯函数是这样的函数:

*   是幂等的—如果提供相同的参数，则返回相同的结果，
*   没有副作用。

如果一个函数使用一个来自更高作用域的对象或随机数，与文件通信等等，它可能是*不纯的*，因为它的结果不仅仅取决于它的参数。

一个函数在它的作用域之外修改对象，写入文件，打印到控制台等等，有副作用，也可能是不纯的。

纯函数通常不使用外部作用域的对象，从而避免共享状态。这可能会简化程序并帮助避免一些错误。

# 匿名函数

匿名(lambda)函数对于函数式编程结构来说非常方便。它们没有名字，通常是临时创建的，只有一个目的。

在 Python 中，使用 lambda 关键字创建一个匿名函数:

```
lambda x, y: x + y
```

上面的语句创建了接受两个参数并返回它们之和的函数。在下一个例子中，函数 f 和 g 做同样的事情:

```
>>> f = lambda x, y: x + y 
>>> def g(x, y): 
return x + y
```

# 递归函数

*递归函数*是在执行过程中调用自身的函数。例如，我们可以使用递归来寻找函数式的阶乘:

```
>>> def factorial_r(n): 
if n == 0:
return 1
return n * factorial_r(n — 1)
```

或者，我们可以用 while 或 for 循环解决同样的问题:

```
>>> def factorial_l(n):
if n == 0:
return 1
product = 1
for i in range(1, n+1):
product *= i
return product
```

# 一级函数

在函数式编程中，函数是第*级*对象，也称为*高阶*函数——数据类型与其他类型的处理方式相同。

函数(或者更准确地说，它们的指针或引用)可以作为参数传递给其他函数，也可以从其他函数返回。它们也可以用作程序内部的变量。

下面的代码演示了将内置函数 max 作为函数 f 的参数传递，并从 f 内部调用它。

```
>>> def f(function, *arguments):
return function(*arguments)>>> f(max, 1, 2, 4, 8, 16)
16
```

非常重要的函数式编程概念是:

*   映射，
*   过滤，
*   减少。

在 Python 中都是支持的。

*映射*使用内置的类别映射进行。它将一个函数(或方法，或任何可调用的函数)作为第一个参数，将一个 iterable(如列表或元组)作为第二个参数，并返回迭代器，其中包含对 iterable 项调用函数的结果:

```
>>> list(map(abs, [-2, -1, 0, 1, 2]))
[2, 1, 0, 1, 2]
```

在本例中，内置函数 abs 分别使用参数-2、-1、0、1 和 2 进行调用。我们可以通过列表理解获得相同的结果:

```
>>> [abs(item) for item in [-2, -1, 0, 1, 2]]
[2, 1, 0, 1, 2]
```

我们不必使用内置函数。可以提供自定义函数(或方法，或任何可调用的函数)。在这种情况下，Lambda 函数可能特别方便:

```
>>> list(map(lambda item: 2 * item, [-2, -1, 0, 1, 2]))
[-4, -2, 0, 2, 4]
```

上面的语句使用自定义(lambda)函数 lambda item: 2 * item 将列表中的每一项[-2，-1，0，1，2]乘以 2。当然，我们可以用领悟来达到同样的目的:

```
>>> [2 * item for item in [-2, -1, 0, 1, 2]]
[-4, -2, 0, 2, 4]
```

*过滤*使用内置的类别过滤器执行。它还将一个函数(或方法，或任何可调用的函数)作为第一个参数，将 iterable 作为第二个参数。它对 iterable 的项调用函数，并返回一个新的 iterable，其中包含函数返回 True 的项或任何被评估为 True 的项。例如:

```
>>> list(filter(lambda item: item >= 0, [-2, -1, 0, 1, 2]))
[0, 1, 2]
```

上面的语句返回[-2，-1，0，1，2]的非负项列表，如函数 lambda item: item >= 0 所定义。同样，使用理解可以获得相同的结果:

```
>>> [item for item in [-2, -1, 0, 1, 2] if item >= 0]
[0, 1, 2]
```

*减少*是通过 functools 模块的减少功能执行的。同样，它需要两个参数:一个函数和一个 iterable。它对 iterable 的前两项调用函数，然后对该操作的结果和第三项调用函数，依此类推。它返回一个值。例如，我们可以找到列表中所有项目的总和，如下所示:

```
>>> import functools
>>>
>>> functools.reduce(lambda x, y: x + y, [1, 2, 4, 8, 16])
31
```

这个例子只是为了说明。计算总和的首选方法是使用内置的减函数 sum:

```
>>> sum([1, 2, 4, 8, 16])
31
```

# 不可变数据类型

*不可变对象*是一个一旦创建就不能修改状态的对象。相反，*可变对象*允许改变其状态。

在函数式编程中，不可变对象通常是可取的。

例如，在 Python 中，列表是可变的，而元组是不可变的:

```
>>> a = [1, 2, 4, 8, 16] >>> a[0] = 32 # OK. You can modify lists. 
>>> a
[32, 2, 4, 8, 16]
>>> a = (1, 2, 4, 8, 16)
>>> a[0] = 32 # Wrong! You can’t modify tuples.
Traceback (most recent call last):
File “”, line 1, in 
TypeError: ‘tuple’ object does not support item assignment
```

我们可以通过向列表追加新元素来修改列表，但是当我们尝试用元组来做这件事时，它们并没有改变，而是创建了新的实例:

```
>>> # Lists (mutable) 
>>> a = [1, 2, 4] # a is a list 
>>> b = a # a and b refer to the same list object 
>>> id(a) == id(b) 
True 
>>> a += [8, 16] # a is modified and so is b — they refer to the same object 
>>> a 
[1, 2, 4, 8, 16] 
>>> b 
[1, 2, 4, 8, 16] 
>>> id(a) == id(b) 
True 
>>> 
>>> # Tuples (immutable) 
>>> a = (1, 2, 4) # a is a tuple 
>>> b = a # a and b refer to the same tuple object 
>>> id(a) == id(b) 
True 
>>> a += (8, 16) # new tuple is created and assigned to a; b is unchanged 
>>> a # a refers to the new object 
(1, 2, 4, 8, 16) 
>>> b # b refers to the old object 
(1, 2, 4) 
>>> id(a) == id(b) 
False
```

# 函数式编程的优势

基本的概念和原则——尤其是高阶函数、不可变数据和无副作用——意味着函数式程序的重要优势:

*   它们可能更容易理解、实现、测试和调试，
*   它们可能更短更简洁(比较上面计算阶乘的两个程序)，
*   它们可能不太容易出错，
*   当实现并行执行时，它们更容易使用。

函数式编程是一个值得学习的有价值的范式。除了上面列出的优点，它可能会给你一个解决编程问题的新视角。

![](img/615b08df31bdb641b1c4fd642e2007ec.png)

[Duomly —编程在线课程](https://www.duomly.com/)

感谢您的阅读。

内容由我们的队友米尔科提供。