# UI 测试调试最佳实践

> 原文：<https://itnext.io/ui-tests-debugging-best-practices-789c4ed4daf6?source=collection_archive---------2----------------------->

一系列最佳实践，在使用 Puppeteer 和其他浏览器自动化工具时简化了测试编写和调试阶段。

![](img/ca9e4e560f577fcdf5f2edb10768d662.png)

由[泰克顿](https://unsplash.com/@tekton_tools?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

我正在 GitHub 上做一个大的 [UI 测试最佳实践](https://github.com/NoriSte/ui-testing-best-practices?source=post_page---------------------------)项目，我分享这个帖子来传播它并有直接的反馈。

在搬到 Cypress 之前，我习惯于用 Puppeteer 编写 UI 测试。理解浏览器中发生了什么，正在运行哪个测试，以及调试测试都不是容易的任务，因此我开始应用一系列的解决方案来帮助我完成整个过程。

像 [Cypress](https://www.cypress.io/) 和 [TestCafé](https://devexpress.github.io/testcafe/) 这样的工具几乎没有用，但是你不会意识到一个专门制作的工具简化了你的生活，除非你以前有过使用像 [Selenium](https://www.selenium.dev/) 或[木偶师](https://pptr.dev/)这样的工具的测试经验。

第一步是以非无头模式启动浏览器，然后…

## Console.log/show 测试的描述

由于您没有关于哪个测试正在浏览器中运行的视觉反馈，请记住在浏览器控制台中记录测试的名称。这在快速测试(不到 1 秒)的情况下毫无用处，但在更长的测试或在使用`test.skip`和`test.only`时对正在运行的测试进行双重检查时很有帮助。

在木偶戏中，这可以通过以下方式实现:

```
test('Test description', async () => {
  await page.evaluate(() => console.log('Test description')); // ... the test code...
})
```

如果您需要更深入的反馈，您甚至可以考虑在页面的左上角添加一个固定的 div，每个测试都用自己的描述填充它…

## 将浏览器 console.log 转发到 Node.js

用木偶师，一个简单的

```
page.on('console', msg => console.log('BROWSER LOG:', msg.text()));
```

允许您在同一个终端窗口中读取测试日志和浏览器日志。简单有效。

## 使用已经打开的 devtools 启动浏览器

像在传统的前端开发中一样，在页面加载已经开始后打开 devtools 可能会让你错过重要的东西，尤其是在 network 选项卡中。调试测试时，用已经打开的 devtools 启动浏览器可以节省您宝贵的时间和信息。

```
const browser = await puppeteer.launch({
 headless: false,
 **devtools: true**
});
```

## 减慢模拟的用户动作

浏览器自动化工具非常快，这让我们可以在几秒钟内运行大量测试。在调试时，这可能是一个缺点，因为你需要用眼睛关注页面上发生的事情。放慢**每一个动作**可能会适得其反——因为整个测试变得缓慢——但通常，这是执行一些快速检查的最简单方法。在《木偶师》中，有一个全局设置来做这件事。

```
const browser = await puppeteer.launch({
 headless: false,
 slowMo: 250, // slow down every action by 250ms
});
```

有些动作，如打字，允许您添加更具体的延迟(发生在全局`slowMo`设置之上)

```
await page.type('.username', 'admin', {delay: 10});
```

## 用调试器语句暂停测试

同样，像在标准的 web 开发中一样，您可以在页面上运行的代码中添加一个`debugger`语句来“暂停”JavaScript 的执行。请注意:只有在受控浏览器中打开 devtools 时，该语句才有效。

```
await page.evaluate(() => {debugger;});
```

通过点击“恢复脚本执行”或按下 F8 ( `debugger`是一个“飞行”断点)将恢复测试执行。

## 增加测试超时

试跑者如 Jest、Jasmine 等。有测试超时。如果出现问题并且测试没有结束，超时对于终止测试是有用的。在 UI 测试中，这种行为很麻烦，因为测试开始时打开浏览器，测试结束时又关闭浏览器。在正常测试的生命周期中，高超时是不起作用的，因为它会在失败的情况下导致巨大的时间损失，而低超时可能会在测试完成之前“截断”测试。

相反，您需要长时间的超时，因为您不希望在测试结束时关闭浏览器。这就是为什么在调试受控浏览器时，10 分钟的超时会很有帮助。

否则，你可以…

## 测试结束时避免关闭浏览器

当测试开始时，您打开浏览器，然后在测试结束时关闭它。避免关闭它给你检查前端应用程序的自由，而不用担心测试超时。这仅在本地运行测试时有效，并且在 CI 管道上运行测试之前必须恢复自动关闭，以避免由于浏览器实例保持打开而导致内存不足。

## 截图

这在无头模式下运行测试时特别有用，在这个阶段，测试被认为是稳定的，只有在回归的情况下才会失败。如果一个测试失败了，很多时候屏幕截图会让你明白你正在开发的特性是如何影响一个先前工作的特性的。最有效的解决方案是在测试失败时截屏，否则，您可以在 UI 测试中识别一些检查点，并在这些步骤中截屏。

## 频繁断言

一个经验法则:如果一个测试失败了，它必须直接驱动你去理解哪里出错了，而不需要重新启动测试和手工调试它。尝试在你的代码库中手工引入 bug(改变请求负载，删除元素，等等)。)并检查测试报告的内容。该错误与您引入的 bug 相关吗？谁读了失败能理解什么需要被修理？

你需要在你的测试中添加很多断言，这完全没问题！单元测试通常只需要一个步骤和一两个断言，但是 UI 测试不同，它们有很多步骤，因此你需要很多断言。可以把它们想象成一系列的 uni 测试，其中前一个测试是为第二个测试创造基础所必需的，以此类推。

## 使用 test.skip 和 test.only

这是每个测试跑步者的基本要求之一，但是你永远不知道:如果你不习惯使用 skip 和 only，现在就开始吧！否则，你将会浪费很多时间，即使你的测试文件仅仅由两三个测试组成。总是运行最少数量的测试，否则你需要调试！

## 连续运行测试

如果你结合 Jest 使用 Puppeteer，记住 Jest 有一个专用的`runInBand`选项，防止它将测试的执行分散到你的 CPU 内核上。并行化测试对于加速执行是完美的，但是当你需要用你的眼睛跟随测试动作时，这是令人讨厌的。`runInBand`连续运行测试。结合使用`test.skip`、`test.only`和 [jest-watch-typeahead](https://github.com/jest-community/jest-watch-typeahead) 可以让你省去很多调试的麻烦。

## 保持你的测试代码简单

比起抽象，更喜欢复制。努力让你的测试代码简单易读。你调试 UI 测试越多，你就越会意识到它有多难。一旦你需要理解正在发生的事情以及哪一步没有按预期工作，你的高度抽象的、完全干燥的测试代码就变成了一种痛苦。

更一般地说，测试是小脚本，必须比它们测试的代码简单两个数量级，让它们成为盟友，而不是一些更复杂的程序。

# 选择专门制作的工具

上面提到的都是有效的解决方案，但是它们有一个共同点:**它们是变通方法**。事实上，我在示例中使用的工具 Puppeteer 不是为 UI 测试而创建的，而是为通用浏览器自动化而创建的，那么，在一些外部实用程序的帮助下并在测试中使用，Puppeteer 可以用于 UI 测试。与自动化操作相比，测试 web 应用程序有不同的需求，需要不同的工具。

如果您需要编写 UI 测试，您应该考虑迁移到 Cypress 或 TestCafé，因为它们旨在简化您的测试工作。怎么会？拥有大量实用程序和默认行为，以及一系列一流的解决方案，允许您了解和调试浏览器中发生的事情。考虑到本文中提到的所有布袋戏演员**的最佳实践**……**对 Cypress 或 TestCafé** 完全没用😉

我的[一些 UI 测试问题和 Cypress 方式](https://medium.com/@NoriSte/some-ui-testing-problems-and-the-cypress-way-22312ab986e3)和[前端生产力提升:Cypress 作为您的主要开发浏览器](https://medium.com/@NoriSte/front-end-productivity-boost-cypress-as-your-main-development-browser-f08721123498)文章包括 Cypress 一流实用程序的概述。

# 你可能会对我的其他文章感兴趣

*   为什么测试一个 web 应用程序如此困难？为什么通用浏览器自动化工具不能很好地满足用户界面/E2E 测试的需求？柏木为什么出众？[一些 UI 测试问题和柏道](https://medium.com/@NoriSte/some-ui-testing-problems-and-the-cypress-way-22312ab986e3)
*   为什么测试非常适合讲述你的代码:[软件测试作为文档工具](https://medium.com/@NoriSte/software-tests-as-a-documentation-tool-e1c463bad1be)
*   睡眠:E2E 测试中最糟糕的实践和确定性事件的概念。[等待，不要让您的 E2E 测试休眠](https://medium.com/@NoriSte/await-do-not-sleep-your-e2e-tests-df67e051b409)

你好👋我是 Stefano Magni，我是一名热情的**反应和打字开发者**，一名**柏树大使**和一名**讲师**。我作为一名高级前端工程师为 WorkWave 做远程工作。我喜欢创造高质量的产品，测试和自动化一切，学习和分享我的知识，帮助他人，在会议上发言，以及面对新的挑战。
你可以在 [Twitter](https://twitter.com/NoriSte?source=post_page---------------------------) 、 [GitHub](https://github.com/NoriSte?source=post_page---------------------------) 、 [LinkedIn](https://www.linkedin.com/in/noriste/?source=post_page---------------------------) 上找到我。你可以找到我最近所有的投稿/演讲等。关于[我的 GitHub 总结](https://github.com/NoriSte/all-my-contributions)。