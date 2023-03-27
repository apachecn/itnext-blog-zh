# 重构 AJAX 调用和观察值

> 原文：<https://itnext.io/how-to-safely-refactor-old-code-part-3-f285a6200988?source=collection_archive---------5----------------------->

![](img/246f23de581fe85bf5aff779b63d4940.png)

由[罗斯·芬登](https://unsplash.com/@rossf?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

## 安全重构旧代码:第 3 部分

在这个系列中，我们一步步地回顾安全重构旧代码的概念设计模式。我编写本指南是为了适应任何类型的重构场景，示例项目是 Node.js with RxJS。

*这是关于安全重构代码的 3 部分系列文章的最后一篇文章**:*

*   [*第 1 部分:防止现有系统的突破性变化*](https://medium.com/@Sawtaytoes/how-to-safely-refactor-old-code-part-1-a1a853263fec)
*   [*第二部分:不要只是重构，要让它具有可维护性*](https://medium.com/@Sawtaytoes/how-to-safely-refactor-old-code-part-2-2d09451b4e8f)
*   [*第 3 部分:重构 AJAX 调用和观察对象*](https://medium.com/@Sawtaytoes/how-to-safely-refactor-old-code-part-3-f285a6200988)

# 我们的故事到此为止

我们终于开始了大量的重构工作，但是现在我们需要通过重构 AJAX 调用和可观察的方法来完成它。

# AJAX 方法的单元测试

`doScaryLightFlash`是一个不使用依赖注入的数据获取函数。正因为如此，我们真的没有办法对这个函数进行单元测试，或者阻止它实际进行 AJAX 调用。

对我们来说，重构唯一真正重要的部分是`fetch`调用，因为这阻止了我们编写单元测试。我们最好的办法是用我们在 [**第二部分**](https://medium.com/@Sawtaytoes/how-to-safely-refactor-old-code-part-2-2d09451b4e8f) 中所用的相同方式来处理函数组合的依赖注入。

尽管这很简单，但它使我们有可能编写所有的单元测试，并最终用我们选择的任何库(包括单元测试的 fetch 模拟库)来切换节点获取。

看起来是这样的:

我使用 fetch-mock 是因为这是我能找到的最流行的解决方案。这些测试没什么特别的。如果我们得到一个 HTTP 202，我想知道它没有错误。虽然它可能不会，但如果它得到了 400 分，它应该会失败。因为这不是我们的 API，我们只需要确保我们正在进行正确的数据调用。我们甚至可以验证正文和 URL 是否与我们的期望值相匹配，但是对于这个项目来说，这并不重要。

# 完事了吗？

我可以就此打住，但是`createScaryLightFlasher`可以在未来受益于更多的配置。请记住，我们希望将它变成一个 npm 包，因此我们必须考虑其他用户的需求。

如果您想让多个灯组同时运行闪光器，当前功能将 LIFX 选择器硬编码在`config`中，不允许任何定制。该代码不应该驻留在数据提取器函数中。理想情况下，LIFX 选择器值位于应用程序的大图部分，我们使用 RxJS 来处理如何复制 AJAX 调用。

现在我们已经有了单元测试，我们可以安全地重构这个函数来获得更多的选项。在这种情况下，我们想要定制的只是 LIFX 选择器，这样任何人都可以调用这个灯光闪烁函数，并以他们想要的任何闪烁配置传递任何灯光配置。

这是最后一次重构:

现在，`lifxSelector`被传递给获取调用。因此，我们能够对其进行定制。如果有人正在使用这个库，现在他们可以传入任何他们想要的选择器，而不必使用来自`config`的单个值。

# 最终确定工作示例

我们最不想重构的是最外部的可观察性。它每 10 秒钟检查一次是否是万圣节。与其这样，为什么不设置一个可观察的计时器，在万圣节的时候启动，然后在这段时间内间隔 10 秒钟。时间超出范围后，切换回下一年的原始计时器可观察值。这样，你可以让应用全年运行，而不必担心它每 10 秒钟就要做一些事情。

看起来是这样的:

这是一次彻底的重构，需要很多改变；新功能和其他更新。这也意味着外部可观察对象现在有大量的逻辑，也需要单元测试，因为以前，外部可观察对象非常简单。现在，它正在做递归和设置等待计时器。

当然，这是正确的解决方案，但是现在逻辑更复杂了，还有更多的需要测试。如果我让它保持原样，我们就不会有这么复杂的应用程序，也不需要对外部可观察对象进行单元测试。这些是重构时要记住的事情。

我更喜欢当前的版本，但是我把旧版本留在 README 中作为一个更简单的例子，因为这个版本需要更多的时间来理解，并且没有单元测试，它也是未经测试的。

# 结论

重构工作量很大。你真的需要确保你有所有必要的业务需求。一旦为现有代码编写了自动化测试，进行重构就容易多了，但是最重要的是弄清楚在哪里花时间编写测试，并以最小的改动获得可测试的代码。

如果你想亲自体验 lifx-万圣节，你可以在 npm 上找到[套餐。](https://www.npmjs.com/package/lifx-halloween)

# 更多阅读

如果你喜欢你所读的，你也应该看看我关于智能家居和函数式编程的其他文章:

*   [RxJS 和可观察的 Flic 按钮](https://medium.com/flicblog/flic-buttons-and-the-observable-customization-using-rxjs-2214bc53d407)
*   [轻触按钮控制物联网设备](https://medium.com/flicblog/controlling-iot-devices-with-the-flic-of-a-button-1349c81bddef)
*   函数式编程的表情爱好者指南:第一部分