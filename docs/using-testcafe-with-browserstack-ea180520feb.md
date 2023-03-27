# 使用 TestCafe & BrowserStack 进行集成测试

> 原文：<https://itnext.io/using-testcafe-with-browserstack-ea180520feb?source=collection_archive---------1----------------------->

![](img/13469767c5fa86002168426a12c6584a.png)

via [TestCafe](https://devexpress.github.io/testcafe)

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fusing-testcafe-with-browserstack-ea180520feb)

我最近被分配了为一个工作项目设置集成测试的任务。我们在两个开源解决方案之间做决定，分别是 [Nightwatch](http://nightwatchjs.org/) 和 [TestCafe](https://devexpress.github.io/testcafe/) 。最终我们选择了 TestCafe，因为[它不是基于 Selenium](https://dzone.com/articles/testcafe-e2e-testing-tool) 的，后者允许更简单的设置和更少的工具。它还提供了对 ES6 语法的开箱即用支持，以及一个[便捷插件](https://github.com/DevExpress/testcafe-react-selectors)，用于扩展其内置选择器以轻松测试 React 组件。这里有一篇关于使用 TestCafe vs Nightwatch 的利弊的文章。

BrowserStack 是一个基于云的测试服务，它允许你启动几乎任何相关的浏览器，而不需要虚拟机。他们还为开源项目提供免费计划。我已经使用 BrowserStack 一段时间来手动检查我们的应用程序的跨浏览器兼容性问题，这显然不是最好的解决方案。幸运的是，TestCafe 通过一个简单的插件[test cafe-browser-provider-BrowserStack](https://github.com/DevExpress/testcafe-browser-provider-browserstack)提供了与 browser stack 自动化 API 的集成，这允许我们在整个支持的浏览器矩阵中运行自动化测试。

## 使用 TestCafe 的 CLI 和 BrowserStack

在一个或多个 BrowserStack 云浏览器中运行测试非常简单。首先安装 TestCafe 和 BrowserStack 插件:

```
npm install --save-dev testcafe testcafe-browser-provider-browserstack
```

接下来，用 shell 配置中的`BROWSERSTACK_USERNAME`和`BROWSERSTACK_ACCESS_KEY`设置环境变量。

最后，向您的 package.json 添加一个脚本:

```
"scripts": {
  "test:e2e": "testcafe 'browserstack:firefox@58.0:OS X HighSierra,browserstack:ie@11:Windows 10' e2e-tests/*.js --app 'commandToStartApp'"
}
```

运行`npm run test:e2e`并前往您的[浏览器堆栈自动化仪表板](https://www.browserstack.com/automate)查看您的测试。上面的命令将运行 Firefox 58 和 IE 11 中“e2e 测试”目录下的所有测试。关于使用 CLI 设置 TestCafe 的更深入的解释，请查看 Markus Oberleher 的文章。

## 浏览器堆栈并行工作进程限制

我得到的第一个教训是，BrowserStack 只根据您的计划提供一定数量的并行工作器。我们的开源计划碰巧给了我们 5 个，但是我天真地用 10 多个浏览器运行了上面显示的命令。令我恐惧的是，我意识到如果你让 TestCafe 让大量浏览器排队，它会使你所有的 BrowserStack 实例崩溃，无论你在 automate 仪表板上疯狂地点击多少次“停止会话”都救不了你。TestCafe 期望所有指定的浏览器都被连接起来，并自动并行运行您的测试。那么，我们如何才能利用我们分配的 5 名员工，而不把一切都搞砸呢？

TestCafe 不支持连续运行测试，所以我需要提出自己的解决方案。我没有使用 CLI 界面，而是创建了一个使用 TestCafe 节点 API 的脚本。

## 通过 BrowserStack 使用 TestCafe 的节点 API

在下面的要点中，我编写了一个异步函数来创建一个新的服务器实例。这个函数将浏览器作为一个字符串，`'browserstack:firefox@58.0:OS X HighSierra'`，或者可选地作为一个字符串数组，例如

```
[
   "browserstack:safari@11.0:OS X High Sierra",
   "browserstack:safari@10.1:OS X Sierra",
   "browserstack:edge@16.0:Windows 10",
   "browserstack:edge@15.0:Windows 10",
   "browserstack:ie@11.0:Windows 10"
]
```

同样的道理也适用于`testFiles`参数，它可以是你想要的测试文件的路径或者路径数组。

现在让我们使用这个函数为每批浏览器创建一个新的 TestCafe 实例。在这种情况下，我有 10 个浏览器需要测试。我最多有 5 个并行工作器可用，所以这意味着我需要将浏览器分成 2 批，每批将被传递给一个新的 TestCafe 实例。下面你会看到一个定义我们支持的浏览器矩阵的数组，以及一个循环遍历这些批处理的`startTests`函数。如果您的 BrowserStack 计划只支持 1 个并行工作器，那么您可以在该数组中定义所有浏览器，而不必将它们分成子数组。

现在，如果你运行这个，你会看到第一批浏览器将启动，只有当这些完成后，第二批才会启动。

您可能已经注意到，我们在上面只定义了一个测试:`e2e-tests/mytest.js`。您可能想要运行一整套测试，如果我们可以使用 glob 模式来获取所有的测试文件，而不是硬编码一个路径数组，那就太好了。不幸的是，TestCafe 节点 API [不支持 glob 模式](https://github.com/DevExpress/testcafe/issues/980)，所以我们需要创建一个简单的助手函数来完成这项工作。首先:

```
npm install --save-dev glob glob-promise
```

然后我们编写助手函数，并像这样更新我们的脚本:

现在我们有了一个脚本，它将根据分配给我们的工作线程来批量初始化我们的 BrowserStack 工作线程，并获取我们想要运行的测试文件。让我们更新我们包中的“测试:e2e”脚本。

```
"scripts": {
  "test:e2e": "node scripts/startTests.js"
}
```

## 使用 BrowserStack 节点 API

但是我们还有一个问题。如果两个开发人员在这个 repo 中工作，并且碰巧同时运行集成测试，会怎么样？或者我们可能希望将这个脚本工作到我们的 CI/CD 管道中，并且需要注意不要同时在本地运行该命令。在某种程度上，我们又回到了起点——没有任何措施来防止我们的 BrowserStack 工作人员过载。

为了解决这个问题，我将利用 [BrowserStack 节点 API](https://github.com/scottgonzalez/node-browserstack) 。在运行我们的测试之前，我们可以使用它来找出当前有多少正在运行的会话可用。

```
npm install --save-dev browserstack 
```

接下来，除了前面设置的环境变量(`BROWSERSTACK_USERNAME`和`BROWSERSTACK_ACCESS_KEY`)之外，我们还需要添加一个环境变量，即访问 BrowserStack 帐户的密码:

```
export BROWSERSTACK_PASSWORD=abc123
```

然后我们在下面添加助手函数。我们用我们的凭证创建一个新的客户机，然后使用`getApiStatus`方法来获取我们正在运行的会话的状态。

`getRunningBrowserstackSessions`将返回类似这样的响应:

```
{ 
   used_time: 214457,
   total_available_time: ‘Unlimited Testing Time’,
   running_sessions: 0,
   sessions_limit: 5 
}
```

我将简单地更新我们的`startTests`函数，以警告我们没有足够的可用工人，并在执行任何操作之前退出脚本。查看下面的最终脚本:

## 还要注意几个“陷阱”

*   [BrowserStack Safari 在测试 localhost URL](https://www.browserstack.com/question/663)时有问题，这就是为什么我没有像在文档中那样在 createTestCafe()工厂函数[中将“localhost”指定为主机名。否则 Safari 只会无限期挂起。](http://devexpress.github.io/testcafe/documentation/using-testcafe/programming-interface/createtestcafe.html)
*   这与编写实际的测试有关，但是我犯的一个错误是[在 http://localhost:3000 之后的页面夹具中遗漏了一个尾随/](https://github.com/DevExpress/testcafe/issues/2005) 。如果你使用散列路由器，这会把事情弄糟，并抛出一些疯狂的，不一致的错误。

```
fixture`My first test`.page`http://localhost:3000**/**`
```

希望这篇文章能帮助你避免我在使用这些工具时犯的一些错误。测试愉快！