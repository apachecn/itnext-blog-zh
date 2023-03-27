# 使用 CSS 加速 React 应用程序

> 原文：<https://itnext.io/using-css-to-speed-up-your-react-apps-a26470829472?source=collection_archive---------3----------------------->

![](img/640087250e640fab5d7cde047786e0e1.png)

虽然这个标题看起来很吸引人，但在这篇文章中，我将讨论如何使用 CSS 来替换一些 JavaScript 代码，以使 React 应用程序响应更快。在我开始之前，我想提一下，这些概念主要适用于大型和复杂的组件，它们很可能在较小的组件或应用中不会产生*明显的结果*。说完这些，我们开始吧:

# 隐藏而不是卸载

想象一下，你有两个或更多的标签，都有大量的内容。每当您在选项卡之间切换时，实现跳转行为的“反应”方式将是卸载您“切换自”的选项卡，并装载您“切换到”的选项卡。以 [Twitter 的个人资料标签](https://twitter.com/AggArvanitakis)为例，试着点击“关注者”和“追随”标签。第一次这样做可能需要一段时间，因为 Twitter 正在执行一个 HTTP 请求来获取它们，但是如果你一直在两者之间交替，你会发现在你点击和内容出现之间有一个延迟。如果你试图进入“Tweets”标签，事情会变得更糟。这个延迟看起来有点奇怪，因为内容已经被获取了，那么是什么原因造成的呢？

答案是 React 运行时。每次挂载一个新组件时，React 都必须创建虚拟 DOM(一个包含已挂载组件树中所有子组件的链表)，运行组件构造函数中的任何代码(或已弃用的`componentWillMount`)，然后将元素添加到 DOM 中。但与此同时，React 还必须卸载以前的组件，这意味着拆除虚拟 DOM，运行`componentWillUnmount`生命周期方法，然后从实际 DOM 中删除相应的元素。

这可能令人吃惊，但是卸载一个组件并不便宜。您可能认为这只是删除内容，但是 React 需要从虚拟 DOM 中的多个位置删除引用，并为即将卸载的组件中包含的每个子组件运行生命周期方法。那可能会很慢。在 React Fiber(最高为 React 15.x.x .)之前，卸载前一个组件和安装下一个组件是一个连续的过程。如今，为了节省时间，React 并行地点燃这两个，因为它们彼此完全独立。

这里的技巧是不要让 React 卸载或重新装载任何东西，而是利用 CSS 来隐藏和显示这些选项卡的内容。只是为了好玩，转到同一个 Twitter 页面，通过浏览器的 devtools 启用和禁用`display: none`规则来尝试切换选项卡的显示。你会注意到反馈几乎是即时的。这种方法的缺点是，如果你有很多包含大量内容的标签，DOM 会变得很大。大多数情况下这是可以的，因为浏览器处理隐藏元素的方式与处理可见元素的方式不同，但是你也必须在你的应用上测试它。此外，你不能利用 React 的生命周期挂钩来执行副作用，因为你不再挂载&卸载任何东西。为了防止这种情况，您必须将一个`hideContents`道具传递给组件(这将触发`display: none`，并使用`useEffect`或`componentDidUpdate`对其变化做出反应。

总结这一节，每当您有频繁安装和卸载的沉重组件时，尝试隐藏它们。这肯定会产生更好的性能结果，但是可能需要您修改组件中的额外代码。根据您的应用程序，响应性的好处可能会也可能不会掩盖这些代码更改所需的工作。

# 在 CSS 中制作动画

在 React 世界中，很容易使用 CSS 来制作 DOM 元素的“外观”动画(例如，React 组件的安装)，但很难制作它的“消失”动画(例如，组件的卸载)，因为节点会立即被删除，CSS 过渡和动画没有机会应用。通常人们通过使用像`react-spring`、`react-motion`或`react-css-transition-group`这样的动画库来解决这个问题。这些库中的大多数都克隆了 DOM 元素，并对其应用淡出样式，而原始元素已经被卸载了。“淡出”动画意味着 React 组件将不得不多次重新渲染，动画的每个“帧”一次。为了解决这个问题，库建议您将您想要制作动画的 React 组件包装在一个额外的`<div />`(通常来自`Animated`库)中，以确保您绕过 React 的需要来参与所有这些。

虽然这些技术已经足够了，但是它们创建了与你的代码运行在同一个线程上的*命令动画*；主要的那个。相反，用 CSS 编写的动画是*声明性的*，由浏览器的合成器线程处理。因此，CSS 动画不受主线程更昂贵的任务的影响，因此后者可以更快地响应用户输入，使整个应用程序感觉响应更快。不幸的是，不利的一面是 JavaScript 给了你更多的控制动画的能力，而且对于有高级效果的动画来说，比如弹跳，它仍然是一种方式。

如果您使用的是本文的第一个技巧(隐藏而不是卸载)，制作元素“消失”的动画突然变得很容易，因为 DOM 元素仍然存在，因此可以应用动画和过渡的 CSS 规则。一般来说，CSS 是简单过渡的理想选择，应该优先于 JavaScript，尤其是当应用程序在动画中感觉笨拙的时候。当然，可能有[很多其他的事情](/6-tips-for-better-react-performance-4329d12c126b)会导致你的[应用在动画中感觉笨拙](/what-to-do-when-your-react-app-feels-slow-3744c966ddf)，但是记住这也是一个促成因素。

# 移除 CSS-in-JS 运行时

大多数 CSS-in-JS 库允许您使用 JavaScript 编写 CSS 规则。像 [emotion](https://github.com/emotion-js/emotion) 这样的库将你的 JavaScript 代码翻译成散列的 CSS 类，然后自动应用于你的组件。为了让它们能够在属性改变时改变这些样式，它们还包含了一个运行时，帮助它们根据组件的当前属性来评估应该应用哪个类。这不仅增加了所发布代码的整体占用空间，还降低了它的速度(即使是很小的速度)，因为每次修改组件的属性时，您都有一个运行时脚本在主线程上运行。通过**不使用**CSS-in-JS 解决方案，您可以减少发布的总体 JS 代码，同时提高应用程序的响应能力。当然，这样做的缺点是潜在的更糟糕的开发体验，因为您将失去现有 CSS-in-JS 解决方案所具有的优势(主题化、变量等。).

我想强调的是，这是一个非常特殊的技巧，99%的情况下你都不应该考虑。我只是把它放在那里作为一个需要考虑的事情，因为你可能有一个属于这 1%的真正沉重的应用程序会从中受益。

# 结束语

这篇文章的目的是让你了解如何利用 CSS 来解决大规模 React 应用程序中常见的响应问题。没有一个解决问题的方法是完美的，总会有某种妥协。或许看完这个，你或许能换一种方式妥协。

感谢阅读:)

*👋 ***嗨，我是***[***Aggelos***](https://aggelosarvanitakis.me)***！如果你喜欢这个，可以考虑在 twitter 上关注我*** ***并与你的开发者朋友分享这个故事*😀***