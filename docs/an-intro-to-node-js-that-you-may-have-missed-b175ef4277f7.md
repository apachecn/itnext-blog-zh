# 你可能错过的 Node.js 简介

> 原文：<https://itnext.io/an-intro-to-node-js-that-you-may-have-missed-b175ef4277f7?source=collection_archive---------2----------------------->

![](img/baeb2c66b220e8200bda012eb7d8e5c6.png)

扎卡里·扬在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

众所周知，Node.js 是一个开源的跨平台 JavaScript 运行时。大多数 Node.js 开发人员都知道它是建立在 V8(JS 引擎)和 libuv(基于事件循环提供异步 I/O 支持的多平台 C 库)之上的。但只有少数开发人员能够清楚地解释 Node.js 内部是如何工作的，以及它如何影响他们的代码。那大概是因为很多 Node.js 开发者在学习 Node 之前就已经知道 JavaScript 了。所以，他们经常从 Express.js，Sequelize，Mongoose，Socket 开始学习 node。IO 等知名库，而不是把时间投入到学习 Node.js 本身和它的标准 API 上。对我来说，这似乎是一个错误的选择，因为理解 Node.js 运行时和了解内置 API 的细节可能有助于避免许多常见错误。

这篇文章以简洁而全面的方式介绍了 Node.js。我们将对 Node.js 架构做一个总体概述。因此，我们将尝试确定一些使用 Node.js 编写更高性能、更安全的服务器端 web 应用程序的指导原则。这对 Node.js 初学者以及有经验的 js 开发人员应该很有帮助。

# 主要构件

任何 Node.js 应用程序都是基于以下组件构建的:

