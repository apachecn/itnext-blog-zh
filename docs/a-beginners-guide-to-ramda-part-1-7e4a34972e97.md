# Ramda 初学者指南(第一部分)——涂抹和构图

> 原文：<https://itnext.io/a-beginners-guide-to-ramda-part-1-7e4a34972e97?source=collection_archive---------3----------------------->

![](img/b77d0f877ea7c795fd4f62d45fc44f54.png)

照片由 [Kazden Cattapan](https://unsplash.com/@kazdenc?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

Ramda 对我来说是一个看起来很酷的库，但是它的大小让我再三考虑把它添加到我们的前端。自从我最近转向后端开发(node.js)后，我开始使用 Ramda，并慢慢意识到它的强大。这份指南基本上是我在过去几个月里学到的，另外还附带了一些例子。

# 自动 currying

Ramda 中的所有函数都是自动生成的。这意味着您可以用比它需要的更少的参数调用该函数，并且它将返回一个部分应用的函数。函数将不断返回用它们的最终参数调用过的函数，此时它们将计算结果。

```
R.add(1, 2)
> 3R.add(1)(2) 
> 3const increment = R.add(1)
> fnincrement(2)
> 3
```

还有一个名为`R.curry`的功能，它可以让你将任何正常的功能变成自动定制的功能。

```
const add = (a, b) => a + badd(1, 2)
> 3add(1)(2)
> Uncaught TypeError: add(...) is not a functionconst curriedAdd = R.curry(add)curriedAdd(1, 2)
> 3curriedAdd(1)(2)
> 3
```

## 为什么奉承很重要？

Currying 允许我们做两件在函数式编程中很重要的事情。即专业化和复合化。

**专门化**来源于部分应用定制功能的能力。我们可以将这些部分应用的函数存储为变量，以便以后再次使用。在下面的例子中，我们可以重用`addTitle`函数的部分应用版本。

```
const bev = {name: 'Bev', gender: 'female'}
const rich = {name: 'Rich', gender: 'male'}const addTitle = (title, name) => `${title}. ${name}`
const curriedAddTitle = R.curry(addTitle)
const addMs = curriedAddTitle('Ms')
const addMr = curriedAddTitle('Mr')addMs(bev.name)
> 'Ms. Bev'addMr(rich.name)
> 'Mr. Rich'
```

然后我们可以继续为`addMrs`、`addMiss`、`addDr`等等创建函数。通过这样做，我们可以创建一个定义良好的接口，而不是每次都用两个参数调用`addTitle`。

关于 currying 的另一件令人敬畏的事情是它允许我们更容易地将**功能组合在一起。在下一节中，我们将了解如何做到这一点。**

# 功能组成

什么是函数构成？根据维基百科:

> 在数学中，函数合成是将一个函数逐点应用于另一个函数的结果以产生第三个函数

在编程中，我们可以将其概括为将任意数量的函数组合在一起，产生一个新函数。这里有一个简单的例子:

```
const increment = (x) => x + 1
const double = (x) => x * 2const doublePlusOne = (x) => increment(double(x))
doublePlusOne(10)
> 21
```

这里我们创建了一个新函数，它返回调用传递给它的值的结果。我们将这些功能组合在一起。这看起来很好，但是当我们想要将两个以上的函数组合在一起时会发生什么呢？

```
const square = (x) => x * x
const halve = (x) => x / 2const calculateThings = (x) => halve(square(increment(double(x))))
calculateThings(10)
> 220.5
```

现在它开始显得笨重。不仅括号堆积在一边，而且我们需要从右向左读取函数调用。

## 管道救援

Ramda 给了我们一个很棒的函数叫做`pipe`。它将任意数量的函数作为其参数，并返回一个函数，该函数依次调用这些函数以产生结果。让我们使用管道函数重写`calculateThings`。

```
const calculateThings = R.pipe(double, increment, square, halve)calculateThings(10) // still 220.5
```

`pipe`去掉了所有的括号，也让我们以更易于阅读的从左到右的顺序来指定我们的函数。

除了第一个函数，所有传递给`pipe`的函数都必须是[一元函数](https://en.wikipedia.org/wiki/Unary_function)。因为每个函数被调用时前面都有函数的结果，所以每个函数只接收一个参数。

另外值得一提的是，还有一个类似于`pipe`的函数叫做`compose`，只不过它会以相反的顺序调用这些函数。你喜欢哪个由你决定，但是`pipe`对我来说更直观。

还记得我们讨论过 currying 如何使组合功能变得更容易吗？因为 Ramdas 的函数是自动执行的，所以我们可以将部分应用的调用传递到管道中。

```
// With partially applied functions
const mathPipe = R.pipe(
  R.multiply(4),
  R.add(2),
  R.divide(2)
)mathPipe(10)
> 21
```

Ramdas 的数学函数`add`、`multiply`和`divide`都有两个参数，但是我们只传入第一个参数，第二个值将通过管道传入。

部分应用的函数也可以用在任何需要一元函数的地方。例如，我们可以将部分应用的函数作为参数传递给 Ramdas 的`map`函数。

```
const people = ['James', 'Hadley', 'Terry', 'Trev', 'Szab']
const addTitle = R.curry((title, name) => `${title}. ${name}`)R.map(addTitle('Mr'), people);
> ['Mr. James', 'Mr. Hadley', 'Mr. Terry', 'Mr. Trev', 'Mr. Szab']R.map(addTitle('Dr'), people);
> ['Dr.  James', 'Dr.  Hadley', 'Dr. Terry', 'Dr. Trev', 'Dr. Szab']
```

# 总结

希望你已经从这篇文章中得到了一些有用的东西，并且像我喜欢写这篇文章一样喜欢阅读它。本系列的第二部分将是关于 Ramda 的镜头功能。请继续关注，它很快就会出现。