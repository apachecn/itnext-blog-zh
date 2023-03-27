# 具有 Typescript 函数重载的多个接口定义。

> 原文：<https://itnext.io/multiple-interface-definitions-with-typescript-function-overloading-86943143d538?source=collection_archive---------3----------------------->

![](img/2ddc0c50181dd7cc77c8604af0d72d3b.png)

照片由 [Raphael Koh](https://unsplash.com/@dreamevile?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

> TS2769:没有与此调用匹配的重载。

熟悉吗？🤔 💭没错。我们将回到这个**打字稿**的错误描述。但首先，我们需要一些奇特的背景。

所以最近我发布了[**vue-use-variant**](https://github.com/lukasborawski/vue-use-variant)NPM 包。这是一个简单的处理程序，通过为你的样式块定义变量来帮助你处理长的 CSS 类定义。为 [vue.js](https://vuejs.org) 和 [Tilewind](https://tailwindcss.com) 而写。它可以与 Bootstrap 和 React 或任何其他类似工具结合使用。

但是，正如它是为 Vue 编写的一样，我希望使用新的组合 API [**ref 方法**](https://v3.vuejs.org/guide/composition-api-introduction.html#reactive-variables-with-ref) 来创建一个具有*值*属性的特殊对象，以保持反应数据引用。同时希望保持使用 CSS 样式变体的常规对象的能力。

这些物体将会/可能看起来像那样。

```
const variantRef = { value: { button: 'font-sm rounded' } }const variant = { button: 'font-sm rounded' }
```

这意味着如果我有一个函数，我想把一个参数传递给它，我需要为它定义两个不同的接口。这样不太好，效率也不高。看一看。

那么如何应对呢？如何通知用户/开发者可以使用两个不同的对象体，并且仍然可以正确使用？这里出现了 **Typescript 函数重载**特性。下面是一些[官方描述](https://www.typescriptlang.org/docs/handbook/2/functions.html#overload-signatures-and-the-implementation-signature)。

> TypeScript 提供了函数重载的概念。可以有多个同名但参数类型和返回类型不同的函数。但是，参数的数量应该是相同的。

那很好，但是如何使用它呢？🤔这是完整的例子。

你在这里看到的是三个函数定义。一个用于常规对象，一个用于特定于 Vue 的对象，一个重载它们。这样，当我们使用我们的函数时，我们将能够传递两个不同的物体作为参数，但是我们仍然可以很好的使用我们的类型。当我们使用例如**数字**作为对象值时，就像这样…

```
const variantRef = { value: { button: 44 } }const variant = { button: 44 }
```

…我们会得到一个错误。

```
TS2769: No overload matches this call.
```

**字符串或布尔值是允许的**，你不会在这里看到任何错误。

你可以在这里查看[的完整类型和用法。所以每次你看到这个错误，你可以确定你有一些选项来定义你的函数数据，但是它不适合定义的重载接口。很花哨，很厉害。✨](https://github.com/lukasborawski/vue-use-variant/tree/develop/src)

感谢阅读。干杯，卢卡斯。也许可以考虑 [**请我喝咖啡**](https://www.buymeacoffee.com/lukas.borawski) 。