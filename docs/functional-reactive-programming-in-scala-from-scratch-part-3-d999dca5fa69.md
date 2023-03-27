# Scala 中的函数式反应式编程(第 3 部分)

> 原文：<https://itnext.io/functional-reactive-programming-in-scala-from-scratch-part-3-d999dca5fa69?source=collection_archive---------3----------------------->

在这一系列的文章中，我们想从头开始为 Scala 中的函数式反应式编程开发一个小框架。如果你还没有阅读该系列的前两部分，请务必在此查看:[第 1 部分](/functional-reactive-programming-in-scala-from-scratch-part-1-9f9db0c47478)、[第 2 部分](/functional-reactive-programming-in-scala-from-scratch-part-2-3d1559a11629)。

![](img/143288294ac5769c04fb3608374d3dc5.png)

[bert sz](https://unsplash.com/@bertsz?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

*注:你可以在* [*这个 GitHub 资源库*](https://github.com/timo-stoettner/frp-scala) *中找到有完整代码的笔记本。*

在上一篇文章中，我们设法编写了一个小框架的工作示例，使我们能够做我们想要做的事情:用一个合并器跟踪几个银行账户的余额。合并器利用了我们的小框架来处理所有更新合并余额的复杂性。

然而，这个 API 还没有我们想要的那么优雅。我们需要将观察到的信号显式地传递给合并器，这样它就知道需要观察哪些其他信号的变化。这是重复且容易出错的。提醒一下，这是我们的函数`consolidated`的样子:

`Signal`的第一个参数构成了计算我们的`Signal`值的函数。第二个参数指的是我们新定义的`Signal`所依赖的其他`Signal`,因此必须观察它们的变化。

这是我们想要的样子:

换句话说，我们想摆脱第二个论点。因此，新的`Signal`必须自己弄清楚，它依赖于其他哪些`Signal`。

为了解决这个问题，我们将利用`DynamicVariable` s。我将不得不绕一点弯子来解释它们，所以跟我说吧。(如果你知道如何使用`DynamicVariable` s 或者它的 Java 对应物`ThreadLocal`，可以随意跳过这一部分。)

# 使用 DynamicVariable 跟踪依赖性

Scala 的`DynamicVariable`是动态范围模式的一个实现(因此也被称为“动态”变量)。不必深入，计算机程序的作用域指的是程序中可以使用标识符(如变量名)的区域。作用域有两种方式:*词法(或静态)作用域*和*动态作用域*。

标准是词法范围，这是您最可能习惯的。这里，范围取决于源代码中的位置。例如，如果在类方法中使用变量名，编译器首先在方法本身中查找该名称，然后在类级别上查找。

让我们看一个小例子来说明这一点:

这里大概没有什么大的惊喜。在`print_class_value`中没有定义`value`，所以使用在类级别定义的那个。在`print_method_value`中，`value`是在方法本身中定义的，它比类级别上的定义具有更高的优先级。

另一方面，动态范围取决于*执行上下文*。这里，当您在类方法中使用变量名时，编译器将首先在方法本身中查找该名称，然后在调用方法的方法中查找*(依此类推)。有一些编程语言实现了动态范围，但是大多数(比如 scala)默认使用静态范围。*

动态范围会使理解变量值的来源变得非常困难。然而，如果你想根据谁调用了一个方法来跟踪一个值，动态范围是一个好办法。

如果您回想一下我们的银行帐户示例，这就是我们在这里所追求的。如果我们知道一个`Signal`被另一个`Signal`调用，我们可以将它添加到一组观察器中，并在发生变化时更新这些观察器。幸运的是，我们实现了这样一种模式。

在回到银行账户之前，让我们先看一个简单的例子来说明`DynamicVariable`是如何工作的:

这里需要注意一些事情:

*   在定义 DynamicVariable 时，我们需要传递一个默认值。这里我们将默认值设置为 1。除非我们明确指定该值为其他值，否则将使用该值。
*   我们可以通过调用`value`来访问动态变量的值。
*   DynamicVariable 有一个方法`whithValue`，它接受一个新值和一些任意表达式作为参数。如果表达式访问`DynamicVariable`的当前值，它将接收已经传递给`withValue`的值。*变量的值因此依赖于执行上下文*(与动态范围一样)。
*   如果我们在没有事先调用`withValue`的情况下访问变量值，我们将简单地获得默认值。

你也可以像这样对`withValue`进行嵌套调用:

关于动态变量的几点进一步说明:

*   那些来自 Java 的人可能已经注意到`DynamicVariable`和 Java 的`ThreadLocal`非常相似。当查看[源代码](https://github.com/scala/scala/blob/v2.12.8/src/library/scala/util/DynamicVariable.scala#L1)时，你可以看到`DynamicVariable`实际上使用了 Java 的`InheritableThreadLocal`
*   正如其 Java 对应物的名字所暗示的那样，`DynamicVariable`非常适合在线程中使用。然而，在这个例子中，这与我们无关
*   如果你想了解更多关于静态和动态范围的知识，请查看维基百科关于范围的页面。

# 回到我们的银行账户例子

那么，在我们的`BankAccount`示例中，我们如何使用动态变量来跟踪被依赖的`Signal`呢？我们将使用它们通过使用`withValue`方法跟踪当前调用者来跟踪谁在调用`Signal`。每当一个`Signal`调用另一个`Signal`时，我们将把它添加到一组观察者中。如果我们程序的另一部分调用了`Signal`，我们的 DynamicVariable 将返回到它的默认值，没有任何效果。

让我们提醒自己到目前为止我们的代码是什么样子的:

在这里，我只是对上一篇文章末尾的代码做了一些小的改动:我去掉了`Signal`构造函数的第二个参数`observed`以及对它的所有引用。一旦我们引入一个`DynamicVariable`来跟踪呼叫者，我们就不再需要它了。注意，我保留了变量`observers`。

为了合并`DynamicVariable`，我们需要一个默认值。因为我们的调用者将是类型`Signal`，缺省值也必须是类型`Signal`:

这只是一个没有实现的`Nothing`类型的伪信号。我们还覆盖了方法`computeValue`,以免陷入无限循环。

接下来，我们将在伴随对象`Signal`中定义一个动态变量，因此完整的伴随对象如下所示:

我们将`caller`定义为一个`DynamicVariable`，其值属于类型`Signal`，默认值为`NoSignal`。(类型声明`Signal[_]`引用了一个存在类型。这基本上意味着我们不关心类型。)

注意，我们在伴随对象中定义了`caller`，而不是在类本身中，因为我们只希望它的一个实例跟踪当前调用的`Signal`，而不是每个`Signal`一个实例。

我们将在两个地方使用`caller`。首先，我们将在每次调用`apply`时将当前调用者添加到观察者集合中:

现在，每次我们访问`Signal`的值时，它的调用者都会被添加到观察者列表中。如果我们在没有指定`caller`的值的情况下从类外的某个地方调用，添加的观察者将只是`NoSignal`，这没有任何效果。如果我们从另一个`Signal`接收到`Signal`的值，我们需要确保我们指定了当前调用者。我们将在`computeValue`这样做:

就是这样！综上所述，我们的代码如下所示:

我只做了一个小改动:我没有直接为`curVal`和`curExpr`设置值，而是在之后调用`update`。这确保了两件事:首先，我们不必以这种方式重复`computeValue`中的代码。第二，当初始化`NoSignal`时，构造函数不试图评估任何东西，因为它的方法`computeValue`被一个空方法覆盖了。

来测试一下吧！

看起来不错！我希望你同意这个解决方案比我们在[上一篇文章](/functional-reactive-programming-in-scala-from-scratch-part-2-3d1559a11629)中提出的方案更好。都是拜`DynamicVariabe` s 所赐。

如果你真的想使用这段代码，你还需要注意两件小事。我们去看看。

# 更多的改进

我还想写两个小的改进:

*   目前，一旦观察者被添加到观察者集合中，它们将永远不会被移除。当给一个`Signal`分配一个新的表达式时，它可能会停止依赖它之前依赖的其他`Signal`。我们应该考虑到这一点。
*   目前可以定义相互依赖的`Signal`。如果处理不当，这种循环依赖将导致无限循环。例如，看看下面的代码，其中`b`是`a`的函数，而`a`是`b`的函数:

幸运的是，这两个问题都很容易解决。为了在必要时更新观察器，我们只需要在每次执行`computeValue`时重置观察器:

因为所有仍然依赖于`Signal`的观察点都将调用`Signal`的 apply 方法，所以它们将在被移除后不久被重新添加到观察点集合中。所有不再依赖于`Signal`的观察者都不会。

其次，为了防止循环的`Signal`定义，我们只需要在 apply 方法中添加一点点`assert`语句:

如果调用的`Signal`观察到它检索其值的`Signal`，assert 语句将抛出一个错误。

就是这样！感谢您耐心听我解释如何从头开始在 Scala 中实现一个函数式反应式编程的小框架。我希望你从中学到了一些东西。

# 从这里去哪里

*   Scala 的实际实现。React 还有很多我在这里没有提到的特性。如果你有兴趣深入了解，可以看看这篇文章[用 Scala 反对观察者模式。反应](https://infoscience.epfl.ch/record/176887/files/DeprecatingObservers2012.pdf)或者 GitHub 上的[源代码。](https://github.com/ingoem/scala-react)
*   Scala 中还有一些其他的函数式反应式编程框架。我知道 [scala.rx](https://github.com/lihaoyi/scala.rx) 和 [reactify](https://index.scala-lang.org/outr/reactify/reactify/3.0.3?target=_2.12) 。(至少第一个也大量基于 scala.react)
*   一个相关的、更受欢迎的框架是 [RxScala](http://reactivex.io/rxscala/) 。然而，RxScala 更多的是一种反应式编程的实现，而不是*功能性*反应式编程。(如果你对这些区别感到困惑，可以看看我的文章[揭开函数式反应式编程的神秘面纱](/demystifying-functional-reactive-programming-67767dbe520b))。不过，根据您的需求，RxScala 可能是解决您的问题的一个非常好的、有据可查的解决方案。

我计划在以后的文章中研究其中的一些库，所以一定要回来看看。此外，我计划研究如何将这一系列文章中的代码翻译成 Python。

如有意见，请留言！