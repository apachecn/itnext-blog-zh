# Vue 异步组件:道具和测试

> 原文：<https://itnext.io/vue-async-components-props-and-testing-cbcb1e5f89a5?source=collection_archive---------0----------------------->

如何测试并将属性传递给异步加载(async)组件的加载和错误对应项。

![](img/4be3b353ba862b3a40c22e985324d0c2.png)

空巢。版权所有诺拉布朗。

Vue 异步组件的基本用法已经被很好地记录下来了。然而，我找不到两个基本问题的答案:

*   如何(手动)测试加载和错误组件？
*   如何将道具传递给加载和错误组件，使其更具重用性？

我会在这个帖子里分享可能的答案。

# Vue 异步组件刷新程序

正如我提到的，Vue 中异步组件的[基础在文档中有很好的介绍，其他地方](https://vuejs.org/v2/guide/components-dynamic-async.html#Async-Components)的[也有介绍。但是提醒一下:当你有一个大的(有点)组件，或者 a)你的应用程序中不需要，或者 b)在给定的会话中可能永远不需要时，异步组件是有用的。将异步组件与 Webpack(处理代码分割)一起使用意味着您的组件可以在需要的时候被延迟加载。](https://vueschool.io/articles/vuejs-tutorials/async-vuejs-components/)

注意，异步*路由*与异步*组件*相关但不同。尽管异步组件使异步路由成为可能，但每种路由的用例略有不同。一个`/profile`页面是一个由 Vue 路由器处理的[异步*路由*的良好候选:它有自己的 url，并且可能永远不会在用户会话中被访问。带有表单的弹出窗口是异步组件的一个很好的候选:它没有自己的路径，当应用程序第一次加载时也不需要。](https://router.vuejs.org/guide/advanced/lazy-loading.html)

(补充说明:一个路由只接受一个组件，所以你通常不能像异步组件那样提供加载和错误组件，但是[这篇文章](https://medium.com/bauer-kirch/how-to-make-vue-router-play-nice-with-loading-states-3f2ff6bfd633)基于[这篇文章](https://github.com/chrisvfritz/vue-enterprise-boilerplate/blob/890438c3b898d6ed921fd2d0e3b84f2fe7162d89/src/router/routes.js#L99)展示了如何解决这个限制。)

# 如何测试加载和错误组件？

你已经在你的应用中找到了一个非常适合异步加载的组件，并相应地编写代码:

异步组件的基本用法，包括加载和错误组件

将在加载期间或出现故障时显示的组件是静态导入的。然后，为了延迟加载我们的子组件，我们传递一个返回对象的函数，该对象具有您在上面的代码中看到的属性。

太好了。当我们查看应用程序时，眼睛盯着网络选项卡，当我们单击按钮时，我们会看到子组件的 JavaScript 按需加载，并且子组件会显示出来。但是如何测试加载和错误组件呢？

我们可以像这样使用一个小助手函数:

使用一个助手函数来延迟组件的加载

上面的代码中有两点发生了变化。首先，我们添加了助手函数`asyncComponentTester`。它需要两个参数:

*   **importPromise** ，由`[import()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)` [函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)返回的承诺
*   **延迟**，一个延迟，它是在包装承诺解析和来自`import()`的原始承诺被返回之前的时间量，以毫秒为单位。

第二，我们不直接称呼`import('@/components/ChildComponent)`，而是称呼我们的新助手:`asyncComponentTester(import('@/components/ChildComponent), 3000)`。

通过操作`delay`、`timeout`和`latency`，我们可以显示加载和错误组件。例如，使用以下值:

*   `delay` : 200
*   `timeout` : 1500
*   `latency` : 2000 年

200 毫秒后，将显示 LoadingState 组件。(请记住，`delay`是显示加载组件之前的时间量，这可以避免加载程序出现奇怪的闪烁，如果它立即显示，几毫秒后就会被真正的组件替换)。1500ms 后，将显示 ErrorState 组件，因为我们的帮助器函数等待 2000ms 才能解决。

你可以在这个沙盒里玩它:

如果您将`latency`更改为 1000 毫秒，您现在应该会看到加载组件在 200 毫秒后出现，然后是子组件在 800 毫秒后出现。

需要明确的是，这些代码更改只是为了测试目的，而不是为了生产。同样，如果你使用了大量的异步组件，你可能会把助手功能放在某个中心位置，或者把它合并到自动化测试中。

# 如何将道具传递给加载和错误组件？

你要做的下一件事是给用户提供比简单的“加载中…”或“错误！”更少的一般性消息。同时，您不希望使 LoadingState 和 ErrorState 组件特定于这个特定的上下文。如果您的应用程序中有另一个异步组件，重用它们会很好。听起来像是道具的工作！

但是情况下哪里能传道具呢？子组件当然接受道具，这很简单。但是似乎没有任何方法可以为加载和错误组件提供道具。没问题——事实证明，它们实际上与子组件本身收到了相同的道具。所以，我们可以这样做:

```
<child-component 
  v-if="showChild" 
  message="I am the child component."
  loadingMessage="Looking for child component..."
  errorMessage="The child component is unavailable.">
</child-component>
```

起初，这让我觉得有点恶心。但事实上，这些特定的错误和加载消息是特定于这个子组件的，甚至可能是这个子组件的实例，所以以这种方式传递它们似乎没问题。

`LoadingState`和`ErrorState`组件应该尽可能的轻，因为这项工作的重点是通过延迟加载较重的组件来初始化下载大小。类似于:

```
<template functional>
  <div><b>Error: {{ props.errorMessage }}</b></div>
</template><script>
  export default {
    name: 'ErrorState',
    props: ['errorMessage']
  }
</script>
```

# 结论

记住，loading 和 error 属性是关于异步加载*组件本身的——一旦组件被加载，它们与任何 API 调用或组件可能进行的其他异步操作是分开的。*

幸运的话，你的应用程序用户将永远保持良好、快捷的连接，永远不会看到这些精心制作的信息，永远不知道你为他们提供友好的微型副本付出了多少爱。但是，抱最好的希望，做最坏的打算。