# Kotlin:类型别名和内联类

> 原文：<https://itnext.io/kotlin-type-aliases-and-inline-classes-ebe9d75f6715?source=collection_archive---------3----------------------->

![](img/f94489588d509306a2896c0ce4ea713f.png)

照片由[布伦丹·丘奇](https://unsplash.com/@bdchu614?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

T 毫无疑问，Kotlin 将长期存在，更何况谷歌刚刚在最近的 Google I/O 2019 上宣布 *Kotlin 现在是谷歌 Android 应用开发的首选语言。*

> 今天我们宣布了另一个重大进展:Android 开发将变得越来越优先。许多新的 Jetpack APIs 和特性将首先在 Kotlin 中提供。如果你正在开始一个新项目，你应该用 Kotlin 写它；用 Kotlin 编写的代码通常意味着更少的代码——需要键入、测试和维护的代码更少。

[](https://android-developers.googleblog.com/2019/05/google-io-2019-empowering-developers-to-build-experiences-on-Android-Play.html) [## Google I/O 2019:赋能开发者在 Android + Play 上构建最佳体验

### 很高兴谷歌 I/O 再次来到我们的后院，与世界各地的 Android 开发者建立联系。7200……

android-developers.googleblog.com](https://android-developers.googleblog.com/2019/05/google-io-2019-empowering-developers-to-build-experiences-on-Android-Play.html) 

Kotlin 是一种现代编程语言，所以今天我们将看到两个很酷的特性:**类型别名**和**内联类**。

# 键入别名

类型别名为您提供了一种为现有类型设置替代名称的好方法。

我们都遇到过这样的情况，某个类的名字太长或太复杂。使用类型别名，你可以引入一个不同的更短或更有意义的名字并使用新的名字。

让我们想象下面的场景，其中您设置了一对带有用户名和密码的字符串。

我已经创建了一个实例`Pair`,作为用户名和散列密码，然而这并没有给我太多的上下文，也不是很清楚。

类似这样的事情不会更好:

这为您的代码提供了更好的上下文和可读性。

**类型别名不引入新类型**。它们相当于相应的基础类型。

另一个可以使用类型别名的好例子是，当你需要在同一个地方使用两个同名但不同包的类时。发生这种情况时，您通常通过第二个类的完全限定类名来引用它。这种情况可以这样绕过:

类型别名也可以用来命名函数类型，也可以与函数类型参数名称一起使用。

# 内嵌类

有时，出于业务逻辑或代码可读性的考虑，有必要创建某种类型的包装器，但这会由于额外的堆分配而产生运行时开销。

为了避免这种情况，在 Kotlin 1.3 的**内嵌类**中引入了 **(** 实验⚠️feature).这些特殊的类使得这种改进成为可能，因为数据被内联到它的使用中，事实上对象实例化被跳过。

内联类必须有一个在主构造函数中初始化的**属性。**

内联类的声明

如前所述，当您使用内联类时，不会发生实际的实例化。在运行时，内联类的实例将使用这个属性来表示。请看下面的例子:

没有发生类 ***年龄*** & ***身高*** 的实际实例化，在运行时*年龄用户 1* 和*身高 1* 只包含 ***Int***

像常规类一样，它们也可以有属性和函数，并且可以从接口继承，但是有一些限制:

*   不允许使用 *init* 块；
*   属性不能有支持字段；
*   不能参与一个类的层次结构，这意味着它们不允许扩展其他类，它们必须是 ***final*** 。

如果你到了这里，你可能会问自己:

> 内联类和类型别名不是一样的吗？

我知道这种痛苦😅

它们看起来非常相似，因为两者似乎都引入了新的类型，并且在运行时都被表示为底层类型。

然而，它们是不同的

*   内联类引入了一个真正的新类型；
*   *类型别名*只为一个现有类型引入一个**替换名**(别名)。

由于它们仍处于试验阶段，Android Studio 会在你使用它们时显示一些警告。可以通过在 gradle 文件中定义显式编译器标志来禁用警告。

您可以在官方文档中阅读有关类型别名和内联类的更多信息:

[](https://kotlinlang.org/docs/reference/type-aliases.html) [## 类型别名(从 1.1 开始)- Kotlin 编程语言

### 类型别名不会引入新类型。它们相当于相应的基础类型。当您添加…

kotlinlang.org](https://kotlinlang.org/docs/reference/type-aliases.html) [](https://kotlinlang.org/docs/reference/inline-classes.html) [## 内联类——kot Lin 编程语言

### Kotlin 编译器更倾向于使用底层类型而不是包装器来产生最佳性能和优化的…

kotlinlang.org](https://kotlinlang.org/docs/reference/inline-classes.html)