# 一个新手玩 ScalaCheck 的方法

> 原文：<https://itnext.io/a-newbie-way-to-play-with-scalacheck-ac409a8205a2?source=collection_archive---------1----------------------->

## 第 2 部分:属性

![](img/8fdb4353bf40d2d8e177133d65353a82.png)

图片由[https://www.flickr.com/people/wicker-furniture/](https://www.flickr.com/people/wicker-furniture/)

这是关于使用 ScalaCheck 工具进行测试的系列文章的第二部分。[的第一部分](/a-newbie-way-to-play-with-scalacheck-333cbabfc00a)描述了**生成器，**为代码中的参数生成数据。而这一部分则深入探讨了**属性**。

**奇闻**

一个测试者走进酒吧。他点了一杯啤酒。然后他点了 0 杯啤酒。他点了一瓶啤酒。过了一段时间，他点了 999999999 瓶啤酒。

一个真正的顾客走进一家酒吧，问道:“厕所在哪里？”。

最后一个动作的结果是，一个酒吧着火了，烧到了地上。

这个故事的寓意是“正确测试你的代码”，正如我的导师一直强调的那样。事实上，大多数情况下，程序员会试图如上所述精确地测试他的代码。

**基于属性的测试简介**

" ScalaCheck 教你研究函数的属性."—我的 Scala 导师([文德森·费雷拉](https://medium.com/u/a43d48be8213?source=post_page-----ac409a8205a2--------------------------------))说。例如，它与您用 JUnit 编写的普通测试有什么不同？而一个**属性**到底是什么？

在 JUnit 中，您可能会这样检查一个求和函数:2 + 3 = 5。这是一个明显的例子。你可能会写一组这样不同的例子，检查“通常的嫌疑”(所谓的常见边缘情况):零，非常大的数字和-1。在作为具体示例编写的测试中，您的测试代码显示了您的应用程序在个别情况下是如何工作的。你的测试总是会产生一个非常详细的，但是不完整的代码评估。

在 ScalaCheck 中，你检查 a + b == b + a！在本例中，您将检查数学求和函数的行为。这是一种更复杂的测试方式。ScalaCheck 将自动生成参数来测试所有 *a* 和 *b* 的可能性，而不仅限于一些“常见的疑点”，正如你通常在具体的测试代码中看到的那样。数学函数的这种特定行为在 ScalaCheck 中被翻译成代码的抽象行为，或者一个**属性**。

**动手操作示例**

我有兴趣测试这个 ScalaCheck 系列的[第一部分](/a-newbie-way-to-play-with-scalacheck-333cbabfc00a)中的一个*计算器*类*T3。加法*选项*的行为是什么，或者在 ScalaCheck 语言中，两个数求和的**属性**是什么？*

为了简单起见，让我们从两个整数的相加开始。从一个数据类型为 *Int* 的例子开始比较容易，因为 *BigDecimal* 有它自己的特点。尽管如此，我将为这两者提出一个解决方案。

**一个简单的例子:用*Int*求和的性质**

首先，我用四种方法创建了一个名为 *SimpleCalculator* 的对象:加、减、乘、除。这是两个数字之间的常规数学运算，在本例中是两个整数之间的运算。

```
#code for *SimpleCalculator* object SimpleCalculator {

  def summate(a: Int, b: Int): Int = {
    a + b
    }

  def subtract(a: Int, b: Int): Int = {
    a - b
  }

  def multiply(a: Int, b: Int): Int = {
    a * b
  }

  def divide(a: Int, b: Int): Int = {
    a / b
  }

}
```

在 ScalaCheck 中创建属性最常见的方法是使用来自 *org.scalacheck.Prop* 类的 *Prop.forAll* 方法。此外，如果您想将几个属性组合在一起，可以使用*org . Scala check . properties*类来实现。下面的代码展示了如何使用继承来实现它(“扩展”选项)。

我想把重点放在所有通话内容的更多细节上。一般来说，如果方法包含的逻辑表达式对于所有元素都为真，则 *forAll* 方法返回真，否则返回假。在下面的例子中，泛型 *a* 和*b*——两个整数参数——被输入到用于所有调用的*中，该调用通过布尔逻辑检查求和方法的行为。ScalaCheck 测试的棘手部分恰恰在于定义这个布尔表达式，因为它必须模拟要测试的函数的行为。*

```
#code for Property “summation”import org.scalacheck.{Prop, Properties}
import org.scalacheck.Prop.forAllobject SimpleCalculatorProps extends Properties ("SimpleCalculator")  
{

  *property*("summate") = *forAll* { (a: Int, b: Int) =>
    val result = SimpleCalculator.*summate*(a, b)
    result == a + b && result == b + a
  }...}
```

正如你所看到的，我们并没有集中在 *a* 和 *b* 的具体例子上。它由 ScalaCheck [**发电机**](/a-newbie-way-to-play-with-scalacheck-333cbabfc00a) 为我们打理，是自动生产的。我们定义了一个布尔表达式，它反映了*求和*方法的一个特定属性，即功能对称性。如果所有生成的数字都符合布尔表达式，ScalaCheck 将批准您的代码。

```
#ScalaCheck testing output for *Int* arguments+ SimpleCalculator.summate: OK, passed 100 tests.
```

这是一个成功的 ScalaCheck 测试的典型输出。或者在这种情况下，成功输出 100 个测试。请注意，所有测试都有随机输入参数，并且是并行执行的。如果你的代码不能处理大量同时发生的事情(并发访问)，那么 ScalaCheck 会立刻让你的代码失败。默认的测试次数是 100，但是你可以根据你的需求来调整。

**一个简单的例子:用*BigDecimal*求和的失败性质**

如果在前面例子的代码中将数据类型从 *Int* 更改为 *BigDecimal* ，那么 ScalaCheck 将使您的程序失败。

```
#ScalaCheck testing output for *BigDecimal* arguments! SimpleCalculator.summate: Falsified after 1 passed tests.
> ARG_0: -0.00001131560419259331
> ARG_0_ORIGINAL: -0.3707897181828976
> ARG_1: 0.00001108784593808157497528613518274946
> ARG_1_ORIGINAL: 6702191619228002457
```

这个 ScalaCheck 输出显示，一个使用 *BigDecimal* 作为参数的代码在一个实际上已经通过的测试之后被篡改。在输出中，您可以看到传递给所有调用的*的初始或原始(ARG_0_ORIGINAL)参数，以及没有传递布尔表达式的 *a* (ARG_0)和 *b* (ARG_1)的简化参数。*

测试用例简化是 ScalaCheck 的一个特性，它可以发现代码中的 bug。碰巧的是，ScalaCheck 缩小了代码失败行为的范围，并打印出导致代码失败的最简单的参数集。通常，当您检查原始输入和简化的参数时，代码中的弱点就变得很明显了。在 *BigDecimal* 的情况下，首先想到的是“为什么点后的位数不同？”嗯，可能是 *BigDecimal* 的舍入不一致？*

**进阶示例:与*BigDecimal*相加的性质**

回到*计算器*类的例子，在[第一部分](/a-newbie-way-to-play-with-scalacheck-333cbabfc00a)中有描述。它得到两个参数，即 *a: Int* 和 *b: BigDecimal* 并有一个*计算器*方法，带有四个*选项*(加、减、除、乘)。加法*选项*是两个数的基本求和。为了测试这个选项，我们可以使用与上面演示的相同的布尔逻辑 a + b == b + a。然而，我们需要应用一个 *round* 方法来处理 *BigDecimal* 参数的精度。

```
# code for property “ADDITION”package caskApiimport caskApi.Operation.*ADDITION*import org.scalacheck.Properties
import org.scalacheck.Prop.forAll
import java.math.MathContextobject CalculatorProps extends Properties ("Calculator") {

  *property*("ADDITION") = *forAll* { (a: Int, b: BigDecimal) =>
    val result = Calculator(a, b).calculator(*ADDITION*).round(new MathContext(4))
    result == (a + b).round(new MathContext(4))  && result == (b + a).round(new MathContext(4))
  }...}
```

使用指定的 BigDecimal 精度快速运行加法属性测试将通过 ScalaCheck 自动生成的所有测试。

```
#ScalaCheck test output+ Calculator.ADDITION: OK, passed 100 tests.
```

最后，尝试猜测其他数学运算的 ScalaCheck 解决方案，即减法、乘法和除法，并在我对该项目的[报告](https://gitlab.com/ullien/flask_in_scala)中检查您的猜测。警告:组织是一个真正的傻瓜！

**总结**

ScalaCheck 是一个基于属性的测试工具。简单地说，ScalaCheck 通过生成随机数据来测试代码块的抽象行为，从而检查数百甚至数千个具体测试。它将尽最大努力去证伪这种行为，即**地产**。尽管如此，如果所有这些具体的测试都将通过，那么 ScalaCheck 将认为这个属性是有效的，您可以认为您的应用程序已经过很好的测试。

*如果你遇到 BigDecimal 的问题，那么首先检查你的项目使用的 Scala 版本。您可能遇到过版本回归( [lchoran](https://github.com/scala/bug/issues/11590) 非常清楚地总结了这一点)。