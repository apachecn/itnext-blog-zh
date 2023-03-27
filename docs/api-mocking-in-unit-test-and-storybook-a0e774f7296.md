# 单元测试和故事书中的 API 模拟

> 原文：<https://itnext.io/api-mocking-in-unit-test-and-storybook-a0e774f7296?source=collection_archive---------1----------------------->

![](img/e75a3220e087889db782b8c5b51e6aed.png)

照片由[克里斯蒂娜@ wocintechchat.com](https://unsplash.com/@wocintechchat?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

这篇文章探讨了 API 模仿的三个层次，以及我们如何让模仿在故事书和单元测试中同样有效。我也会分享我在调试为什么嘲讽`XMLHttpRequest`不起作用的过程中的学习。如果您想继续查看最佳解决方案，请检查选项三:`msw`。

# 为什么

在 Web 开发中，我们不想依赖后端 API，原因有几个:

*   后端未准备好；
*   加快 UI 开发节奏

通常我们会在故事书和单元测试中模拟 API 响应来进行开发。故事书([示例](https://storybook.js.org/docs/react/workflows/build-pages-with-storybook#mocking-connected-components))和单元测试([示例](https://kentcdodds.com/blog/stop-mocking-fetch))的解决方案分别存在。我的目标是有一个一致的 API 来模拟故事书和单元测试，并在测试中重用故事书故事。

# 选项一:模仿 XMLHttpRequest

我的项目使用`superagent`作为 API 客户端，它在幕后使用`XMLHttpRequest`。我从用 [xhr-mock](https://www.npmjs.com/package/xhr-mock) 嘲讽出`XMLHttpRequest`开始(或者你也可以[不借助第三方库](https://stackoverflow.com/questions/28584773/xmlhttprequest-testing-in-jest)嘲讽)。我创建了一个 React 组件包装

我们可以在故事书里使用它

一件很酷的事情是我们可以在测试中导入这个故事

注意**我们只定义了一次服务器模拟响应*，并在测试****中重用它们。本质上，我们在各种场景中为组件创建故事(作为可视化测试)，并以编程方式对每个故事进行单元测试。很酷，不是吗？*

*…直到测试真正中断*

*这个错误表明我们的单元测试仍然试图发出一个真实的 http 请求，即使我们使用了 mock。到底怎么回事？*

*原来`superagent`在浏览器中使用`XMLHttpRequest`，在 nodejs 环境中使用`node:http`。在`superagent` package.json 中，你可以看到浏览器和 nodejs 加载了不同的文件。*

*不仅`superagent`而且其他 API 客户端(例如`axios`)也做同样的事情。原因是`XMLHttpRequest`是一个浏览器对象(通过`window.XMLHttpRequest`访问)。它不是 nodejs 中的对象；相反，nodejs 使用`node:http`进行 API 请求。*

*现在应该可以理解为什么我们的测试失败并带有`XMLHttpRequest`嘲讽了:单元测试在 nodejs 环境中运行，而`superagent`根本没有调用`XMLHttpRequest`！*

# *选项二:模仿超级代理*

*第二种选择是在更高的层次上进行模拟:我们的 API 客户端(在我的例子中是`superagent`，对于`axios`或其他的例子也是如此)。与`xhr-mock`类似，人们为嘲讽`superagent`做了库(如 [superagent-mock](https://www.npmjs.com/package/superagent-mock) )。让我们替换我们的 React 包装器:*

*类似地，我们可以在组件故事中使用这个包装器来模拟服务器响应，并在单元测试中重用它。这对两者都适用🎉*

*但还可以更好。*

# *方案三:都市固体废物*

*msw 在不创建服务器的情况下提供了最高级别的模拟。这个想法是**使用服务工作者来*拦截*所有请求，我们可以决定如何处理每个请求**。这个解决方案更好，因为*

*   *真正的 http 请求正在发生。这意味着您可以在浏览器的`network`选项卡中看到网络呼叫。这与模仿 API 客户端形成了对比:没有进行网络调用，这可能会使开发人员感到困惑，并误导他们相信组件本身没有发出任何请求。*
*   *更接近真实的用户体验。API 客户端的工作方式与生产环境中的相同。它带来了更多的信心，事情真的在工作。*
*   *您可以在处理请求时进行类似的服务器检查，例如*

*一个挑战是`msw`为浏览器和 nodejs 使用稍微不同的 API。让我们创建一个 React 包装器来提供一致的 API:*

*msw 对浏览器使用 service worker，对 nodejs 使用 monkey 补丁`http:node`。React 包装器使用适当的 msw 集成来返回模拟响应，而不管运行环境如何。现在你可以在故事书和单元测试中重用你的嘲讽，就像选项一中描述的那样。*

*还有一点关于`msw`的设置:*

*在单元测试设置中(`setupFilesAfterEnv`用于 jest)*

*在`.storybook/preview.js`中，启动服务人员*

*最后，使用 msw cli 创建一个服务工作者脚本*

*`$ npx msw init public`*

*这将在`public`文件夹中生成一个脚本(服务工人拦截实现)。我们应该将这个文件包含在 Git 中，并用它开始 storybook:*

*`$ start-storybook -s public`*

*现在你可以走了！*

# *摘要*

*这篇文章描述了模仿 API 的三个选项，从低级到高级。我建议高水平嘲讽，以获得最接近的生产经验。另外，在故事书和单元测试之间重用代码是对开发人员体验的真正提升！*

**请给出你最大的声音👏(s)如果你觉得这个帖子有帮助，就关注我的* [*twitter*](https://twitter.com/imDongCHEN) *！**