# 测试可观测量的快速技巧👀

> 原文：<https://itnext.io/a-quick-tip-on-testing-observables-e2fbdebef4c?source=collection_archive---------2----------------------->

测试异步可观测性，就像它们是同步的一样！

![](img/70fc831b7f9388442b342d0cf0d6c391.png)

由[莎伦·皮特韦](https://unsplash.com/@sharonp?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

如果你用过 Angular (2+)，你一定用过 Observables。事实上，Angular 使用的实现——RxJs 紧密地结合在框架本身的基础中，用于路由和 Http 之类的东西。

如果你想了解更多关于 Observables 以及如何使用它们，你可以参考下面 [Gerard Sans](https://medium.com/u/9530b046d2ac?source=post_page-----e2fbdebef4c--------------------------------) 关于 RxJs 的酷帖。

[](https://medium.com/google-developer-experts/angular-introduction-to-reactive-extensions-rxjs-a86a7430a61f) [## 角度—反应式延伸(RxJS)简介

### 如何在 AngularJS 中使用可观察序列

medium.com](https://medium.com/google-developer-experts/angular-introduction-to-reactive-extensions-rxjs-a86a7430a61f) 

我们都知道(我希望如此)我们应该测试我们的代码，但是我们也知道测试异步代码可能很麻烦，尤其是在处理像 Observables 这样复杂的主题时。

例如，我们可以有以下方法:

在这里，我们返回一个平面数组，并故意将其延迟 500 毫秒。

我们如何测试这个？

这能行吗？显然不会，因为我们的测试运行程序不知道什么时候执行它的断言，并且会在 500 毫秒的延迟过去之前退出。

为了解决这个问题，我们可以使用 **done** 回调并指示测试运行程序何时执行其断言，比如:

这个可以，但是因为某些原因你不喜欢，对吧？这有点冗长，它有嵌套和一个每次都要调用的额外回调。

## 我们能做得更好吗？

我们当然可以，这是 Javascript，你可以做任何事情！

我向你展示了一个简单的助手函数，它将接收一个流(可观察的)和一个 skipTime。它将订阅可观察值，将其值赋给一个局部变量，**跳过一段时间**并返回值。

**等等，什么？** **你不能这么做！** `jasmine.clock.tick()`如何工作？

[](https://stackoverflow.com/a/28889643/1841820) [## 茉莉花钟是如何工作的？

### 我不想花几个小时阅读代码来找到相关的部分，但我很好奇 jasmine 是如何实现它的时钟的。的…

stackoverflow.com](https://stackoverflow.com/a/28889643/1841820) 

它基本上用定制函数模仿基于时间的 API，使这些调用同步，并让您在时间上“前进”到`ticking`。

您还需要在每次测试之前安装 jasmine 的时钟，然后卸载它，比如:

好的，但是我如何使用它？

很酷吧。同步，线性，简单！

你可以在 [ngx 可缓存](https://github.com/angelnikolov/ngx-cacheable)的**测试** [这里](https://github.com/angelnikolov/ngx-cacheable/blob/master/cacheable.decorator.spec.ts)看到更多用例。另外，如果您还没有阅读我之前关于简单缓存装饰器的文章，请阅读:

[](https://medium.com/@darkysharky/improve-your-angular-app-performance-by-using-this-simple-observable-cache-decorator-7d7ecea470d2) [## 使用这个简单的可观察缓存装饰器来提高 Angular 应用程序的性能🎉

### 当我们即将在 SwiftViews 中完成我们的应用程序开发时，我们注意到我们所有的…

medium.com](https://medium.com/@darkysharky/improve-your-angular-app-performance-by-using-this-simple-observable-cache-decorator-7d7ecea470d2) 

页（page 的缩写）在 Angular 的最新版本中，你也可以使用 NgZone 的`fakeAsync` + `tick`并获得相同的效果。然而，如果你在一个非 angular 项目中使用 Observables，你就不需要 zone.js 在测试中的开销，但是如果你测试一个 Angular 应用，也许那会是更好的选择。

![](img/d09768ad586a4129e75a8149eda183f0.png)

想找一份远程前端工作？
有空的时候通知我们！现在就订阅[https://www.remotefrontendjobs.com/](https://www.remotefrontendjobs.com/)