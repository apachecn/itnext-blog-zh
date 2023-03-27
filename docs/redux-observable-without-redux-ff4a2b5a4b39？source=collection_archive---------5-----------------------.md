# Redux-没有 Redux 的可观察

> 原文：<https://itnext.io/redux-observable-without-redux-ff4a2b5a4b39?source=collection_archive---------5----------------------->

![](img/df69e74a4d3c23663f4479572b737715.png)

Gabriel Matula 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

## 用 Redux-Observable 编写无状态浏览器插件

*就像* [*为什么我用 Lua*](/pointlessly-writing-redux-in-lua-b64894e8ede5) *写 Redux 一样，我把这篇文章从头到尾写成了一个故事，这样你就可以了解我是如何在没有 Redux 的情况下使用 Redux-Observable 的，以及它是如何解决一个特定于我所处情况的业务问题的。*

*虽然这听起来可能是一个复杂的想法，但我强烈推荐使用 Redux-Observable，或者一个类似的风格，因为它确实有助于代码的可维护性，弄清楚动作和消息的流程，并使调试变得非常容易。*

# 历史

早在 11 月，我就开始了一项兼职工作，开始编写一个浏览器插件。虽然我在过去写了一个非常简单的程序，并事先做了一些研究，但直到我开始使用它时，我才意识到共享的信息是有限的，这也很奇怪，因为有这么多的浏览器插件，而且大部分都是 JavaScript、CSS 和 HTML。

最初，他们想让我派生出另一个开源项目，但是在评估了我们的选择并尝试了一会儿之后，我们决定从头开始构建一个。

# 从头开始

我习惯于从每个项目开始，用配置文件写出一个构建系统，然后从那里开始。我做过的几乎每个项目都是这样开始的，但是因为我是按小时计酬的，并且只能在业余时间工作，所以我不得不使用现有的解决方案。

在网上搜索了几个小时后，在 [StackOverflow](https://stackoverflow.com/) 上几乎什么也没找到，没有信息的文章，以及一些分散的 npm 项目，我登陆了[create-react-chrome-extension](https://github.com/suevalov/create-react-chrome-extension)。谢天谢地，我得到了使用 React 的许可，因为编写 HTML 和 vanilla JS 最终会让我编写一些节省时间的框架。

虽然这个 create-react-chrome-extension 的细节并不重要，但它为我提供了类似的创建-react-app 的体验，并帮助我开始使用。快进到今天，我将编写自己的系统，但这是一个奇妙的起点:)。

# 信息系统

虽然 Redux-Observable 是我在 RxJS 上使用过的最好的消息系统，但是它有一个巨大的需求:Redux。我签约的公司不是一家网络公司，所以复杂的 JS 是一个禁忌。虽然 Redux 很简单，但它需要很多知识，RxJS 也是如此，然后 Redux-Observable 是最重要的。我继续使用函数式编程，在此基础上增加更多的复杂性对于任何非 web 人员来说都是有害的；即使是一个经验丰富的浏览器插件开发人员也会很纠结。

我原本计划只工作几个月，所以在某些时候，我不得不把这些代码交给另一个开发人员。由于我在一个紧张的期限内工作，并且希望尽快得到一些可交付的东西，我开始直接用普通的 JS 来处理`background.js`中的业务逻辑。

