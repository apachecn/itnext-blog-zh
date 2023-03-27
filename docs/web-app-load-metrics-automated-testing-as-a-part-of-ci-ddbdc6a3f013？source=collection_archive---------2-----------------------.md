# 作为 CI 一部分的 Web 应用程序负载指标自动化测试

> 原文：<https://itnext.io/web-app-load-metrics-automated-testing-as-a-part-of-ci-ddbdc6a3f013?source=collection_archive---------2----------------------->

![](img/0f042380fe87d79938c9f9a0e6106bfc.png)

度量图像

现代 Web 应用程序在大多数情况下都在努力尽可能快地呈现 API 数据，并为 UI 呈现进行批处理。尤其是使用 React，有时我们必须检查我们的 web 应用程序加载/渲染时间如何受到一些第三方库或新功能的影响。

## 怎么开始的

大约两年前，当我们将我们的 web 应用程序/仪表板发布到产品中时，我们并不担心加载速度或渲染时间，因为应用程序很小，没有与之匹配的复杂性。但是随着时间的推移，当我们开始一个功能一个功能地发布时，应用程序的渲染时间变得更长了。这是几乎所有类型的应用程序开发过程的通常过程，但这里的关键是，我们只是在它非常慢之后才开始测量性能指标。

到了那一步，就意味着你要在 UI 端重构很多组件，或者调查 API 响应时间。对于我们的例子，我们只是在几个冲刺阶段阻止了新功能的开发，并通过代码重构和加载/渲染时间测量来处理，慢慢地提高了原始应用程序的速度和加载时间。

但是！这里是关键的事情:我们应该在每个特性发布时都这样做，当代码改进可以在小块中完成时。

## CI 期间的网站指标

当我们意识到我们必须花大量时间重构大多数应用程序组件时，我们决定以某种方式跟踪应用程序性能指标，修复它们，然后跟踪它对新特性的影响。因此，我们只是将 headless Google Chrome 集成到我们的自动化测试流程中，并开始在我们的时间序列数据库中保存性能指标，以跟踪随时间的变化，并在应用程序开始变慢时进行重构。

```
import puppeteer from 'puppeteer';
import devices from 'puppeteer/DeviceDescriptors';

....
const page = await browser.newPage();

// Getting Desktop metrics
await page.setViewport({ width: 1366, height: 768 });
await page.goto(url);
const metricsDesktop = await page.metrics();
...

// Getting Tablet metrics
await page.emulate(iPad);
await page.reload();
const metricsTablet = await page.metrics();
....

// Getting Mobile metrics
await page.emulate(iPhone);
await page.reload();
const metricsMobile = await page.metrics();
```

每当我们将代码推送到`master`分支，最终的应用测试开始运行时，这个过程就会发生。如果性能指标非常糟糕，我们的 CI 系统会拒绝新版本，这样我们就会知道，存在一些重大的负载/呈现问题。

## 自动获取结果

作为我们项目[Hexometer.com](https://hexometer.com)的一部分，我们开始收集全网的网站指标，并对 React 应用或 Angular 应用的平均加载时间进行内部排名。这将有助于提供基于 API 的 CI 集成，以便始终获得有关 web 应用指标的数据，并跟踪新功能发布的性能。

一般来说，每个 CI 平台都可以与`Google puppeteer`集成，并使用 headless chrome 作为服务器端浏览器测试的一部分。但是请记住，在 CI 平台和笔记本电脑上，您的 web 应用程序的整体性能指标可能完全不同。通常 CI 平台使用共享的服务器空间进行数百个测试，其中每个测试在不超过 512 MB ram 和 1 个 CPU 的容器内运行。这意味着所有网站指标都将与 CI 服务器上发生的事情相关，但总体流程不应有太大不同。对于我们的情况，我们设置了最小和最大垃圾收集，这在大多数情况下是可以的，但当然有时我们会遇到测试失败，而没有任何具体的性能降级，这只是因为我们的 CI 服务器过载，性能指标上升。

## 结论

恭喜你到了这一步！！现在我们知道为什么保持网站满负荷时间和应用程序渲染的指标是重要的。这有助于让用户满意，让你的代码不断优化。

拍手声👏并分享您在全自动测试方面的经验！