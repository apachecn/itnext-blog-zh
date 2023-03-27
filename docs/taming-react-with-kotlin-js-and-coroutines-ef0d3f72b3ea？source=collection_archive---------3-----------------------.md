# 驯服与 Kotlin-JS 和协同程序反应

> 原文：<https://itnext.io/taming-react-with-kotlin-js-and-coroutines-ef0d3f72b3ea?source=collection_archive---------3----------------------->

![](img/42f56389b2d56a34b52ec243482cc229.png)

驯服野马——乔治·凯特林([https://www.canvasreplicas.com/](https://www.canvasreplicas.com/))

这篇短文讲的是一个小技巧，它最终让我能够在我的 [Kotlin-JS](https://kotlinlang.org/docs/tutorials/javascript/kotlin-to-javascript/kotlin-to-javascript.html) 应用程序中充分使用 [React-js](https://reactjs.org/) 组件。

React 是浏览器 JavaScript 应用程序最流行的平台之一。它的主要思想是允许用户用 javascript 构建声明性 UI，并自动传播状态变化，而不需要显式回调和重绘触发器。React 还以其 reconcile 算法而闻名，该算法允许计算内存中 DOM 树的变化，而无需实际将它们绘制到屏幕上。

坦率地说，我不确定 Kotlin-JS 是否需要 React。在纯 Kotlin 中创建一个反应式应用程序是非常容易的，并且`kotlinx.html`库允许轻松地创建内存中的 DOM 结构，而不需要渲染它们。关于这一点的一些机制和考虑写在关于 ploty.kt 的文章中。尽管如此，我们并不是生活在真空中，当我们需要创建一个合理的 UI 时，我们需要依赖现有的库。我的经验表明，很难找到一个编写良好的纯 JS 组件库。大多数旧版本都是用旧版本的 JavaScript 编写的，很难阅读，甚至更难与静态类型的 Kotlin-JS 进行互操作。新的主要是为 React、Vue 或 Agular 框架编写的。React 相当流行，有大量现成的组件，所以从它开始是有意义的。另外，在 [Kotlin 动手材料](https://play.kotlinlang.org/hands-on/Building%20Web%20Applications%20with%20React%20and%20Kotlin%20JS/)中有一个很棒的 React 教程。

尽管如此，我还是遇到了一个相当严重的问题，这个问题几乎使 React 无法用于我的目的。问题是 React 真的不喜欢它内部的变化受到外部的影响。对于不与任何外部组件通信的单页 React 应用程序来说，这是很好的，但是如果我有一些非 React 应用程序，并且只想使用单独的基于 React 的组件，这就很方便了。当然，我可以让 React 感染我的应用程序，并将我自己的 UI 片段包装在 React 组件中，但我最初的想法是在我更大的应用程序中使用 React 组件，而不是反过来。那我该怎么帮呢？

我一直在阅读的 React 教程建议将侦听器函数作为组件属性的一部分来传递。在 Kotlin 中，这种函数的签名看起来像`typealias Listener = State.()->Unit`或类似的东西。然后，只要我想通知外部侦听器状态的变化，就可以从组件中触发该函数。但这不是我想做的，我想通过外部触发来改变内部状态，而且不是一次，而是多次。当然，我可以创建一个允许订阅的特殊对象，将它传递给组件，然后强制组件订阅更改，但是这个解决方案看起来相当麻烦。但是在非同步实体之间传递消息是 kotlin 协程的完美案例。让我们看看我们能做些什么。

要点(用依赖项和 HTML 文件的构建脚本部分补充)展示了如何完成。想法如下:我们创建一个协程[通道](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-channel/)，并将其作为参数传递给 React 组件。然后，我们所要做的就是在组件内部创建一个消费者循环，我们可以从外部完全控制我们的状态变化。我们可以发送单独的消息，定期发送或手动关闭频道。我们甚至可以使用流到通道转换来组织来自外部源的数据管道。我使用 rendezvous channel，这样一旦我们的页面无法处理呈现，新的状态变化就会停止传播(这是协程提供的自动反压机制)。如果我们生成了太多的更改，那么发送方将会挂起并等待组件评估之前的请求。还可以使用由通道和流 API 提供的不同变体。例如，当我们有几个变化等待渲染时，我们可以只绘制最新的状态变化。