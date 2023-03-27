# 从 Scala 到 Go 的过渡

> 原文：<https://itnext.io/the-transition-from-scala-to-go-27603e373986?source=collection_archive---------2----------------------->

![](img/fad7768bc4149c00189d5ce6f35e91cb.png)

一年前，我作为一名软件工程师加入了谷歌。我的团队使用 Go 编写了一些代码来保持谷歌云的运行。一年后，我已经变得对这门语言非常有生产力，我已经获得了 Go 的可读性，而且无论如何，我都不怀念用 Scala 编写软件。

至少对我来说，Scala 的一些特性使它成为一门有吸引力的语言。其中一个特征是可以使用高阶函数。

高阶函数是简单的函数，可以接收函数作为参数，并可以返回另一个函数作为结果。让我们看一些例子。

```
def addS(n: Int) = n.toString() + "S"
val xs = List(1,2,3,4,5)
val someNumsWithS = xs.map(addS)
```

注意`map`接收一个函数作为参数，在我们的例子中，接收的函数是`addS`。

另一个例子是函数返回另一个函数，而不是一个具体的值。

```
def aFunc = addS 
val fn = aFunc
fn(5)
 --> 5S
```

在这里，`aFunc`是一个返回另一个函数`addS`的函数。

Scala 中的这两个函数构造非常重要，但是 Go 以相似的方式呈现了它们，让我们看看如何呈现。

```
func AddS(n int) string {
  return string(n) + "S"
}func map(xs []int, fn func(int)string) []string {
  result := []string{}
  for _, n := range xs {
    result = append(result, fn(n))
  }

  return result
}xs := []int{1,2,3,4,5}
someNumsWithS := map(xs, AddS)
```

在这个例子中，我们实现了与 Scala 中相同的功能。我们有一个函数`map`，它接收另一个函数`fn`作为参数。我们可以将任何函数传递给`map`，只要它具有相同的签名。

和 Scala 一样，我们可以创建返回其他函数的函数。

```
func addOne(n int) func(int)int {
  return func() { return n + 1 }
}fn := AddOne(5)
x := fn()
 --> 6func sum() func(int, int)int {
  return func(a, b int) int { return a + b}
}var s func(int, int)int = sum() // same as s := sum()
s(3, 4)
 --> 7
```

注意`s`是如何被类型注释来显示类型的，但是这完全没有必要，因为它可以写成`s := sum()`。

上一个例子让我想到了另一个让从 Scala 过渡变得非常容易的地方，那就是类型推断。

Go 有很好的类型推断。它与 Scala 最大的不同在于，在 Go 中，我们需要指定函数的返回类型，就像我们在预览示例中看到的那样。`AddS`返回`string`，`sum()`返回`func(int, int)int`以此类推。在 Scala 中，大多数时候，我们可以跳过函数的返回类型，它仍然可以从上下文中正确地推断出来，编译器总是会检查类型的正确性。然而，即使类型推理系统允许我们省略一些字符，我还是会用 Scala 来写，因为这有助于更好地理解函数的作用。

Go 是一种简单的语言，与 Scala 相比更容易学习。例如，在 Scala 中有很多很多方法可以迭代集合:

```
val values = ...values.forEach
values.map
values.filter
values.first
values.foldLeft
values.collect
values.takeWhile
...
```

在围棋中，有一种更简单、更明确的方法:

```
values := ...for idx, value := range values {
  // do what you need to do
}
```

集合中的所有东西都是用最基本的工具解决的，一个`for`循环。在 Scala 中，有许多奇特的实用程序，它们都以非常高效的方式解决特定的问题。然而，这意味着我们必须知道所有这些不同的实现，它们的类型签名，它们的用例，等等…

在 Go 中，我们有`for`，一个简单、优雅的解决方案，永远有效。

从 Scala 过渡到 Go 还有一点，那就是并发程序是如何编写的。这本身就需要一个帖子，不过幸运的是，我们已经把这个内容写好了，参见 [**基本并行计算中的 Go**](https://levelup.gitconnected.com/basic-parallel-computing-in-go-fda50894241c?source=friends_link&sk=55bd268422c3321581ac6382d0fa41ae) 和[**更多关于 Go 通道、并行性和并发性**](https://levelup.gitconnected.com/as-a-follow-up-of-basic-parallel-computing-in-go-i-wanted-to-build-a-more-complex-example-where-13978bf84608?source=friends_link&sk=33ee3b3489b7762f8cfcfa255128b22c)

总的来说，我认为 Scala 是一种非常强大的编程语言，拥有丰富的工具，在函数式编程世界中占有很好的地位。然而，它带来了额外的复杂性，固执己见，并有一个非常分裂的用户群。它的库通常实现得很好，但是很难使用，特别是对于那些缺乏函数式编程经验的人。

另一方面，Go 是一种简单、快速、干净的小型语言，非常容易学习。通过 Go，我们构建了世界上最大的(可能是最大的)基础设施系统之一，为 Google Cloud 提供动力。Go 允许成千上万的工程师一起工作，似乎证明了简单性大有裨益。