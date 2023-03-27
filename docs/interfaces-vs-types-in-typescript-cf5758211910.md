# TypeScript 中的接口与类型

> 原文：<https://itnext.io/interfaces-vs-types-in-typescript-cf5758211910?source=collection_archive---------1----------------------->

## 有什么区别

![](img/50954fd598f1ad853c9013d3d3287498.png)

照片由 [David Pisnoy](https://unsplash.com/@davidpisnoy?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/paint?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

我在做 Typescript 项目的时候，有的开发者用`interfaces`，有的开发者用`types`。尽管如此，我还是尝试使用接口和类型。第一次我很困惑，因为他们很像。那么，我应该使用什么接口或类型呢？

如果你很困惑或者不确定区别在哪里，我们来谈谈不同情况下的最佳实践。

# 类型与接口

在开始之前，我们先来看看什么是接口和类型。

接口[在开发阶段是一个有用的工具，它就像你的应用程序中的一个契约，或者类要遵循的语法。该接口也被称为鸭打印，或子类型。如果你有兴趣，你可以在我的另一篇文章](https://www.typescriptlang.org/docs/handbook/2/objects.html)[中阅读更多关于何时以及如何在 TypeScript](https://blog.logrocket.com/when-how-use-interfaces-classes-typescript/) 中使用接口和类的内容。

[类型或类型别名](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases)类似于一个接口，为任何类型创建一个新名称。注意，它没有定义新的类型。

接下来，我将使用 [Typescript Playground](https://www.typescriptlang.org/play/index.html#) 进行编码，而不是创建一个新项目。让我们比较一下如何使用不同类型的类型和接口。

## 基元

原语只能由`types`使用。

```
type AppName = string
type AppType = string
type AppSize = number
```

*我们不能通过* `*interfaces*`创建它们。所以这是他们的第一个区别*。*

## 联合

[Unions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types) 是一个可以存储多种类型值的变量。我们可以通过`types`建立工会。

```
// AppName - string or undefined
type AppName = string | undefined
type AppType = 'mobile' | 'web' | 'desktop'
type AppSize = number | bigint
```

同样，我们可以通过两种类型创建一个新的 union `Employee`:

```
type Developer = {
  devName: string
}type Tester = {
  testerName: string
}type Employee = Developer | Tester
```

让我们定义一个结合了两个接口的类型:

```
interface Developer {
  devName: string
}interface Tester {
  testerName: string
}type Employee = Developer | Tester
```

该接口不支持联合。

## 元组类型

[元组](https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types)是一对具有不同类型值的元素。

```
type App = [name: string, type: 'mobile' | 'web' | 'desktop']
```

我们可以使用`interface`声明一个元组。但是相比`type`就不清楚了。我更喜欢使用类型。

```
interface AppInfo {
  value: [name: string, type: 'mobile' | 'web' | 'desktop']
}
```

## 功能

我们可以使用`types`和`interfaces`来定义函数:

```
type runApp = () => void
```

然而，使用`interface`似乎有些棘手:

```
interface runApp {
  (): void
}
```

## 声明合并

[声明合并](https://www.typescriptlang.org/docs/handbook/declaration-merging.html)是指 TypeScript 将两个或两个以上同名的接口编译合并成一个声明。让我们定义两个`App`接口:

```
interface App {
  name: string
}interface App {
  type: 'mobile' | 'web' | 'desktop'
}// TypeScript will automatically merge both App declarations into one
const app: App = {
  name: 'facebook',
  type: 'mobile'
}
```

*如果我们试图声明两个* `*App*` *via 类型，就会抛出错误。*因为和`types`不配合。

```
// error - Duplicate identifier 'App'.
type App {
  name: string
}// error - Duplicate identifier 'App'.
type App {
  type: 'mobile' | 'web' | 'desktop'
}
```

## 交集

[交集](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types)通过`&`关键字将多个类型合并为一个类型。

```
interface Developer {
  devName: string
}interface Tester {
  testerName: string
}type Employee = Developer & Tester
```

*但是它没有将多种类型合并到一个接口中。*

## 遗产

[继承](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming))是从父代继承属性和方法的能力。这只能使用`interfaces`来实现。T *他的不可能跟* `*types*` *。*

```
interface Developer {
  devName: string
}interface Tester {
  testerName: string
}interface Employee extends Developer, Tester {}class User implements Employee {
  devName = "Dev Name"
  testerName = "Tester Name"
}
```

# 结论

在本文中，我们讨论了类型和接口之间的主要区别。它们非常相似，但是您应该使用什么取决于您的用例。

感谢阅读——我希望这篇文章对你有用。编码快乐！

# 资源

[](https://ultimatecourses.com/blog/typescript-interfaces-vs-types) [## 类型脚本接口与类型-终极课程

### 许多开发人员在选择 TypeScript 接口还是类型时感到困惑。这可能是因为他们…

ultimatecourses.com](https://ultimatecourses.com/blog/typescript-interfaces-vs-types) [](https://blog.logrocket.com/types-vs-interfaces-in-typescript/) [## TypeScript - LogRocket 博客中的类型与接口

### 在 JavaScript 中进行静态类型检查的想法真的很棒，而且 TypeScript 的采用正在增长…

blog.logrocket.com](https://blog.logrocket.com/types-vs-interfaces-in-typescript/) [](https://dev.to/saadsharfuddin/type-vs-interface-in-typescript-35i6) [## TypeScript 中的类型与接口

### 当我第一次开始使用 TypeScript 时，我很快发现自己在质疑类型和接口在…

开发到](https://dev.to/saadsharfuddin/type-vs-interface-in-typescript-35i6)