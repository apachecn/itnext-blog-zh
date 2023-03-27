# 反应上下文和反应跟踪

> 原文：<https://itnext.io/react-context-and-react-tracked-296a6b86f890?source=collection_archive---------2----------------------->

![](img/fc54920ec3f9a23e781caefd8dd310ef.png)

由 [Unsplash](https://unsplash.com/s/photos/headphones?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的 [C D-X](https://unsplash.com/@cdx2?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄的照片

这篇文章是我的[上一篇文章](/react-hooks-with-context-as-a-state-management-solution-526d1c13a07d)的后续文章，上一篇文章解释了我如何使用 React 钩子和上下文来管理应用程序状态。本文将关注上下文的负面影响，以及[反应跟踪](https://www.npmjs.com/package/react-tracked)如何帮助提高应用程序性能。

**上下文的缺点** 当应用程序状态更新时，它可能是一个具有许多键和值的大对象，上下文提供者(监听上下文变化)下的所有嵌套 DOM 元素都将更新，这是使用上下文进行状态管理的主要缺点，它没有优化以支持状态的微小变化。在我的第一篇文章中，我解释了你可以通过把你的状态分成几个独立的状态或者不听上下文来克服这个缺点，但是这并不能解决问题。

**React-Tracked to rescue** libs 像 [redux](https://redux.js.org/) 和 [mobx](https://mobx.js.org/README.html) 甚至新的[反冲](https://recoiljs.org/)都是为了解决这个问题而优化的，它们可以告诉你的应用程序状态中哪个键得到了更新，并只为订阅这些变量(键)的组件触发 DOM 树更新。如果你从零开始一个项目，我强烈建议尝试其中之一，但如果你有一些代码约束，或者想尝试一些精益的东西，那么我想分享我如何改变我的代码以更好地优化，但首先让我们谈谈反应跟踪。

什么是 React-Tracked
React-Track 是一个由 [Daishi Kato](https://github.com/dai-shi) 维护的[库](https://www.npmjs.com/package/react-tracked)，就像几年前创建 redux 的 [Dan Abramov](https://github.com/gaearon) 一样，Daishi 创建了这个库来更好地管理 React 状态。我不知道你是否知道，但 redux 的第一个版本并没有那么优化，只有在后来的版本中，当他们添加了选择器时，redux 才解决了手头的问题。

这个库背后的主要思想是用一个由代理组成的对象替换我们用作上下文状态的普通对象，使用[代理](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)使我们的增强状态能够跟踪状态中发生了什么变化以及哪些订阅者对该变化感兴趣。所以这个库基本上是跟踪你的每一个组件，它监听状态的哪一部分，当状态的那一部分被更新时，只有订阅的组件会被重新呈现。这些代理保存在一个 [WeakMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap) 中，它支持持有对象，允许对不使用的键进行垃圾收集，如果你手头有键，还支持更好的搜索 O(1)。

我最喜欢这个库的一点是，它的用法非常优雅，不需要对代码进行重大修改。

**使用 React-Tracked 的缺点是什么？**
在使用这个库以及与作者交谈时，我发现了两件不太完美的事情，所以为了表明我没有偏见，我想分享一下我的发现。
1。在调试过程中，当您将鼠标悬停在状态参数上时，您将看到一个代理对象，然后您需要单击 target 键来查看实际值，这没什么大不了的，只是不太方便。如果它不是一个对象，而是一个像字符串或数字这样的原语，你会看到实际的值。
2。React-Tracked 知道根据您在组件中引用的键来监听状态的某个部分，但是如果我们在助手函数中而不是在主 react hook 函数中使用这些键，它不知道触发订阅。要解决这个问题，您可以在 react 钩子函数中创建变量，该函数实际订阅状态参数，并使用您创建的变量。

让我们看看我是如何实现的
如果你没有阅读我的第一篇文章，我建议你在阅读本节之前阅读，因为我的代码是基于第一个代码设计的。

如您所见，只对上下文文件做了更改，对我们如何使用被跟踪的上下文做了微小的更改，同时在性能上有了很大的提高。

**结论** 将 React-Tracked 添加到您的代码中，同时使用上下文管理状态相对容易，并且通过防止不需要的组件呈现来提供更好的性能。这主要是通过用代理对象包装状态来实现的。