可悲的是，这最终是一个糟糕的选择。当然，我有一些工作很快的东西，但是浏览器插件；至少那些使用 web 标准的(looking ' in you Safari)是完全基于消息的，大量使用回调。它看起来越来越像 Redux-Observable 一直是正确的选择，但是由于该插件实际上是无状态的——使用本地存储并将转换后的 AJAX 响应直接传递给其他特定于插件的函数——它没有意义。

# 添加 RxJS

接下来我做的就是介绍 RxJS。没过多久，这一堆复杂的回调逻辑就需要某种转换层了。如果你读了我关于用转换器加速 JavaScript 数组 [的文章，这就是需要转换器的插件。关于](/using-transducers-to-speed-up-javascript-arrays-92677d000096)[处理缓存和 AJAX 竞争条件](/handling-cache-and-ajax-race-conditions-4cb152db8764)的文章也是因为这个向 RxJS 的特殊转变而受到刺激。

没有进入太多的细节，我很快发现使用 RxJS 本身是不够的。我需要在它的基础上开发一个消息传递系统，使代码更容易使用。尽管如此，我还是不想引入 Redux 或 Redux-Observable，因为这些库的重量加上它们引入的代码复杂性对于这个简单的浏览器插件来说太大了。

因为 RxJS 主题是消息传递的门户，既是观察者又是被观察对象，所以我创建了相当多的主题，然后用这些主题加载了我的`background.js`文件:

```
someDataFromLocalStorage$
.subscribe()someOtherDataList$
.subscribe(someMessage$)
```

试图弄清楚一个主题从哪里开始，另一个主题从哪里结束，很快变得很难。这几乎就像是我在命令式地写作；传递函数以在其他函数完成处理后调用。这比根本不用它更糟糕！

更重要的是，一些 API 使用实际上可以接受同步返回值的回调，而其他 API 发送一个响应消息的回调。虽然后者很容易管理，但前者不是由 RxJS 的`fromEventPattern`自然处理的。我必须创建一些东西来发送预期的参数和同步回调函数来管理这个用例。这个解决方案要复杂得多，但是由于 RxJS 的可组合性，它完全是一个黑盒，应该是未来文章的主题。

# Redux-没有 Redux 的可观察

好消息来了，对吧？

在没有 Redux-Observable 的情况下，我已经创建了一个 Redux 兼容的`actions.js`文件来帮助判断插件的各个部分之间正在发送什么样的消息。这是我意识到我应该为应用程序的其余部分做类似的事情的地方，因为所有那些在`background.js`管理的订阅都太复杂了。

因为我已经在应用程序的一部分中有了一个 Redux 兼容的系统，所以我认为利用我对其 API 的了解来编写 Redux-Observable 的简化版本不会太难。

第一件事是创建一个`action$`主题，然后通过编写 RxJS 的`filter`操作符来编写我自己的`ofType`。

很简单:

在测试了一段时间后，我注意到如果遇到错误的话,`ofType`会导致整个插件死亡。为什么？不知道，但我不会让这阻止我。我创建了一个单独的用于本地调试的`actionLoggerEpic`,并将我的不正确动作错误处理程序放在那里。然后我让`ofType`有弹性:

虽然我可以将所有的逻辑放在一个`filter`中，但我选择使用 3，因为它更符合每个`filter`执行一个简单任务的情况。我还发现它更容易阅读。

现在对于好的东西，我需要一些方法来创建一个根史诗，然后订阅`action$`主题。可悲的是，我做了件蠢事:

这是可行的，但完全是愚蠢的。反正我没有正确使用`combineEpics`，这非常难读。现在看这个，对自己挺失望的。我想“哦，是的，Redux-Observable 有`combineEpics`和`createRootEpic`。我真聪明！”。

我非常快速地构建了这个解决方案，没有检查我的其他 Redux-Observable 项目或其源代码。事实上，我完全忘记了。我知道这很愚蠢。

没过多久，我就把它重构成了这样:

唷！之前的修改让我很困扰，不去想它我很难入睡。

综上所述，这是您可以在自己的项目中使用的内容:

这是一个无状态的 Redux-Observable，不需要 Redux。看到这是多么简单后，我的大脑“轰”的一声。我不敢相信我一直在避免使用 Redux-Observable，而实际上使用它是如此简单。

在重写了整个插件以利用 Redux-Observable 之后，它变得容易多了，我能够通过简单地记录每个动作并在出错的地方抛出错误来快速调试问题。

下面是`actionLoggerEpic`的样子:

我还添加了一个黑名单，或者我称之为`ignoreList`,用于隐藏记录太频繁的行为。

我还能够通过动作名称模块化所有东西，这修复了旧系统中可能出现的许多兔子洞调试。Redux-Observable 就像尽可能把所有东西都移到左边，与回调-hell 相反。

# 结论

如果你熟悉 Redux-Observable 或者另一个消息系统，比如 CQRS，我强烈建议你在异步应用中使用这些概念；尤其是像浏览器插件这样完全依赖异步发送消息的插件。

在过去几周围绕 Redux-Observable 重新编写了插件之后，更快地获得可交付成果变得更加容易了。

# 更多阅读

如果你喜欢你所读的，你也应该检查我的其他文章；尤其是 Redux 上的那些:

*   [在 React 组件中使用 Redux 还原剂](/using-redux-reducers-in-react-components-4e92985dd9cb)
*   [使用 Redux 的秘密:createNamespaceReducer](https://medium.com/@Sawtaytoes/the-secret-to-using-redux-createnamespacereducer-d3fed2ccca4a)
*   [为什么不需要 React-Redux 的“连接”](https://medium.com/@Sawtaytoes/why-you-shouldnt-need-connect-from-react-redux-498876de9e4e)
*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)