*   [V8](https://v8.dev/)——谷歌开源的高性能 JavaScript 引擎，用 C++编写。谷歌 Chrome 浏览器等也在使用。Node.js 通过 V8 C++ API 控制 V8。
*   libuv——一个多平台支持库，专注于异步 I/O，用 c 语言编写。它主要是为 Node.js 开发的，但也为 Luvit、Julia、pyuv 和其他人使用。Node.js 使用 libuv 将非阻塞 I/O 操作抽象到所有支持平台的统一接口。这个库提供了处理文件系统、DNS、网络、子进程、管道、信号处理、轮询和流的机制。它还包括一个线程池，也称为工作池，用于卸载一些不能在操作系统级别异步完成的工作。
*   其他开源、底层组件，大多用 C/C++编写:
    -[C-ares](https://c-ares.haxx.se/)-一个异步 DNS 请求的 C 库，用于 Node.js 中的一些 DNS 请求
    -[http-parser](https://github.com/nodejs/http-parser)-一个轻量级的 HTTP 请求/响应解析器库。
    - [OpenSSL](https://www.openssl.org/) —知名的通用密码库。用于`tls`和`crypto`模块。
    -[zlib](https://zlib.net/)-一个无损数据压缩库。用于`zlib`模块。
*   应用程序——它是用 JavaScript 编写的应用程序代码和标准 Node.js 模块。
*   C/C++绑定——围绕 C/C++库的包装器，用 N-API 构建，一个用于构建本机 Node.js 插件的 C API，或其他用于绑定的 API。
*   Node.js 基础设施中使用的一些捆绑工具:
    -[NPM](https://www.npmjs.com/)-一个知名的包管理器(和生态系统)。
    -[gyp](https://gyp.gsrc.io/)-基于 python 的项目生成器，从 V8 复制而来。由 node-gyp 使用，这是一个用 Node.js 编写的跨平台命令行工具，用于编译本机插件模块。
    -[gt est](https://github.com/google/googletest)-谷歌的 C++测试框架。用于测试本机代码。

下面是一个简单的图表，显示了列表中提到的主要 Node.js 组件:

![](img/0afe54aa63d08ff6966f564c04396d89.png)

主 Node.js 组件

# Node.js 运行时

下图显示了 Node.js 运行时如何执行 js 代码:

![](img/7cccdb4a8a23373a1a1bf71a230babb3.png)

Node.js 运行时图(简化)

这个图没有显示 Node.js 中发生的所有细节，但是它突出了最重要的部分。我们将简要讨论它们。

一旦 Node.js 应用程序启动，它首先完成一个初始化阶段，即运行启动脚本，包括请求模块和注册事件回调。然后应用程序进入事件循环(也称为主线程、事件线程等。)，它在概念上是为了通过执行适当的 JS 回调来响应传入的客户端请求而构建的。JS 回调是同步执行的，可能会使用节点 API 来注册异步请求，以便在回调完成后继续处理。这些异步请求的回调也将在事件循环中执行。这种节点 API 的例子包括各种定时器(`setTimeout()`、`setInterval()`等)。)、`fs`和`http`模块的功能等等。所有这些 API 都需要一个回调，一旦操作完成，回调就会被触发。

事件循环是基于 libuv 的单线程半无限循环。之所以称之为半无限循环，是因为它会在某个时刻退出，这时已经没有工作可做了。从开发者的角度来看，这是你的程序退出的时候。

事件循环相当复杂。它假设对事件队列进行操作，并包括几个阶段:

*   计时器阶段——该阶段执行由`setTimeout()`和`setInterval()`安排的回调。
*   挂起回调阶段—执行推迟到下一次循环迭代的 I/O 回调。
*   空闲和准备阶段—内部阶段。
*   轮询阶段—包括以下内容:检索新的 I/O 事件；执行与 I/O 相关的回调(除了 close、timers 和`setImmediate()`回调之外的几乎所有回调)；Node.js 将在适当的时候阻塞这里。
*   检查阶段— `setImmediate()`回调在这里被调用。
*   关闭回调阶段——这里执行一些关闭回调，例如`socket.on('close', ...)`。

**注**。查看以下[指南](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)以了解更多关于事件循环阶段的信息。

在轮询阶段，事件循环通过使用 libuv 对特定于操作系统的 I/O 轮询机制的抽象来实现非阻塞的异步请求(通过节点 API 启动)。这些特定于操作系统的机制是用于 Linux 的 epoll、用于 Windows 的 IOCP、用于 BSD 和 MacOS 的 kqueue、Solaris 中的事件端口。

Node.js 是单线程的，这是一个普遍的误解。本质上，这是正确的(或者曾经是部分正确的，因为有对 web workers 的实验性支持，称为 Worker Threads ),因为您的 JS 代码总是在事件循环中的单个线程上运行。但是您可能还会注意到图中的 Worker 池，这是一个固定大小的线程池，因此任何 Node.js 进程都有多个并行运行的线程。原因如下:并非所有节点 API 操作都可以在所有支持的操作系统上以非阻塞方式执行。使用工作池的另一个原因是事件循环不适合 CPU 密集型计算。

因此，Node.js(尤其是 libuv)尽最大努力为这种阻塞操作保持相同的异步、事件驱动 API，并在单独的线程池上执行这些操作。以下是内置模块中此类阻塞操作的一些示例:

*   I/O 绑定:
    -`dns`模块中的一些 DNS 操作:`dns.lookup()`、`dns.lookupService()`。
    -大部分文件系统操作由`fs`模块提供，如`fs.readFile()`。
*   CPU 绑定:
    —`crypto`模块提供的一些密码操作，如`crypto.pbkdf2()`、`crypto.randomBytes()`或`crypto.randomFill()`。
    -`zlib`模块提供的数据压缩操作。

注意，一些第三方本地库，比如`bcrypt`，也将计算卸载到工作线程池。

现在，当您应该对 Node.js 的整体架构有了更好的理解时，让我们讨论一些编写更高性能、更安全的服务器端应用程序的准则。

# 规则 1——避免在函数中混合同步和异步

当您编写任何函数时，您需要使它们要么完全同步，要么完全异步。您应该避免在一个函数中混合使用这些方法。

**注**。如果一个函数接受回调作为参数，这并不意味着它是异步的。举个例子，可以想到`[Array.forEach()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)`函数。这种方法通常被称为[延续传递式](http://matt.might.net/articles/by-example-continuation-passing-style/) (CPS)。

让我们以下面的函数为例:

```
const fs = require('fs')function checkFile (filename, callback) {
  if (!filename || !filename.trim()) {
    // pitfalls are here:
    return callback(new Error('Empty filename provided.'))
  } fs.open(filename, 'r', (err, fileContent) => {
    if (err) return callback(err)

    callback(null, true)
  })
}
```

这个函数很简单，但是很好地满足了我们的需求。这里的问题是`return callback(...)`分支，因为回调是在无效参数的情况下同步调用的。另一方面，在有效输入的情况下，回调在`fs.open()`调用中以异步方式调用。

为了显示这段代码的潜在问题，让我们尝试用不同的输入来调用它:

```
checkFile('', () => {
  console.log('#1 Internal: invalid input')
})
console.log('#1 External: invalid input')checkFile('main.js', () => {
  console.log('#2 Internal: existing file')
})
console.log('#2 External: existing file')
```

该代码将向控制台输出以下内容:

```
#1 Internal: invalid input
#1 External: invalid input
#2 External: existing file
#2 Internal: existing file
```

你可能已经注意到这里的问题了。在这些情况下，代码执行的顺序是不同的。它使函数不确定，因此必须避免这种风格。通过用`setImmediate()`或`process.nextTick()`包装`return callback(...)`调用，该函数可以很容易地固定为完全异步的风格:

```
if (!filename || !filename.trim()) {
  return setImmediate(
    () => callback(new Error('Empty filename provided.'))
  )
}
```

现在我们的函数变得更加确定了。

# 规则 2——不要阻塞事件循环

就服务器端 web 应用程序而言，例如 RESTful 服务，所有请求都在事件循环的单线程中并发处理。因此，举例来说，如果应用程序中的 HTTP 请求处理在执行计算量很大的 JS 函数上花费了大量时间，那么它会阻塞所有其他请求的事件循环。作为另一个例子，如果您的应用程序在处理每个 HTTP 请求的 JS 代码上花费 10 毫秒，那么应用程序的单个实例的吞吐量大约是每秒 1000 / 10 = 100 个请求。

因此，Node.js 的第一条黄金法则是“永远不要阻塞事件循环”。这里有一个简短的建议列表，可以帮助你遵循这个规则:

*   避免任何繁重的 JS 计算。如果你有时间复杂度低于 O(n)的代码，考虑优化它，或者至少把计算分成块，通过定时器 API 递归调用，比如`setTimeout()`或`setImmediate()`。这样，您将不会阻塞事件循环，并且其他回调将能够被处理。
*   在服务器应用程序中避免任何`*Sync`调用，如`fs.readFileSync()`或`crypto.pbkdf2Sync()`。这条规则的唯一例外可能是应用程序的启动阶段。
*   明智地选择第三方库，因为它们可能会阻塞事件循环，例如通过运行一些用 JS 编写的 CPU 密集型计算。

# 规则 3——明智地阻止工人池

这可能令人惊讶，但工人池也可能被阻塞。众所周知，这是一个固定大小的线程池，默认大小为 4 个线程。可以通过设置`[UV_THREADPOOL_SIZE](https://nodejs.org/api/cli.html#cli_uv_threadpool_size_size)`环境变量来增加大小，但是在很多情况下这并不能解决你的问题。

为了说明工人池问题，让我们考虑下面的例子。RESTful API 有一个身份验证端点，它计算给定密码的哈希值，并将其与从数据库中获得的值进行匹配。如果你做的一切都是正确的，散列是在工人池中完成的。让我们想象一下，每次计算需要大约 100 毫秒才能完成。这意味着，在默认的工作池大小下，就哈希端点的吞吐量而言，您每秒将获得大约 4*(1000 / 100) = 40 个请求(重要说明:我们在这里考虑 4 个以上 CPU 核心的情况)。当工作池中的所有线程都很忙时，所有传入的任务，比如散列计算或`fs`调用，都将被排队。

所以 Node.js 的第二条黄金法则是“明智地阻塞工人池”。这里有一个简短的建议列表，可以帮助你遵循这个规则:

*   避免在工作池中发生长时间运行的任务。例如，比起用`fs.readFile()`读取整个文件，我更喜欢基于流的 API。
*   如果可能，考虑对 CPU 密集型任务进行分区。
*   再次，明智地选择第三方库。

# 规则# 0——一个规则来统治所有人

现在，作为总结，我们可以制定一个编写高性能 Node.js 服务器端应用程序的经验法则。这条经验法则是“如果在任何给定时间为每个请求所做的工作足够小，Node.js 就很快”。此规则涵盖事件循环和工作池。

# 进一步阅读

作为进一步的阅读，我建议你阅读以下内容:

*   来自节点团队的指南，提供了更多模式来帮助您避免阻塞事件循环和工作池:[https://nodejs . org/en/docs/guides/don-block-the-Event-Loop/](https://nodejs.org/en/docs/guides/dont-block-the-event-loop/)
*   对于那些想要真正深入了解 Node.js 内部工作原理的人来说，这是一系列精彩的文章:[https://blog . insiderattack . net/event-loop-and-the-big-picture-nodejs-event-loop-part-1-1cb 67 a 182810](https://blog.insiderattack.net/event-loop-and-the-big-picture-nodejs-event-loop-part-1-1cb67a182810)