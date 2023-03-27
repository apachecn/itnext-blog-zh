# 不要让 Angular Universal 击垮你(或你的应用)。一些让你保持理智的建议。

> 原文：<https://itnext.io/dont-let-angular-universal-break-you-or-your-app-some-tips-to-keep-your-sanity-5bf263495bb9?source=collection_archive---------0----------------------->

服务器端渲染(SSR)并不是没有警告。调试和热修复几乎是必然的。

![](img/1de120d9e5e93c3efa55f941f6d2a5bb.png)

你想给**提供两全其美的东西**。你想要一个单页面应用程序(SPA)，使用一个现代框架构建，比如 Angular，它具有令人难以置信的动态渲染、可扩展的基于组件的架构、令人难以置信的工具和渐进式 Web 应用程序( [PWA](https://developers.google.com/web/progressive-web-apps) )功能。

但是你也想要高质量的搜索引擎优化(SEO ),你意识到这严重依赖于搜索引擎及其机器人抓取和索引你的网站的能力。**然而**，尽管声称 AJAX 爬虫及其 javascript 渲染能力正在变得越来越好，但你动态插入的`<meta>`标签不会出现在社交卡片上，google fetch 或`$ curl [https://yoursite](https://yoursite)here.com`的预览会告诉你，你实际上并没有一个你发誓花了一年时间开发的网站。

此外，您意识到快速显示内容对用户参与度至关重要，并希望将您的 chrome lighthouse 分数提升到一个新的水平。

> [如果页面加载时间超过 3 秒，53%的移动网站访问会被放弃。](https://www.thinkwithgoogle.com/marketing-resources/data-measurement/mobile-page-speed-new-industry-benchmarks)

**服务器端渲染**是一个可行的解决方案，可以解决很多与你的 **SEO** & **性能**相关的问题。然而，重要的是要事先了解你需要做的改变，这样可以省去你很多麻烦，知道为什么一切都很好…直到你上了服务器(或者更糟；**展开**

# 什么是**角万能？**

**角度通用**是**服务器端渲染** (SSR)的角度特定解决方案。SSR 是通过服务器动态呈现应用程序静态版本的过程。Angular Universal 的魔力可以从它通过平滑的服务器到客户端的转换引导客户端的方式中看出；考虑到应用程序的延迟加载和动态特性；生成也是可发现的高性能应用程序。该指南可在[这里](https://angular.io/guide/universal)找到

 [## 角度通用

### Angular 是一个构建移动和桌面 web 应用程序的平台。加入数百万开发者的社区…

angular.io](https://angular.io/guide/universal) 

# **准备您的代码(isPlatformBrowser 是您的朋友)**

如果要在服务器上成功呈现代码，需要对代码进行一些重要的更改。每个代码库都是不同的，所以细节也会不同。然而，您通常可以将 SSR 特定的注意事项分成几个规则。

1.  **不要**直接访问 **DOM** (没有浏览器无法访问`document` / `window`)。
2.  **限制** / **移除**使用`setInterval`或类似**定时**功能
3.  确保你的依赖关系与 SSR 兼容。
4.  SSR 呈现应用程序的初始状态。举个例子，如果你的图片被延迟加载，使用下一个步骤( **5** )提供一个兼容 SSR 的替代方法来确保内容在那里。
5.  如果您需要阻止某些代码在服务器上运行，请将它包装在`isPlatformBrowser(this.platformID) {}`中

```
constructor(@Inject(PLATFORM_ID) private platformID: Object) {

    // run main initialisation code if (isPlatformBrowser(this.platformID)) {
       this.observable$ = interval(1000)... // safe to run code }
    // OR the alternative if (isPlatformServer(this.platformID)) { // run server side code 
    }}
```

# 要记住的事情(提示)

调试角度通用问题包括:

*   查看源代码中的预防性修复(代码不兼容问题)。
*   测试应用程序生产版本的渲染过程，在生产环境中进行**和**测试。这些要点很重要，因为一切看起来都很好，工作正常，直到您在更改后访问您的实时网站，发现它要么没有呈现，没有无数的错误，要么遇到网关超时相关的错误。

**记住**

*   您的应用程序将在每个路由请求上被渲染两次，因此请记住，除非您使用缓存，否则您将会重复数据请求(XHR)。
*   为了真正看到 SEO 的改进，确保填充`<meta>`标签所需的数据尽快得到解决。
*   任何`window of undefined`错误通常意味着你要么必须 1。**嘲笑**那些功能/行为，否则你必须 2。**阻止**服务器到达该代码(isPlatformBrowser)。从 1 号**开始**。，但是如果它不容易被模仿或者你真的不需要服务器端的功能，那么 **No 2** 。

# 如何收回你的灵魂(调试/修复)

*   请注意某些 **RXJS 运算符**，如`timer()`、`interval()`，因为它们在 SSR 的控制下会导致网关超时。
*   测试 SSR 时，使用**木偶师**或类似的无头浏览器。
*   当在服务器环境中测试时，**关闭 javascript** 以发现您的应用程序的 SSR 版本的全部功能。
*   在服务器平台上，对服务器请求使用绝对 URL。
*   检查你的依赖关系 **github 问题标签**中已知的不兼容性。
*   准备好几天来质疑你的知识、能力，以及你是否应该把你的代码扔进垃圾箱。

# 我一路上发现的一些宝石

**TransferHTTPCacheModule**

[](https://github.com/angular/universal/blob/master/docs/transfer-http.md) [## 角度/通用

### TransferHttpCacheModule 安装了一个 Http 拦截器，它可以避免客户端上重复的 HttpClient GET 请求，用于…

github.com](https://github.com/angular/universal/blob/master/docs/transfer-http.md) 

**多米诺骨牌**

[](https://www.npmjs.com/package/domino) [## 多米诺骨牌

### 基于 Mozilla 's dom.js 的服务器端 DOM 实现

www.npmjs.com](https://www.npmjs.com/package/domino) 

**木偶师**

[](https://developers.google.com/web/tools/puppeteer) [## 木偶师|网络开发者工具|谷歌开发者

### Puppeteer 是一个节点库，它提供了一个高级 API 来控制无头 Chrome 或 DevTools 上的 Chrome

developers.google.com](https://developers.google.com/web/tools/puppeteer) 

**SSR 兼容 Flex 布局**

[](https://github.com/angular/flex-layout/wiki/Using-SSR-with-Flex-Layout) [## 角度/柔性布局

### 为角度应用程序提供 HTML UI 布局；使用 Flexbox 和响应 API - angular/flex-layout

github.com](https://github.com/angular/flex-layout/wiki/Using-SSR-with-Flex-Layout)