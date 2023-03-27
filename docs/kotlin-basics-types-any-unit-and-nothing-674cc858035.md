# 科特林基础:类型。任何、单位和无

> 原文：<https://itnext.io/kotlin-basics-types-any-unit-and-nothing-674cc858035?source=collection_archive---------0----------------------->

![](img/8bcba11a931036f129067b33eebede2c.png)

由[杯先生/杨奇煜·巴勒](https://unsplash.com/@iammrcup?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

Java 用`Object`来表示类层次的根，用`void`来表示缺少类型。我们将在这里看到 Kotlin 如何表示这些类型，以及它如何改进它们。

# 任何的

`Object`是 Java 中类层次的根，每个类都有`Object`作为超类。在 Kotlin 中，`Any`类型代表所有**不可空**类型的超类型。

它与 Java 的`Object`有两点不同:

*   在 Java 中，原语类型不是层次结构的类型，你需要隐式地将它们装箱，而在 Kotlin `Any`中是所有类型的超类型。
*   `Any`不能保存`null`值，如果你需要`null`成为你的变量的一部分，你可以使用类型`Any?`

> `java.lang.Object`方法`toString, equals`和`hasCode`是从`Any`继承而来的，而要使用`wait`和`notify`则需要将变量强制转换为`Object`才能使用它们。

# 单位

在 Java 中，如果我们希望函数不返回任何东西，我们使用`void`，`Unit`在 Kotlin 中是等价的。

`Unit`对阵 Java 的`void`的主要特点是:

*   `Unit`是一个类型，因此可以用作*类型参数。*
*   只存在一个这种类型的值。
*   它是隐式返回的。不需要一个`return`声明。

# 没有任何东西

Java 中不存在这种类型。当函数永远不会正常终止，因此返回值没有意义时，就使用它。

在分析这种类型的代码时，知道函数永远不会终止是非常有用的。

这种函数的一个例子是测试系统中的`fail`函数，或者游戏引擎中的主循环。

# 结论

我们已经看到了 Java 基本类型`Object`和`void`在 Kotlin `Any`和`Unit`中是如何对应的，以及它们的好处，比如更好地用作*类型参数。我们还了解了 Kotlin 的特定`Nothing`类型如何在我们需要实现永不返回的函数的场景中帮助我们。*