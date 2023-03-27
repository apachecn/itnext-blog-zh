# 编写更好的反应，组成多个功能 hoc，高阶组件

> 原文：<https://itnext.io/write-better-react-compose-multiple-functional-hocs-higher-order-components-442a11bd2e86?source=collection_archive---------0----------------------->

![](img/ae78d14e7f227862cf0f9143d12d9def.png)

在[之前的一篇文章](/write-better-javascript-function-composition-with-pipe-and-compose-93cc39ab16ee)中，我写了关于使用管道和组合链接函数的**的概念。今天，我想通过提供一些场景来扩展这个主题，在这些场景中，我发现**函数组合**在前端开发人员的日常生活中变得非常方便，他们使用 React **以更具功能性的方式应用多个高阶组件！****

# 什么是高阶函数

在深入研究高阶组件之前，您应该熟悉**高阶函数**的含义，我们可以将其描述为**一个至少执行以下**之一的函数:

*   **将一个或多个函数作为参数** s
*   **返回一个函数作为其结果**

让我们以一个你可能已经熟悉的标准 ES 高阶函数为例:***array . prototype . map***，它将一个函数作为参数，用作回调，并将其应用于数组的每个元素。一个简单的提醒:

我们现在可以编写一个定制的高阶函数:

很明显，这是一个非常简单的例子，但是高阶函数有很多应用，这种方法的好处是**你可以重用提供不同运算函数的 HoF，减少代码重复，有利于**[](https://en.wikipedia.org/wiki/Single-responsibility_principle)****的单一责任原则。****

# **带 React 的高阶组件**

**高阶组件**与高阶函数**非常相似，下面是 React 文档中的定义:**“具体来说，高阶组件是一个接受一个组件并返回一个新组件的函数。”**。**

**这里有一个简单的例子非常有用，让我们首先定义一个标准组件，稍后我们将把它包装成一个特设组件:**

**假设您希望这个组件是用某种信息增强的****，在这个非常简单的例子中，我们传递一个自定义属性，一个静态用户，在实际应用程序中，您希望以某种方式获取它:******

****现在，我们可以用新创建的 HoC 来包装应用程序组件:****

******应用程序中由“withUser”特设包装的每个组件都将拥有 currentUser 属性**。如果我们有一个非常复杂的逻辑，这可能是一个非常好的模式来**避免代码重复**。你可以在 **Klarna 知识库**中看到很多这方面的真实例子:****

****[https://github.com/klarna/higher-Order-components](https://github.com/klarna/higher-Order-components)****

# ****构成多个 hoc****

****如果我们希望**一个组件被多个 hoc**包装会怎样？好了，这里我们有 compose at the rescue(它们在我的[上一篇文章](/write-better-javascript-function-composition-with-pipe-and-compose-93cc39ab16ee)中有深入的解释)。让我们创建另一个简单的特设:****

****现在我们可以将我们的两个 HoC 封装在一起([我们可以使用 Ramda 函数 compose，而不是创建我们的自定义函数](https://ramdajs.com/docs/#compose)****

****我创建了一个代码沙箱，所以你可以玩代码:****

# ****概述****

****高阶组件对于**抽象逻辑**真的很有用，比如你的大部分页面会有相同的布局，也许它们共享相同的元素；**它们易于处理**，**它们使代码更具可读性**和**它们不会改变原始组件，这意味着它们是纯函数**。****

****如果你已经来了，谢谢你的阅读，❤****

****在这篇文章中，我们用非常简单的组件解释了一些复杂的概念，并分享了一个你可能会觉得有用的模式。
*一些参考资料深入到本文的主要话题:*****

*   ****[https://tommmyy . github . io/ramda-react-redux-patterns/pages/react-ramda . html # high-order-component-hoc](https://tommmyy.github.io/ramda-react-redux-patterns/pages/react-ramda.html#high-order-component-hoc)****
*   ****[https://it . react js . org/docs/higher-order-components . html #:~:text = A % 20 higher % 2d order % 20 component % 20(HOC，and % 20 returns % 20a % 20 new % 20 component](https://it.reactjs.org/docs/higher-order-components.html#:%7E:text=A%20higher%2Dorder%20component%20(HOC,and%20returns%20a%20new%20component)。****
*   ****[https://github.com/klarna/higher-Order-components](https://github.com/klarna/higher-Order-components)****