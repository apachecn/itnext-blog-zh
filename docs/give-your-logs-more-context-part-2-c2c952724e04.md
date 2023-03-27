# 给你的日志更多的上下文—第 2 部分

> 原文：<https://itnext.io/give-your-logs-more-context-part-2-c2c952724e04?source=collection_archive---------0----------------------->

## 构建上下文记录器

这是我上一篇关于日志记录上下文的文章的继续。请查看它，以便更好地理解我们将要构建的内容的目的。

[](/give-your-logs-more-context-7b43ea6b4ae6) [## 给你的日志更多的上下文

### 如何理解 Node.js web 应用程序日志

itnext.io](/give-your-logs-more-context-7b43ea6b4ae6) 

# TL；速度三角形定位法(dead reckoning)

我们将基于这个故事构建的代码在我的 [Github](https://github.com/hbarcelos/give-your-logs-more-context) 上。如果只是想查看最终版本，可以在`master`分支获取。

# 介绍

上一次，我们通过使用`[pino](https://github.com/pinojs/pino)`和`[cls-hooked](https://github.com/jeff-lewis/cls-hooked])`通过并发请求来管理上下文。现在让我们围绕`pino`构建一个包装器，它将自动为我们处理这个问题。

![](img/cb6aa760a2ca1771f0ecb34c1ac26fc2.png)

现在，时间到了！

# 我们想要实现什么？

我们需要构建一个记录器，它将通过`cls-hooked`拥有基本的“全局”上下文，但也允许我们在实际调用记录器方法时扩充这样的上下文。

为了提高可重用性和互操作性，我们希望保持原来默认的`pino` [API](https://github.com/pinojs/pino/blob/master/docs/api.md) ，所以我们已经有了一个很好的测试用例集。此外，我们需要为我们的应用程序提供一种与上下文交互的方式。

# 我们将如何编写代码？

我们将实现这种包装 TDD 风格。然而，我们将编写的测试在严格意义上不是“单元”测试，因为它们将包含`pino`本身，并对生成的日志数据做出断言。这是可能的，因为`pino`接受一个自定义`[WritableStream](https://nodejs.org/api/stream.html#stream_writable_streams)`作为它的目的地。

作为测试框架，我们将使用`[ava](https://github.com/avajs/ava)`。请记住，虽然默认情况下`ava`trans files 测试文件，但如果没有正确设置`babel`，它不会对实际代码进行测试。为了避免增加该解决方案的复杂性，所有代码(包括测试)都不会使用 ES 模块或 Node.js 10.9.0 中不可用的任何功能。

如果您想按照实现进行操作，请查看 Github 资源库中的说明:

[](https://github.com/hbarcelos/give-your-logs-more-context) [## hbarcelos/give-your-logs-more-context

### pino 上的 give-your-logs-more-context-Wrapper 提供了与 cls-hooked 的集成，以在…

github.com](https://github.com/hbarcelos/give-your-logs-more-context) 

> 我试图让这个序列尽可能自然，只消除一些在常规编码会话中发生的内部循环和斗争。

# 实施步骤

## 初始设置

```
yarn init -y
yarn add pino cls-hooked
yarn add --dev ava
```

这是可能的，因为`pino`接受一个自定义`[WritableStream](https://nodejs.org/api/stream.html#stream_writable_streams)`作为它的目的地

## 确保日志级别的方法

为了简单起见，让我们坚持使用`pino`的默认日志级别:`trace`、`debug`、`info`、`warn`、`error`和`fatal`。

最简单的方法是:

`logger.js`目前只是一个返回普通`pino`实例工厂函数。`logger.test.js`文件为每个可用的方法生成一个测试用例，以确保我们以后不会破坏任何东西。

`parse-json-stream.js`是一个实用程序，它将解析日志输出流并返回普通的 Javascript 对象，使针对日志输出运行断言变得更加容易。

`stream-to-generator.js`有没有方便:`ava`和基于流的 API 玩不好。为了使测试更加简洁，我们将日志流转换为一个[生成器](http://2ality.com/2015/03/es6-generators.html)，该生成器生成对下一个日志条目的承诺。

后两者在我们努力实现的目标中并不重要，它们在此仅作参考。剩余的片段不会包括它们。

## 保留记录器方法调用的上下文

另外，请注意`pino`允许我们通过在参数列表前添加一个对象来将本地上下文传递给日志条目。这是我们想要保持的行为。

因此，让我们添加一个测试用例来涵盖这个场景:

因为到目前为止我们只是创建了一个`pino`实例，所以测试将会通过。

## 增加 CLS 意识

现在我们开始接触 CLS。首先，我们需要创建名称空间并将其公开:

## 防止实例之间共享 CLS 上下文

出于某种原因，我们可能希望在一个给定的应用程序中有多个记录器。这样做时，不要混合两者的名称空间，这一点很重要。然而，我们在上面实现的方法，所有的实例都有相同的名称空间`'@@logger'`，这可能会导致奇怪的行为。

解决这个问题最简单的方法是有一个`counter`变量，每当我们调用`createLogger`并将计数器值附加到名称空间名称时，这个变量就会递增。

虽然计数器并不是生成唯一名称的最安全的方法，因为它们在应用程序重新启动时被重置，但是在这种情况下它们是有效的，因为当服务器重新启动时，所有的 logger 实例都会被重新创建。此外，这个值不会暴露在任何地方，它只是为了创建不同的名称空间，所以我们没有问题。

> 有时候少即是多！

变化如下:

## 将 CLS 上下文应用于日志

这是一个很大的飞跃，所以请原谅我。首先，我们来看看代码中的变化，然后我们来讨论一下:

![](img/0408c3492c4ee72ef16c9ed809a289b2.png)

抱歉，我不能把这个分成更小的零钱:/

测试代码没有什么特别的，只是注意我们必须在`logger.cls.run`方法回调中运行我们的日志和断言。

不过，实际代码开始变得有趣了。我们利用 Javascript [代理](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)来拦截日志方法调用并修补它们的参数。

因此，在[第 52 行](https://gist.github.com/hbarcelos/a351b95fd869240f87cd12d5ad202742#file-logger-js-diff-L52)中，我们为我们的 logger 对象创建了一个代理，其处理程序被命名为`loggerObjectHandler` — [第 34–43 行](https://gist.github.com/hbarcelos/a351b95fd869240f87cd12d5ad202742#file-logger-js-diff-L34-L43)。处理程序定义了一个`[get](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/get)` [陷阱](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/get)，该陷阱将只拦截对日志方法`trace`、`debug`等的调用。它所做的是将这些方法包装到另一个代理中，其处理程序名为`logMethodHandler`—[lines 11–32](https://gist.github.com/hbarcelos/a351b95fd869240f87cd12d5ad202742#file-logger-js-diff-L11-L32)。

`loggerMethodHandler`收集 CLS 当前的活动上下文，从中排除一些不相关的属性。然后，基于当前的参数列表，它检查方法调用是否有本地上下文。如果我们不这样做，那么我们只需要将 CLS 上下文添加到参数列表中—第[20–23](https://gist.github.com/hbarcelos/a351b95fd869240f87cd12d5ad202742#file-logger-js-diff-L20-L23)行。否则，我们需要将本地上下文合并到 CLS 上下文中— [第 24–28 行](https://gist.github.com/hbarcelos/a351b95fd869240f87cd12d5ad202742#file-logger-js-diff-L24-L28)。最后，我们调用带有适当参数的原始方法— [第 30 行](https://gist.github.com/hbarcelos/a351b95fd869240f87cd12d5ad202742#file-logger-js-diff-L30)。

## 将更改传播到子记录器

来自`pino`的一个很好的特性是它允许我们通过`.child()`方法创建子记录器。子记录器维护其父记录器的所有属性，但也可以接受额外的上下文。所以，我们也需要让我们的下一代 CLS 意识到:

同样，新的测试是自我描述的。让我们把重点放在实现上。首先，我们将包装器创建提取到它自己的函数中，命名为`createWrapper`—lines[47–52](https://gist.github.com/hbarcelos/3c0d258329696e306c10ba3d3451d871#file-logger-js-diff-L47-L52)。这也允许我们为子记录器创建一个包装器。

接下来，我们定义一个`childMethodHandler`，它将拦截对`.child()` — [行 18–25](https://gist.github.com/hbarcelos/3c0d258329696e306c10ba3d3451d871#file-logger-js-diff-L18-L25)的调用。这个处理程序将在新创建的子记录器上调用`createWrapper`，将来自父记录器的 CLS 上下文作为参数传递。这将保证父节点和子节点(以及子节点子节点)都具有相同的上下文。

最后，我们修改了`loggerObjectHandler`的实现，使其也包含了`.child()`方法的代理——[第 30–45 行](https://gist.github.com/hbarcelos/3c0d258329696e306c10ba3d3451d871#file-logger-js-diff-L30-L45)——包括一些对条件的内部重构。

## 进一步的改进

到目前为止，我们的代码似乎工作正常，但它可能不是最佳的。一个容易发现的问题是，我们正在为子方法和日志方法的每个调用动态地创建新的代理。虽然这对于前者来说可能不是问题——因为我们不会经常给`.child()`打电话——但对于后者来说却不是。

为了防止这个问题，我们可以在创建 logger 本身时为所需的方法创建代理，并将它们作为 logger 对象的属性。当我们调用方法时，`loggerObjectHandler`会检查当前方法是否有代理集。如果是，则返回代理，否则，返回原始属性:

# 与我们的 web 应用程序集成

所以现在我们有了自己的伐木工厂。现在我们需要将它集成到我们的应用程序中。从上一篇文章的最后一个例子中，我们可以重构为:

# 结尾部分

上面的代码与我们目前在 easy carros[的生产中使用的代码非常相似，到目前为止，它对我们非常有效。如果你有任何可以改进它的建议，我们非常欢迎。](https://easycarros.com/)

你喜欢你刚刚读的吗？用 [tippin.me](https://tippin.me/@hbarcelos909) 给我买啤酒。

![](img/fb4d3ddc0f5a5e1695f79fdbd2a0d8d3.png)