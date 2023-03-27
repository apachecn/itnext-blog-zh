# 给你的日志更多的上下文—第 1 部分

> 原文：<https://itnext.io/give-your-logs-more-context-7b43ea6b4ae6?source=collection_archive---------5----------------------->

## 如何理解 Node.js web 应用程序日志

在构建现实世界的应用程序时，日志记录可能是最难做对的事情之一。记录太少，你会盯着屏幕试图理解它们(或从中生成的图表)。记录太多，你最终会迷失在无用信息的沼泽中，仍然不知道是否一切正常或是否有问题。

![](img/72770ba10150a7dedffb1353cd3e56d0.png)

没有适当上下文的日志看起来像…

*第 2 部分现已推出:*

[](/give-your-logs-more-context-part-2-c2c952724e04) [## 给你的日志更多的上下文—第 2 部分

### 构建上下文记录器

itnext.io](/give-your-logs-more-context-part-2-c2c952724e04) 

具体说到 Node.js/Javascript 生态系统，三大日志库——[Winston](https://github.com/winstonjs/winston)、 [Bunyan](https://github.com/trentm/node-bunyan) 和[Pino](https://github.com/pinojs/pino)——可以帮助你更好地管理上下文，这是好人做不到的。

对于本文，我将使用 Pino，但是这些想法可以很容易地复制到 Bunyan 和 Winston(或任何其他主流日志记录实用程序)上。

# 明智地使用日志级别

Pino 有 6 个默认日志级别，严重性依次为:`trace`、`debug`、`info`、`warn`、`error`和`fatal`。这些级别中的每一个都映射到从`10`到`60`的整数。这使得以后使用像`[jq](https://stedolan.github.io/jq/)`这样的工具来分析您的日志变得容易:

```
jq 'select(.level > 40)' # gets ERROR and FATAL logs
```

虽然 Pino 允许您定义自定义日志级别，但我从未见过需要它们的用例，所以我倾向于使用默认的级别。

通常，对于生产，建议忽略`trace`和`debug`级别，除非您明确尝试调试一些生产问题。

Pino 有一个[配置选项](https://github.com/pinojs/pino/blob/master/docs/api.md#options)，允许您定义生成日志条目所需的最低级别。您可以使用环境变量来避免仅仅为了更改日志级别而进行部署:

```
import pino from 'pino';const logger = pino({
  level: process.env.LOG_LEVEL || 'info'
});
```

## 经验法则

*   使用`trace`进行具有潜在高吞吐量的内部记录。
*   将`debug`用于您可能需要的最终调试会话，但是记住在您完成后移除它们。
*   将`info`用于常规应用程序工作流日志。
*   将`warn`用于预期的和频繁的错误情况(如用户输入验证)。
*   将`error`用于预期但不常见的错误情况(如网络故障、数据库超时)。
*   使用`fatal`处理意外错误情况。

# 拥抱请求 id

当我们仍然在开发应用程序，运行单元/集成测试，手动触发一些请求来查看是否一切都顺利运行时，一切都很好。产生的事件或多或少是按照可预测的顺序发生的，所以很容易理解。

然而，一旦推出生产版本，事情可能会变得非常疯狂。您的应用程序肯定会处理并发请求。如果有几个异步步骤——比如查询数据库或调用一些外部服务——每个事件的顺序将完全不可预测。在这种情况下，如果您手动检查日志(我们在某些时候都必须这样做😅)，你可能会变得非常沮丧，试图找到一个执行的线程。

一些框架——如[哈比神](https://hapijs.com/)——已经为你做好了准备。但是如果你和我一样还依赖好 ol '[' express，](https://expressjs.com/)你就要自己动手了。定义这样做的中间件非常简单:

```
function setRequestId(generateId) {
  return (req, res, next) => {
    req.id = generateId();
    next();
  };
}
```

然后使用它:

```
let i = 0;
const generateId = () => i++;
app.use(setRequestId(generateId));
```

当然，如果您重启服务器，这种幼稚的实现将不会工作，因为计数器将被重置为`0`。对于真实世界的应用程序，建议使用更健壮的 ID 生成器，比如 [uuid](https://github.com/kelektiv/node-uuid) 或者我个人选择的 [cuid](https://github.com/ericelliott/cuid) 。

如果您使用微服务架构(或者想要准备好这样做)，您可以通过允许您的服务转发和接收给定的请求 ID 来利用分布式跟踪:

```
function setDistributedRequestId(generateId) {
  return (req, res, next) => {
    const reqId = req.get('X-Request-Id') || generateId(); req.id = reqId;
    res.set('X-RequestId', reqId); next();
  };
}
```

现在我们可以创建另一个记录传入请求的中间件:

```
function logIncomingRequests(logger) {
  return (req, res, next) => {
    // without custom serializers, we must be explicit
    logger.trace({ req, requestId: req.id}, 'Incoming request');
    next();
  }
}
```

并使用它:

```
import pino from 'pino';// ...app.use(logIncommingRequests(pino()))
```

生成的日志条目如下所示:

```
{"level":30, "time":1533749413556, "pid":15377, "hostname":"henrique-pc", "msg":"Incoming request", "req":{"method":"GET", "url":"/", "headers":{"host":"localhost:4004", "user-agent":"curl/7.61.0", "accept":"*/*"}}, **"requestId":1**, "v":1}
```

到目前为止，一切顺利。我们甚至可以使用`[express-pino-logger](https://github.com/pinojs/express-pino-logger)`来进一步整合记录器和我们的 express 应用程序。这里的主要问题是请求 ID 与我们的 web 层紧密耦合。除非您在 express handlers 中定义了所有的业务逻辑——我强烈建议您[请不要](https://github.com/i0natan/nodebestpractices/blob/master/sections/projectstructre/createlayers.md)——否则您将无法访问其他层中的请求 ID 值。

> 哦，我可以将 id 存储在内存或 Redis 缓存中，然后在其他层中记录内容时检索它！！！

是啊，不错的尝试。我自己也这么想，但是没用。原因是当你有并发访问时，你无法知道你当前正在处理哪个请求。还是可以？

# 满足延续本地存储

想象一下，每个请求都是一个独立的连接执行路径(函数调用)的“线程”,当原始调用的结果返回时，这个线程就会被丢弃。

虽然 [Javascript 没有产生处理用户请求的真正线程](https://nodejs.org/en/docs/guides/blocking-vs-non-blocking/)，但是它通过注册回调来模拟这一点，当函数调用的结果可用时，回调将按正确的顺序被调用。

幸运的是，Node.js [提供了一种方式](https://nodejs.org/api/async_hooks.html)来通过这个执行“线程”拦截跳转。[延续本地存储](https://github.com/jeff-lewis/cls-hooked)(或简称为 CLS)利用这种能力在给定的“线程”内保持数据可用。

> 当您在延续本地存储中设置值时，在从原始函数调用的所有函数(同步或异步)执行完毕之前，这些值是可访问的。这包括传递给`process.nextTick`和[定时器函数](https://nodejs.org/api/timers.html) ( [setImmediate](https://nodejs.org/api/timers.html#timers_setimmediate_callback_arg) 、 [setTimeout](https://nodejs.org/api/timers.html#timers_settimeout_callback_delay_arg) 和 [setInterval](https://nodejs.org/api/timers.html#timers_setinterval_callback_delay_arg) )的回调，以及传递给调用本机函数的异步函数的回调(如从`fs`、`dns`、`zlib`和`crypto`模块导出的回调)。

![](img/a64aeab670341c852348ce21012bb13a.png)

我第一次发现 CLS 的时候…

重新定义我们的请求 ID 中间件，我们将得到类似于:

```
import { createNamespace } from 'cls-hooked';
import cuid from 'cuid';const loggerNamespace = createNamespace('logger');function clsRequestId(namespace, generateId) {
  return (req, res, next) => {
    const reqId = req.get('X-Request-Id') || generateId();
    res.set('X-RequestId', reqId); namespace.run(() => {
      namespace.set('requestId', reqId); next();
    });
  };
}app.use(clsRequestId(loggerNamespace, cuid));
```

分解一下:

*   **名称空间**大致相当于关系数据库中的表或文档存储中的集合/键空间的 CLS。要创建一个，我们只需要将它标识为一个字符串。
*   我们的“高阶”中间件`clsRequestId`现在需要两个参数:名称空间和 ID 生成器函数。
*   `namespace.run`是创建新上下文的函数，绑定到执行“线程”。
*   `namespace.set`将请求 ID 放入本地存储。
*   `next`将调用下一个快递处理员。**重要提示:**为了让这个工作正常进行，必须在`namespace.run`回调函数内部调用`next`。

现在，每当我们需要访问这个值时，我们可以使用来自`cls-hooked`的`getNamespace`:

```
import { getNamespace } from 'cls-hooked';
import pino from 'pino';const logger = pino();
loggerNamespace = getNamespace('logger');function doStuff() {
    // ...
    logger.info({ requestId: loggerNamespace.get('requestId') }, "Some message");
}
```

如果函数`doStuff`调用最终源自注册了该`clsRequestId`中间件的 express 应用程序的一个处理程序，则该值将可用。

把所有东西放在一起:

一个有点做作的例子，但说明了一点…

下面是用[auto canon](https://github.com/mcollina/autocannon)生成的示例输出:

```
$ autocannon -c 2 -a 5 -r 1 http://localhost:4004/{"level":30,"time":1533759930690,"msg":"App is running!","pid":4985,"hostname":"henrique-pc","endpoint":"[http://localhost:4000](http://localhost:4000)","v":1}
{"level":30,"time":1533759933634,"msg":"Before","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2awx0000uhwg9qh20e0b"**,"v":1}
{"level":30,"time":1533759933636,"msg":"Before","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2awz0001uhwgoyiptfxv"**,"v":1}
{"level":30,"time":1533759935531,"msg":"Middle","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2awz0001uhwgoyiptfxv"**,"v":1}
{"level":30,"time":1533759939590,"msg":"Middle","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2awx0000uhwg9qh20e0b"**,"v":1}
{"level":30,"time":1533759941222,"msg":"After","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2awz0001uhwgoyiptfxv"**,"v":1}
{"level":30,"time":1533759941228,"msg":"Before","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2grw0002uhwgzz14qyb6"**,"v":1}
{"level":30,"time":1533759943632,"msg":"Before","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2imo0003uhwgf4dutgz3"**,"v":1}
{"level":30,"time":1533759946244,"msg":"Middle","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2grw0002uhwgzz14qyb6"**,"v":1}
{"level":30,"time":1533759949490,"msg":"After","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2awx0000uhwg9qh20e0b"**,"v":1}
{"level":30,"time":1533759951621,"msg":"Middle","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2imo0003uhwgf4dutgz3"**,"v":1}
{"level":30,"time":1533759952464,"msg":"After","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2grw0002uhwgzz14qyb6"**,"v":1}
{"level":30,"time":1533759953632,"msg":"Before","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2qcg0004uhwgnmgztdr7"**,"v":1}
{"level":30,"time":1533759954665,"msg":"Middle","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2qcg0004uhwgnmgztdr7"**,"v":1}
{"level":30,"time":1533759955140,"msg":"After","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2imo0003uhwgf4dutgz3"**,"v":1}
{"level":30,"time":1533759957183,"msg":"After","pid":4985,"hostname":"henrique-pc",**"requestId":"cjkll2qcg0004uhwgnmgztdr7"**,"v":1}
```

如果您仔细观察，您会发现，尽管 logger 函数的调用顺序是非线性的，但是每个不同请求的`requestId`都是保留的。

![](img/195991e66554a99104000aa232a1b3d6.png)

这完全是胡说八道！

现在，每当您想单独查看单个请求的日志时，您可以再次使用`jq`并运行:

```
jq 'select(.requestId == "cjkll2qcg0004uhwgnmgztdr7")' <log_file>
```

输出将是:

```
{
  "level": 30,
  "time": 1533759953632,
  "msg": "Before",
  "pid": 4985,
  "hostname": "henrique-pc",
  "requestId": "cjkll2qcg0004uhwgnmgztdr7",
  "v": 1
}
{
  "level": 30,
  "time": 1533759954665,
  "msg": "Middle",
  "pid": 4985,
  "hostname": "henrique-pc",
  "requestId": "cjkll2qcg0004uhwgnmgztdr7",
  "v": 1
}
{
  "level": 30,
  "time": 1533759957183,
  "msg": "After",
  "pid": 4985,
  "hostname": "henrique-pc",
  "requestId": "cjkll2qcg0004uhwgnmgztdr7",
  "v": 1
}
```

# 进一步的改进

虽然这个故事中呈现的结构是可行的，但对于日常使用来说并不实用。像上面的示例代码一样，手动获取名称空间并检索所有需要的值是非常乏味的:

```
const namespace = getNamespace('logger');                                                 logger.info({ requestId: namespace.get('requestId') }, 'Before')
```

下次我们将围绕`pino`构建一个包装器来透明地处理所有这些。

![](img/a745dee3ac3b2ad6fcf07e7695b3b132.png)

再见！

你喜欢你刚刚读的吗？用 [tippin.me](https://tippin.me/@hbarcelos909) 给我买啤酒。