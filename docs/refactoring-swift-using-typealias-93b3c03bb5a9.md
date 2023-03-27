# Swift 中的重构:类型别名化

> 原文：<https://itnext.io/refactoring-swift-using-typealias-93b3c03bb5a9?source=collection_archive---------2----------------------->

## 除了将 JSON 定义为[String: Any]之外，Typealiases 还可以有更多用途

![](img/3e7b00ef308212d1e2efb87206b3dfde.png)

`typealias Swift = Apodidae.` 翠鸟无血缘关系。

## 经典用途

您可能以前见过或使用过这些类型别名之一:

```
*// To give commonly used structures a name*
typealias JSON = [String: Any]*// To combine two or more protocols into a protocol composition*
typealias UserAPI = GetUserInterface & UpdateUserInterface*// To make a closure type more readable*
typealias Callback = (() -> ())
```

本文不讨论这些情况——有一种更简单的方法可以改进代码！

## 单位清晰度

前几天，我编写了一个包含一些商业逻辑的函数来计算一顿饭中的卡路里数。结果是这样的:

```
func calculate(carbs: Float, protein: Float, fat: Float) -> Float
```

现在，*我*知道这个函数取什么单位，返回什么单位，但是其他人会知道吗？

考虑一下`TimeInterval`。它实际上是一个`Double`，但是每当我们看到一个`TimeInterval`时，我们立即知道该值是以秒为单位的预期值。

```
*// UIView.h*
func animate(withDuration duration: TimeInterval, animations: @escaping () -> Swift.Void)
```

我的`calculate`功能甚至可以在添加文档注释之前得到改进:

```
*/// A weight representation in Grams* typealias Grams = Float*/// Kilo-calories*
typealias kCal = Floatfunc calculate(carbs: Grams, protein: Grams, fat: Grams) -> kCal
```

看起来那些论点根本不是用国际单位制的！

## 传统 API

类型别名还有许多其他类似的用途。在我们当前的项目中，我们在与以整数形式返回布尔值的 API 集成时使用了它们(是的，🙄).这是我们从其中一个端点获得的结果的模型:

```
struct BadgeInfo: Decodable {

    let lunch_flg: Int *// 0 or 1*
    let dinner_flg: Int *// 0 or 1*
    let snack_flg: Int *// 0 or 1*
}
```

这并不理想，但是我们可以确保需要与这些模型交互的其他开发人员意识到这三个属性只能是`0`或`1`，而不需要他们检查模型文件的注释:

```
*/// An integer that can only be zero or one (support for* 🙄 *API)*
typealias BoolInt = Intstruct BadgeInfo: Decodable {

    let lunch_flg: BoolInt
    let dinner_flg: BoolInt
    let snack_flg: BoolInt
}
```

你现在就可以对你的项目这样做——它只改变了一个文件，而且(在我看来)它使你的代码更具可读性。

## 会员人数

我最喜欢的 typealias 的另一个用途是跟踪标识符:

```
typealias MemberIdentifier = String
typealias PassengerIdentifier = String
```

如果您可以在需要成员编号时使用这些标识符，那么任何阅读您的代码的人都会立即清楚哪个参数需要哪个标识符。

```
*// Before - are we linking with the membership id or passenger id?*
func link(user: String, to flight: Flight)*// After - aha! (of course, the argument name could be tidied too..)*
func link(user: PassengerIdentifier, to flight: Flight)
```

我们在这个场景中发现的另一个受欢迎的副作用是:我们的静态模拟 API 返回整数形式的成员数，但是当我们第一次被允许访问真正的模拟时，我们发现成员数实际上是字符串。将`MemberIdentifier`的定义从`Int`更改为`String`，然后清除一些编译错误，这比对每个引用都进行更改要快得多！

## 还有更多

类型别名是一种不断出现的语言特性。作为这篇文章的结尾，下面是一些关于下次编写函数时如何使用类型化的灵感:

```
*// It's clear that the angle is in Radians and not Degrees*
func pointOnCircumference(given angle: Radians) -> CGPoint*// This can be used for a number that must be from 0.0 to 1.0*
typealias Percentage = Float*// Sometimes UIKit APIs still use NSAttributedStringKeys as strings
// But don't make your attribute dictionaries [String: Any]!*
let someLegacyAttributes: [NSAttributedStringKey.RawValue: Any]*// For when you are passing around a dictionary of 
// ["English": "en", "Japanese": "ja"...]*
typealias Language = String
typealias LanguageCode = String
let languageMap: [Language: LanguageCode] = ...
```