# 碎木头

> 原文：<https://itnext.io/broken-logs-4619968f9cc3?source=collection_archive---------9----------------------->

![](img/d21ba2f4d04e6723c0e387e5eedb4933.png)

[Cold_deck_and_logging_crew，_Florence_Logging_Company，_ca_1916_(KINSEY_141)](https://upload.wikimedia.org/wikipedia/commons/7/72/Cold_deck_and_logging_crew%2C_Florence_Logging_Company%2C_ca_1916_%28KINSEY_141%29.jpeg)

## TL；DR: Node.js 让开发人员失望，因为完全没有日志框架和原语来支持它。

**免责声明**:这不是抨击 Node.js 的帖子，我在日常工作中使用 Node.js，我发现它非常有表现力和生产力。我对试图构建 node.js 并推进它的开源社区没有任何抱怨。但令我非常沮丧的是，我们现在处于 Node.js 11 中，却看不到这方面的进展。

## 记录

日志是开发和交付软件的重要部分。我们通过日志来宣布我们正在尝试启动应用程序，或者当我们完成启动并准备好接受连接时。当我们尝试连接到资源时，我们会记录日志。我们记录错误和 api 响应。我们可能会记录关于有问题的条件或状态的警告。我们甚至可以使用日志记录来报告某些度量(这不是最佳实践，但是我已经见过很多了)。

但是什么才是真正的“日志”呢？我先定义一下日志不是什么:日志不是“打印到 stdout”printf/fprint/echo/console . log()/print()/println 都不是日志。虽然它们允许我将文本输出到 stdout，但是它们仅限于 text(大部分)和 stdout，并且不允许 real logging 系统所允许的控制和精度。什么是日志记录？日志是一种以受控格式输出文本(主要是)消息的方式，允许我回答 4 个问题:发生了什么？那是什么时候发生的？那是在哪里发生的？发生的事情有什么重要性？。

日志记录的一个重要部分是将控制配置与实际日志记录语句分开的能力。我们可能会在程序中散布日志消息(具有不同的日志“级别”)，但是我们需要能够控制哪一类日志应该放在哪里。日志配置控件通常有“附加器”(python 有处理程序)，它定义了处理日志消息的方式。它将有过滤器，控制记录的内容，以及记录级别的概念。例如，一个日志配置将允许我从 INFO 级别和以上级别记录到 stdout。在错误级别日志的情况下，发送到 stdout 和一个专用的错误聚合服务(想想 rollbar 或 sentry)。在为敏感服务的 http 请求定义时，我还将记录到一个循环日志文件中，来自特定用户的消息除外，这些消息将被过滤掉。一个不错的日志记录配置将允许我将 sentry 错误日志记录级别更改为 INFO，或者在我记录的内容上添加另一个过滤器，而无需接触实际的应用程序数据。

## 记录系统和良好的记录系统

虽然真正的日志记录系统(与 console.log 相比)总是有时间戳的概念，但是好的日志记录解决方案(例如 python 中的)也可以自动添加“where”部分:什么文件/模块？什么功能？什么线？。

好的日志记录解决方案会自动处理上下文，比如异常堆栈跟踪(就像 python 中的 logger.exception 一样)。好的日志解决方案还会将日志消息评估推迟到实际的日志输出步骤，如果最低日志级别高于记录的消息，则不需要进行文本格式化。对于足够大的应用程序来说，这可能是一个很大的性能打击。

好的日志解决方案有一个内置的日志命名空间概念，以及一个内置的过滤功能。这是一个强大的工具，允许我打开/关闭某个应用程序或已安装库的日志记录的特定部分，而无需触及其余部分。

## 作为集成点进行日志记录

日志也是一个集成点。许多集中的错误/异常报告工具通过特定的日志处理器/附加器集成在一起。这是显而易见的——如果我记录一个异常或错误，我会在同一行代码中将它报告给错误聚合服务。但是有一个问题。只有在编程语言中有标准化的日志记录原语的情况下，这才是可能的。如果有，错误聚合提供者可以构建他们的 appender/handler 日志记录集成，并且知道这个解决方案可能在 99%的 python 应用上工作。但是，如果没有一个标准化的日志记录解决方案，或者至少没有一个普遍采用的解决方案，供应商需要维护许多不同的 node.js 客户端库，或者只是将这一负担转移给开发人员(这反过来可能会忽略这一集成，因为它需要额外的工作，而且没有时间，也没有人要求您这样做)。

## Node.js 日志记录的状态

虽然我希望我们已经过了用 console.log 和 console.error“记录日志”的日子(我们**已经过了那些日子了，对吗？)node.js 日志的状态还是破。**

stdlib 中缺少日志记录功能，而且我不知道在可预见的将来会有添加它们的计划。同样，仍然缺少实现更有用的日志解决方案所需的语言特性:比如“自动地”将模块名和行号提供给日志记录器。因此，我们似乎没有任何接近 stdlib 解决方案。

虽然我真的更喜欢 stdlib 解决方案，但似乎也没有普遍采用的“标准化”解决方案。零散的、部分维护的、开源的解决方案，真的不是解决问题的好办法。

## 但是！(我得到的常见回复)

*   "*我们不想要臃肿的标准库*！"stdlib 中的另一个库在后端并不重要，node.js 进程就在后端。此外，我最新的 node.js 服务只有 24 个直接依赖项(其中 2 个连接到日志记录，9 个用于林挺和测试),它的 node_modules 有 1404 个文件夹和子文件夹！我想知道其中有多少是与日志相关的，但是可以肯定的是，节点模块膨胀在社区中并不是一个真正的问题。
*   “*它不会与浏览器中的前端 js 一致*”——虽然 Emcascript 委员会确实将浏览器作为主要目标，但 Node.js 确实已经有了在浏览器中没有位置的库(比如说 fs)。这很酷，这是一个不同的运行时。
*   *我们已经抓到温斯顿了。只需使用 winston"* (用 bunyan/log4js 或您更喜欢的日志解决方案替换 Winston)——见上文。虽然其中一些库很好，没有完全崩溃，但它们仍然缺乏日志原语，无法成为完整的日志解决方案。即使你配置了 winston 来正确记录你所有的应用程序，你安装的 npm 模块也不太好。温斯顿帮不了你。
*   "*我们可以改变原型来做我们想做的事情* " —哦，即使每个人都使用 console.log(他们没有),这看起来仍然不是一个好主意，改变内置函数的行为。我敢肯定，如果我这么做，每个人都会开始尖叫“红色警报，红色警报”。

## 那么我们现在能做什么呢？

目前，我已经在我的项目中使用 log4js，这是一个很好的解决方案(在现有的约束下)。它有一个 appender 和级别的概念(以及使用一个行为类似过滤器的专用 appender 来控制 appender 的不同级别的能力:“logLevelFilter”)。

Log4js 还有一个配套的 log4js-api 库，我在可安装的 npm 模块中使用了这个库。Log4js-api 允许我的 lb 使用记录器，但是它不会做任何事情，除非使用库的应用程序安装了 Log4js 并配置了 appenders。一些开源库将日志记录公开为环境变量配置或对象/文件配置。其他库允许我在初始化和使用 logger 函数时传递它。这两个解决方案远非完美，但比我遇到的绝大多数 npm 库要好，它们只是忽略这一点，并以它们想要的任何方式打印日志/控制台。

## 前进的道路？

也许前进的道路可以绕过指导委员会？也许日志库可以采用日志存根(如 log4js-api)的思想，用一个通用的 api 来进行日志记录和配置，任何日志应用程序都可以连接和配置它？也许我们至少可以标准化不同的配置选项，这样所有的日志库都可以读取一个公共的日志配置？我当然希望如此，但我担心 js libs 的碎片化景观不会帮助我们到达那里。我相信 node.js 成长的一部分应该包括将日志作为平台的一部分。