# Dart 中 Const 和 Final 之间的差异

> 原文：<https://itnext.io/difference-between-const-and-final-in-dart-78c129d0c573?source=collection_archive---------0----------------------->

## 让我们找出 Dart 和 Flutter 中 const 和 final 关键字的区别

![](img/c4fd969d852816e39f5ae396e41ef908.png)

# Dart 中的 const 和 final 是一样的吗？

凭借我的 JavaScript/TypeScript 背景，我已经熟悉了`const`关键字。在 Java 中，你没有`const`，但等号是`final`。当我开始使用 Dart(用于 Flutter)时，我对这两个可用的关键字感到惊讶，对我来说，它们看起来完全一样。但当然，他们不是。

## 比较关键词

```
**const** String personConst = 'Jeroen';
**final** String personFinal = 'Jeroen';personConst = 'Bob'; // Not allowed
personFinal = 'Bob'; // Not allowed
```

在上面的代码中，我们创建了一个`const`和一个`final`变量，并将我的名字赋给这两个变量。您不能同时重新分配它们。

但是有什么区别呢？

## 最后的

*   带有`final`关键字的变量将在**运行时**被初始化，并且只能被赋值一次。
*   在类和函数中，你可以定义一个`final`变量。
*   对于 Flutter specific，当状态被更新时，build 方法中的所有内容都将被再次初始化。这包括所有带`final`的变量。

```
[@override](http://twitter.com/override)
Widget build(BuildContext context) {
  // your code
}
```

## 常数

*   带有`const`关键字的变量在**编译时**被初始化，并且在运行时已经被赋值。
*   不能在类内部定义`const`。但是你可以在函数中。
*   对于特定的 Flutter，当状态更新时，build 方法中的所有内容都不会被再次初始化。
*   `const`在运行期间不能更改。

## 何时使用哪个关键字？

两者的简单例子:

*   使用`final`:如果你不知道它在编译时的值是多少。例如，当您需要从 API 获取数据时，这种情况会在运行代码时发生。
*   使用`const`:如果你确定在运行你的代码时一个值不会被改变。例如，当您声明一个始终保持不变的句子时。

## 感谢您的阅读！如果你觉得这篇文章有用，请给一些掌声👏点击关注按钮，考虑阅读我的其他文章:

[](/building-your-first-reusable-widget-with-flutter-cadb54c3c253) [## 用 Flutter 构建第一个可重用的小部件

### 在本指南中，我将向您介绍如何使用 Flutter 和 Dart 构建一个可重用的小部件。

itnext.io](/building-your-first-reusable-widget-with-flutter-cadb54c3c253) [](https://levelup.gitconnected.com/how-to-use-push-notifications-with-flutter-23bed9e47e52) [## 如何在 Flutter 中使用推送通知

### 开始以最简单的方式在本地向您的用户推送通知

levelup.gitconnected.com](https://levelup.gitconnected.com/how-to-use-push-notifications-with-flutter-23bed9e47e52) [](https://levelup.gitconnected.com/ecmascript-vs-typescript-private-fields-640ae37aa162) [## EcmaScript 与 TypeScript —私有字段

### TypeScript 的 private 关键字和 EcmaScript/JavaScript 的#字符有什么区别

levelup.gitconnected.com](https://levelup.gitconnected.com/ecmascript-vs-typescript-private-fields-640ae37aa162)