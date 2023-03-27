# 从类型脚本(或流)调用 ReasonML，简单的方法

> 原文：<https://itnext.io/calling-reasonml-from-typescript-or-flow-the-easy-way-6372cbf4ead3?source=collection_archive---------7----------------------->

![](img/ba4bfed4f16c3f3568f0d5c1c85e4253.png)

# 一点背景知识

我最近开始研究一个组件库，它实现了一些设计系统规范。这将使创建更复杂的用户界面和网站功能变得更容易，在我正在开发的产品上保持页面外观、用户体验和品牌风格的一致性。

我们选择使用 React、styled-components 和 TypeScript 来构建它，这样每个组件都更容易抽象、开发和测试。我们还想处理多个 UI 主题(暗/亮)，为什么不呢，也许将来会支持其他主题，样式化组件有一个很棒的内置主题提供程序来完成这项工作。

那么，让我们看看如何使用这些技术实现一个简单的按钮:

当然，这只是对实际按钮组件的简化，但它让我们能够理解我们是如何开始构建组件的。

因此，根据当前的主题(亮/暗)，我们的按钮看起来像这样:

太好了。现在的问题是我们需要多种样式(带轮廓的按钮，带有自定义图标，不同的颜色，等等)，所以我们想要构建一个非常抽象的组件:

我们最终得到了这样的结果:

每个帮手看起来都像这样:

其他所有的助手也是如此(同样，我只是简化了上面例子中的事情)。但是正如你所看到的，这里我们实际上是针对一种特定的颜色进行模式匹配，然后我们提取一个“元组”的值，可以描述如下:`[TextColor, BackgroundColor]`。

如您所见，在 TypeScript 中实现这一点非常容易，但我们在其他编程语言中实现了一种广泛使用的模式，如 Elixir、Haskell…和 ReasonML！那么，如果我必须与一对夫妇进行模式匹配呢？如果我不得不进行更复杂的计算来生成正确的字符串呢？对于像我这样的 FP 爱好者来说，这可能是 TypeScript 的一个很好的替代品，为什么不呢？

因此，这里出现了使用 ReasonML 重构至少那些助手的想法，保持与 TypeScript 的类型互操作性。

# ReasonML 如何让它变得更简单

首先，我们需要定义我们的理性类型。让我们从`ButtonProps`类型开始(我们暂时不考虑`theme`):

如果我们试图实现`getColor`助手，我们最终会写出这样的代码:

用 ReasonML 编写这种函数感觉非常自然！模式匹配是函数式编程语言中广泛使用的特性。尽管如此，在下一个 EcmaScript 版本中实现它还没有稳定的规范(只有一个提议和一个巴别塔插件，[在这里了解更多](https://www.hackdoor.io/articles/BGJDaNkv/pattern-matching-proposal.html))。

# ReasonML 和 TypeScript 之间的互操作性

我们只是触及了你应该尝试理性的表面原因！(我不搞笑，我知道)但是现在我们写了我们的 ReasonML 函数，怎么从 TypeScript 调用呢？信不信由你，这比你想象的要容易得多！

首先，让我们为我们的`theme`写下类型:

现在让我们用类型注释重写我们的助手函数，使它们更容易阅读:

厉害！现在让我们向我们的 typescript 代码库添加两个新的依赖项:

第一个是需要通过 Bucklescript 编译器将 ReasonML 编译成 JavaScript 第二个用于生成 TypeScript(甚至流)兼容的类型。

让我们向`package.json`文件添加两个新脚本:

太好了，我们需要创建一个名为`bsconfig.json`的新文件(类似于 Node.js' `package.json`文件):

如您所见，您可以指定应该生成哪种类型。在这种情况下，我们将生成 TypeScript 兼容的类型。只是少了一样东西。我们需要向 BuckleScript 编译器指定应该分析哪些函数来生成类型，这非常容易做到:

我们可以通过在函数声明前添加`[@genType]`来实现！

现在我们已经准备好通过运行`yarn build:re`来构建我们的 ReasonML 函数。它将在与我们原来的`.re`文件相同的目录下生成两个新文件(假设我们称之为`helper.re` ): `helper.bs.js`和`helper.gen.tsx`。让我们看看他们看起来怎么样:

***helper . bs . js:***

如您所见，BuckleScript 生成的文件看起来与我们在第一段中编写的原始 TypeScript 解决方案极其相似！

***helper . gen . tsx:***

这有点复杂，并且利用了`bs-platform`库来工作。但是正如你所看到的，它公开的类型和我们在第一段中编码的方式完全一样！它还会自动添加 Eslint:disable 注释，这样就不会与您的 Eslint 配置冲突。

多棒啊。

你会用 ReasonML 和 TypeScript 构建什么？我打赌你会做一些伟大的事情！

[![](img/e05f00ed2cddfd2907284cb397168c3d.png)](https://github.com/sponsors/micheleriva)