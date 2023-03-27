# 柏树+故事书。将测试场景、数据和组件呈现保存在一个地方

> 原文：<https://itnext.io/cypress-storybook-keeping-test-scenario-data-and-component-rendering-in-one-place-c57b23cc1640?source=collection_archive---------2----------------------->

![](img/266ac69b4a31d8b1926e72934562c31b.png)

本文的 RU 版本可在[此处](https://habr.com/ru/post/497544/)获得。

**重要更新#1**
故事书只是一个主机组件。您可以按照自己喜欢的任何方式捆绑和托管您的组件。例如，您可以在一个 JavaScript 文件中导入所有组件，并在测试期间通过 webpack-dev-server 提供服务。

**更重要的更新#2**
本文是 Cypress 4.5 版本以下的时候写的。
今天，我们对 Cypress 和 [cypress-react-unit-test 插件](https://github.com/bahmutov/cypress-react-unit-test)进行了惊人的更新，让一切变得更简单。它不需要主机(故事书)来存放你的组件，因为它在 Cypress 内部运行它们。

设置如下所述的唯一原因是速度或一些偶然的错误。首先尝试[cypress-react-unit-test](https://github.com/bahmutov/cypress-react-unit-test)。

一开始，塞浦瑞斯感觉像是 e2e 的测试工具。令人好奇的是，前端工程师对 Selenium 在何处发挥作用的话题越来越感兴趣。在那个时候，任何关于 Cypress 能力的典型视频或文章都被随意选择的网站所限制，并赞扬所提供的输入 API。

我们中的许多人选择 Cypress 作为工具来测试通过 Storybook/Styleguidist/Docz 托管的组件。一个很好的例子——[斯特凡诺·马尼的文章](/testing-a-virtual-list-component-with-cypress-and-storybook-494dc2d1d26b)。他建议创建故事书故事，将组件放在那里，并将重要数据暴露给全局变量，以便在测试中访问它。实际上，这是一个很好的方法，但是测试变成了故事书和 Cypress 之间的碎片。

在这里，我想进一步展示如何在 Cypress 中最大限度地执行 JavaScript。要查看它的运行，[您可以从我的 Github](https://github.com/daedalius/article-exposing-component-from-storybook-to-handle-them-in-cypress) 下载源代码，然后在控制台中执行 **npm i** 和 **npm 运行测试**。

TL；博士:

*   您可以公开 Storybook Story 中的组件引用，以便在 Cypress 中测试它(无需将测试逻辑分成几部分)。
*   Cypress 对我们的团队来说非常强大，所以我们没有另一个使用 js-dom 测试 UI 组件的工具。

# 任务

假设我们正在为现有的 Datepicker 组件编写一个适配器，以便在所有公司网站上使用它。我们不想意外打破任何东西，所以我们必须通过测试来覆盖它。

# 故事书

我们从 Storybook 中所需要的——一个在全局变量中保存对测试组件的引用的空故事。为了不那么没用，这个故事呈现了单个 DOM 节点。这个节点将是我们在测试中的战区。

好了，我们已经看完了故事书。让我们来看看柏树。

# 柏树

就个人而言，我喜欢从测试用例枚举开始。似乎我们有下一个测试结构:

好吧。我们必须在任何环境下进行这项测试。打开故事书，点击侧边栏中的*“在新标签中打开画布”*按钮，直接进入空故事。复制 URL 并让 Cypress 访问它:

正如你可能猜到的，为了进行测试，我们将使用*id = component-test-mount-point*在同一个 *< div >* 中呈现所有组件状态。为了使测试不会相互影响，我们必须在下一次测试执行之前卸载这里的任何组件。让我们添加一些清理代码:

现在我们准备完成测试。检索组件引用，呈现组件并做出一些断言:

你看到了吗？没有什么可以阻止我们将任何道具或数据直接传递给组件！这一切都在一个地方——在赛普拉斯！

# 用包装器分几步测试

有时我们想要测试组件根据变化的道具的可预测的行为。

检视 *<弹出>* 组件与“显示”道具。当【显示】为*真*时，*弹出<可见>弹出*。之后，将“显示”改为*假*，*，<弹出>，*应变为隐藏。如何测试这种转变？

这些问题很容易以命令的方式处理，但是在声明式反应的情况下，我们需要想出一些东西。

在我们的团队中，我们使用一个附加的带状态的包装组件来处理它。这里的状态是布尔型的，它响应“显示”的道具。

现在我们即将完成测试:

提示:如果这样的钩子不起作用或者你不喜欢在组件之外调用钩子——通过简单类重写包装器。

# 测试组件方法

其实我从来没写过这样的测试。写这篇文章的时候有了这个想法。用单元测试的方式来测试一个组件可能是有用的。

然而，你可以很容易地做到这一点。只需在渲染之前创建一个对组件的引用。值得一提的是，ref 提供了对组件的状态和其他元素的访问。

我在 *<弹出框>* 中添加了“hide”方法，使其被强制隐藏(为了举例而举例)。以下测试如下所示:

# 总结一下:每个参与者的角色

故事书:

*   托管包含用于测试目的的捆绑 React 组件的故事书故事。
*   提供一个真正的非合成环境来运行测试。
*   每个故事在全局变量中公开一个组件(稍后在 Cypress 中检索它)。
*   每个故事都公开了一个组件挂载点(在测试中挂载一个组件)。
*   能够在新标签中单独打开每个组件。

提示:请为您的组件库或页面运行 Storybook 的另一个实例。

柏树:

*   包含并运行测试和 Javascript。
*   访问孤立的组件故事，从全局变量中检索组件引用。
*   根据测试需要呈现组件(使用任何数据或测试条件，如移动分辨率)。
*   为您提供 UI，以便您可以看到您的测试进展如何。

# 结论

在这里，我想就阅读过程中可能出现的问题表达我个人的观点和我同事的立场。下面写的不冒充真实，可能与现实有出入，含有坚果。

我的测试工具在幕后使用 js-dom。我限制自己了吗？

*   Js-dom 是一个合成环境。分离的 DOM 不是真正的浏览器。
*   它实际上并不像用户那样使用 js-dom。尤其是在模拟输入事件时。
*   如果 CSS 中的一个组件由于一个不正确的 z-index 而被破坏，那么您能从编写的单元测试中获得多少信心？如果组件由 Cypress 测试，您将看到一个错误。
*   你盲目地编写单元测试。但是为什么呢？

**我应该选择建议的方法吗？**

*   如果你使用测试作为开发环境——当然，是的！
*   如果你看看在 **live** 文档中的测试——是的。
*   如果你真的编写单元测试来覆盖那些太接近实现和反应生命周期的东西，我不知道。好久没写这样的测试了。你确定覆盖的逻辑是组件责任吗？也许这种逻辑应该被提取出来并进行相应的测试？

那为什么不使用 cypress-react-unit-test 呢？我们为什么需要故事书？

我毫不怀疑——测试组件是我们的未来。不需要维护故事书的一个单独的实例，所有的测试将完全由 Cypress 负责，配置将被简化，等等。

̶b̶u̶t̶̶n̶o̶w̶̶t̶h̶e̶̶t̶o̶o̶l̶̶h̶a̶s̶̶[s̶o̶m̶e̶̶](https://github.com/bahmutov/cypress-react-unit-test/issues/34)p̶r̶o̶b̶l̶e̶m̶s̶̶t̶h̶a̶t̶̶m̶a̶k̶e̶̶t̶h̶e̶̶e̶n̶v̶i̶r̶o̶n̶m̶e̶n̶t̶̶p̶r̶o̶v̶i̶d̶e̶d̶̶i̶n̶c̶o̶m̶p̶l̶e̶t̶e̶̶f̶o̶r̶̶r̶u̶n̶n̶i̶n̶g̶̶t̶e̶s̶t̶s̶.̶̶h̶o̶p̶e̶̶t̶h̶a̶t̶̶[g̶l̶e̶b̶̶b̶a̶h̶m̶u̶t̶o̶v̶](https://github.com/bahmutov)̶a̶n̶d̶̶t̶h̶e̶̶c̶y̶p̶r̶e̶s̶s̶̶t̶e̶a̶m̶̶w̶i̶l̶l̶̶m̶a̶k̶e̶̶i̶t̶̶w̶o̶r̶k̶e̶d̶̶🤞

PS:我们团队的意见是建议的方法允许我们回顾使用 js-dom 的工具垄断。你怎么想？