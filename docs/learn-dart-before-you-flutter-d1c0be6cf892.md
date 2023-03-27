# 扑之前先学会飞镖

> 原文：<https://itnext.io/learn-dart-before-you-flutter-d1c0be6cf892?source=collection_archive---------2----------------------->

![](img/eb7f1a4b4649a336c8e31a6de8c1c13e.png)

[欧文·赫斯里](https://unsplash.com/photos/Hpxsdu4Bdm8?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/flutter?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的原始照片

鉴于以移动领域为目标的技术创新的“淘金热”，开发移动应用从未如此简单，这要归功于跨平台解决方案，如 Flutter、React Native、NativeScript、PhoneGap 等。

特别是 Flutter，它通过一种富于表现力的风格成功地吸引了开发社区的注意力，这种风格使得为移动应用程序构建 ui 成为一种乐趣。它结合了现代开发经验中熟悉的某些概念，如反应式编程和小部件合成，使用 Dart 平台作为其主要操作基础。

## 更喜欢视频？

# 那么，什么是*镖*？

Dart 是 Google 开发的面向对象编程语言，旨在帮助开发人员构建现代 web 应用程序。它涵盖了客户端，服务器和现在的移动与 Flutter。它配备了一系列工具[，包括虚拟机、核心库和包管理库，为您的下一个项目提供足够的弹药。](https://www.dartlang.org/guides/get-started)

虽然 Flutter 越来越受欢迎，但它很容易掩盖 Dart 平台的美丽和它所提供的东西，与 Flutter 无关。

在本文中，我们将研究如何编写 Dart 程序，探索它的一些语言特性。这将有望让你有一个概览，帮助你在开发下一个应用程序时看到 Dart 的光芒。

→ [**继续阅读我的博客**](https://creativebracket.com/learn-dart-before-you-flutter/)

# 第一步

下面是一个灾难援助反应队计划的例子:

这演示了带有属性和方法的订单的蓝图。顶层函数是引导 Dart 应用程序的地方。

在这里，我希望你会开始看到其他面向对象语言的语法看起来非常熟悉，如果你用 Java、C#甚至 JavaScript( *ES2015 及以上*)编程，你会感觉很舒服。

也就是说，Dart 附带了一些语言特性，这些特性肯定会让您作为开发人员更有效率。看看我们之前课程的改进:

这里没有做太多，但是我们现在有了一个简化的构造函数，省去了重新分配传递给类属性的参数的重复工作。

我们还通过从我们的`getInfo`方法中移除加号(+)来使用 ***相邻的字符串*** 。哦，在类属性前加上下划线(_)会使它们成为私有的。这是常规的，省去了键入 `private` *关键字的需要。*

好，让我们看看我们能做到什么程度。我们的构造函数参数是以位置方式定义的。 ***这意味着它们是必需的*** 。Dart 允许我们定义可选的参数，有两种味道: ***可选位置*** *和* ***可选命名*** 。这些本质上允许我们在定义和使用它们的方式上有更好的灵活性:

```
Order(this._id, this._reference, **[date]**); // optional positional
Order(this._id, this._reference, **{date}**); // optional named
```

以及它们的用法:

```
new Order(1, 'ref1', **new DateTime.now()**); // optional positional
new Order(2, 'ref2', **date:** new DateTime.now()); // optional named
```

当然，我们可以将它映射到我们的内部属性，尽管对于可选的命名参数，该属性必须是公共的:

```
Order(this._id, this._reference, **[this.date]**); // opt. positional
Order(this._id, this._reference, **{this.date}**); // optional named
```

我希望到目前为止这是可以预见的。现在让我们在我们的属性上实施一些类型信息:

```
int _id;
String _reference;
DateTime _date;
```

OO 语言的另一个共同特征是能够多次声明构造函数，区别在于传递给它的参数数量。相反，Dart 为您提供了 ***命名构造函数*** ，这实际上允许您向构造函数添加一个名称空间，使您不必担心参数数量:

```
Order(this._id, this._reference, [this.date]); // normal
**Order.withDiscount**(this._id, this._reference, this.code); // named 
```

我们将这样实例化它:`new **Order.withDiscount(...)**`。让我们看看目前为止的重构:

我想介绍的最后一个特性是 ***方法级联*** 。这些允许您为 getters 和 setters 使用链式模式，这种模式在 JavaScript 库 jQuery 中很流行。为了演示这一点，我们将添加另一个名为`bookings`的公共属性:

```
int _id;
String _reference;
DateTime _date;
String code; **List<String> bookings;**
```

当我们实例化对象时，我们会:

```
Order order1 = new Order.withDiscount(1, 'ref1', 'WEEKENDFTW1')
   **..code** = 'WEEKENDFTW1' **..bookings** = ['booking1', 'booking2', 'booking3'];
```

双倍周期(..)显示了每次总是返回实例的级联。链接与 setters 一起工作，当您调用方法时也是如此。

这里有一个完整的解决方案:

# 结论

一旦您开始构建您的 Flutter 应用程序，学习这些 Dart 概念将带您走得更远，因此它不会让您感到惊讶。

**首先，为什么不尝试修改打印输出以包含其他订单信息？**通过[在线 Dart 编辑器](https://dartpad.dartlang.org/bd7899cddbcf3726d8c01d8ee2d9ff5b)可以轻松获得完整的示例。

# 进一步阅读

*   [镖语之旅](https://www.dartlang.org/guides/language/language-tour)
*   [试用 DartPad 在线编辑器](https://dartpad.dartlang.org/)
*   [**全栈 web 开发用飞镖**](http://bit.ly/2P2N1jC)

# 分享是关怀🤗

如果你喜欢读这篇文章，请通过各种社交渠道分享。此外，检查并 [**订阅我的 YouTube 频道**](https://youtube.com/c/CreativeBracket) (也点击铃铛图标)上的 Dart 视频。

[**订阅我的电子邮件简讯**](http://eepurl.com/gipQBX) 下载我的免费 35 页**Dart 入门电子书**并在新内容发布时得到通知。

喜欢、分享和 [**关注我**](https://twitter.com/creativ_bracket) 😍有关 Dart 的更多内容。