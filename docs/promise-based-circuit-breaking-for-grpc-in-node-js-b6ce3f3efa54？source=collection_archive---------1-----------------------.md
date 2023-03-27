# Node.js 中 gRPC 的基于承诺的断路

> 原文：<https://itnext.io/promise-based-circuit-breaking-for-grpc-in-node-js-b6ce3f3efa54?source=collection_archive---------1----------------------->

![](img/8c8e5f5ed7487fea34fb7a2ca2e23dab.png)

构建微服务需要关注架构的许多方面。其中之一是可靠性，而“[断路器](http://microservices.io/patterns/reliability/circuit-breaker.html)可以帮助解决系统中的这一问题:服务有时会在处理请求时协作，并且总是存在另一个服务不可用或太慢而不可用的可能性。在等待另一个服务响应时，资源可能会被浪费，这可能会导致资源耗尽，并可能级联到整个应用程序中的其他服务。

在过去的一个项目中，我确实在 Node.js 内置的 API 网关中实现了这样的机制(它必须通过 gRPC 与各种服务进行通信)——所以我使用 [Hystrix.js](https://www.npmjs.com/package/hystrixjs) (最初在网飞用 Java 编写的库)为这样的调用编写了一个小包装器。

下面代码的存储库是这里的。

# 创建命令功能

***createCommand*** 函数设置您想要运行的 RPC 命令，并为配置设置一些可定制的默认值。

```
const CommandsFactory = require("hystrixjs").commandFactory;
const createCommand = (
  runFn,
  opts = {
    name: Symbol(runFn),
    errorThreshold: 1,
    timeout: 3000,
    concurrency: 0
  }
) =>
  CommandsFactory.getOrCreate(opts.name)
    .circuitBreakerErrorThresholdPercentage(opts.errorThreshold)
    .timeout(opts.timeout)
    .run(runFn)
    .circuitBreakerSleepWindowInMilliseconds(opts.timeout)
    .statisticalWindowLength(10000)
    .statisticalWindowNumberOfBuckets(10)
    .requestVolumeRejectionThreshold(opts.concurrency)
    .build();
```

这由***createrpcommand***函数在内部使用，该函数“承诺”该命令——这是基本库中所缺少的功能。

```
const createRpcCommand = rpcFn =>
  createCommand(
    request =>
      new Promise((resolve, reject) => {
        rpcFn(request, (error, response) => {
          if (error) reject(error);
          resolve(response);
        });
      })
  );module.exports = {
  createRpcCommand
};
```

# 使用

包装器现在可以用于现实世界的应用程序，在这些应用程序中，您需要一个基于承诺的断路器来与 Node.js 应用程序中的任何其他服务进行通信:

```
const messages = require("your-protobuf-messages");
const rpcClient = require("your-grpc-client");
const { createRpcCommand } = require("grpc-circuitbreaker");// Let's say rpcClient.get(request, (err, res) ...) is your function
// You really need .bind()!
const command = createRpcCommand(rpcClient.get.bind(rpcClient));
const request = new messages.YourMessage();command
  .execute(request)
  .then(response => {
    // do something with response
  })
  .catch(error => {
    // do something with error
  });
```

***想了解更多微服务和创新？关注@lucavallin 上*** [***推特***](https://twitter.com/lucavallin) ***和***[***Github***](https://github.com/lucavallin)***或者来*** [***Xebia 工作室***](https://www.xebia.studio/) ***和我们见面吧！***