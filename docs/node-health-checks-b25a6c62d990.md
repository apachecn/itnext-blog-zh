# 节点健康检查

> 原文：<https://itnext.io/node-health-checks-b25a6c62d990?source=collection_archive---------2----------------------->

![](img/ebf7561e1c12c8c53aa9af6d0af17155.png)

Cloud Foundry (CF)有一个*健康检查*的概念。这些检查有几种形式，但首选的方法是 HTTP(而不是基于端口或基于进程的检查)。应用程序可以公开一个`/healthcheck`端点，由[云控制器](https://docs.cloudfoundry.org/concepts/architecture/cloud-controller.html)进行轮询。如果应用程序变得不健康(由非 200 HTTP 状态代码指示)，则会尝试重新启动。

有一个*健康检查超时*，我喜欢把它称为*启动超时*。在应用程序的初始启动期间，运行状况检查过程将等待这么长时间(每隔几秒钟轮询一次)以使应用程序变得运行状况良好。该值是可配置的(在 [Pivotal Cloud Foundry](https://pivotal.io/platform) 中为 60-600 秒)，支持具有更密集设置任务的应用。

![](img/0cbeb4239a7bfb154092b8c30ca59a8d.png)

还有一个*调用超时*，它控制运行状况检查器等待正在运行的应用程序响应的时间(在初始启动完成后，应用程序已经提供了第一个运行状况良好的响应)。历史上，这被硬编码为一秒。

这使健康检查不会使系统陷入困境，但也意味着需要对许多依赖项进行密集健康检查的应用程序经常会超时，并被判断为不健康(重启风暴！).

[提供了一个补丁，使这成为可配置的](https://github.com/cloudfoundry/cloud_controller_ng/issues/1055)，但当我第一次听说它时，我回想起以前的生活…一般的问题并不具体。我记得当 Nagios 和 Cacti 处于领先地位时，我曾解决过类似的问题(轮询超时)。

这不是我发明的模式，比我聪明得多的同事指出，解决方案是*从测试执行中分离响应*，而不是更少(减少覆盖率)或更不准确的测试(端口或过程)。

# 代码时间！

[克隆存储库以遵循](https://github.com/deadlysyn/node-healthcheck) …

即使有可配置的超时，解耦也是好的…所以我想创建一个简单的[节点](https://nodejs.org/)应用，利用[承诺](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)和[异步/等待](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)来模拟一个*健康检查模式*，该模式被设计为与[云铸造](https://www.cloudfoundry.org/)一起工作。想法是*使用 async/await 来正确地运行和收集任意数量的长期测试的结果*，同时尽可能地保持您的端点的响应性和准确性。

如果您没有完整地看过示例应用程序，那么这些示例会更有意义，我创建了一个全局对象，我们可以用它来存储测试结果。我们还设置了一个默认的状态代码，稍后您会看到更多。

为了模拟潜在的缓慢的应用程序依赖测试，我使用了`setTimeout`来引入延迟。想象一下，一个过载的数据库或您最喜欢的上游 API 在高峰时段受到冲击。

在示例应用程序中，我有几个这样的函数，而一个真正的应用程序可能有很多……所以下一步是将测试套件包装在一个`async`函数中，该函数调用每个测试并`await`响应。在简单的情况下，它检查我们的模拟结果是否失败，并相应地更新状态代码。

最后一部分是正确构建健康检查端点本身…我们希望启动`testRunner`，但不是作为中间件，这会导致响应阻塞。

当健康检查请求进来时，异步函数将运行并做所有需要的检查，随着进程更新`testResults`。如果出现问题，后续请求将收到一个非 200 的状态代码。此外，每次测试都会返回一条潜在有用的消息和时间戳…可能还集成了第三方监控。

如果你的机器上运行着 [Docker](https://www.docker.com/) ，你可以简单的[克隆这个项目](https://github.com/deadlysyn/node-healthcheck)，然后运行`make build; make run`在`[http://localhost:3000/healthcheck](http://localhost:3000/healthcheck.)` [上获得快速监听。](http://localhost:3000/healthcheck.)

端点将立即使用默认响应代码(200)进行响应。这保持了健康检查过程的愉快。如果您观察 stdout，您会看到几秒钟后 *db 测试运行*，几秒钟后*网络测试运行*。刷新端点将显示测试结果。您可以将`message`设置为`OK`以外的其他值来模拟故障。任何测试失败都会导致 500 状态代码。

```
❯ http localhost:3000/healthcheck
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 14
Content-Type: application/json; charset=utf-8
Date: Sun, 20 Jan 2019 18:09:30 GMT
ETag: W/"e-QlsUp1vTYvBgYHrHCBYe2n/q268"
X-Powered-By: Express{
    "status": 200
}❯ http localhost:3000/healthcheck
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 121
Content-Type: application/json; charset=utf-8
Date: Sun, 20 Jan 2019 18:06:56 GMT
ETag: W/"79-topudR8vULOkkpcpIVCdvk+S1nQ"
X-Powered-By: Express{
    "database": {
        "message": "OK",
        "timestamp": 1548007610802
    },
    "network": {
        "message": "OK",
        "timestamp": 1548007612803
    },
    "status": 200
}❯ http localhost:3000/healthcheck
HTTP/1.1 500 Internal Server Error
Connection: keep-alive
Content-Length: 125
Content-Type: application/json; charset=utf-8
Date: Sun, 20 Jan 2019 18:09:43 GMT
ETag: W/"7d-NbOObgl/2uT9jLi9gSpqy8qDyWE"
X-Powered-By: Express{
    "database": {
        "message": "OK",
        "timestamp": "1548007773994"
    },
    "network": {
        "message": "FAIL",
        "timestamp": 1548007775999
    },
    "status": 500
}
```

希望这能给你一些方法来为你的节点应用程序创建更好的健康检查！

![](img/9f8b767a65d5ec095ac35f235e597ab1.png)