# NGRX |使用效果和路由器存储来隔离与路由相关的端🧙🏼‍♂️效应

> 原文：<https://itnext.io/ngrx-use-effects-and-router-store-to-isolate-route-related-side-%EF%B8%8F-effects-d18faaa750c1?source=collection_archive---------4----------------------->

`ngrx`的主要优势之一是我们可以将副作用从组件中分离出来。

![](img/8b9cc81196d5dffd0048b8f30519c5a1.png)

[https://unsplash.com/@abeosorio](https://unsplash.com/@abeosorio?utm_medium=referral&utm_campaign=photographer-credit&utm_content=creditBadge)

当我们需要组件上的路由器相关数据时，我们通常会使用组件本身的“ActivatedRoute”服务。例如，我们可以使用以下技术从一条路线中获取 id:

与 ActivatedRoute 服务交互的组件

让我们使用`@ngrx/effects`来隔离它。第一个目标是从 URL 获取路由参数。

# 如何不去做

我们可能会尝试在特效上使用`ActivatedRoute`服务，但这没用。它只能在“与插座中加载的组件相关联”上访问，引自[https://angular.io/api/router/ActivatedRoute](https://angular.io/api/router/ActivatedRoute)。

我们可能想到的另一件事是使用“路由器”服务。但是同样，我们无法以一种清晰的方式访问我们的参数。

我们能做的，就是使用来自`@ngrx/router-store`的`selectRouteParam`状态选择器。

让我们深入一下这个例子。(文章底部 stackblitz 链接)

# 怎么做才是正确的

设置效果:

要在注射剂上记录的效果

那么我们的组件可以简单到:

仅注册到商店服务的组件

# 外卖

这是一个演示如何获取参数的基本例子。当我们需要使用那个参数并从中获取一些数据时，真正的力量就来了。然后我们会有一个效果，我们将使用“mergeMap”操作符从 URL 中获取使用这个 id 的数据。在组件上，我们只需订阅获取的数据，就这样。

[https://stackblitz.com/edit/ngrx-effects-router-store](https://stackblitz.com/edit/ngrx-effects-router-store)

如果你喜欢它，请在这里或在我的 twitter 上给我一些爱。