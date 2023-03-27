# 我如何使用变异测试来改进我的代码

> 原文：<https://itnext.io/how-i-used-mutation-testing-to-improve-my-code-71c7fcf5174d?source=collection_archive---------2----------------------->

![](img/9f3b0322af03af0f2ac3d8b7b7a9bdc9.png)

曼谷一夜(图片由作者提供)

## 让我的测试更加诚实

不久前，我写了一个名为`[amqp-delegate](https://github.com/davesag/amqp-delegate)`的小工具，它使用标准的`amqplib`库，允许通过`aqmp`消息总线(如`[Rabbit MQ](https://www.rabbitmq.com)`)轻松创建和调用远程工作者。

我写了一篇关于这个的文章，叫做“使用 NodeJS 和 AMQP 委派工作”。

我写这篇文章的时候正在海滩上，感觉很懒。为了在我的回购协议上获得一个漂亮的绿色`100%`覆盖徽章，我欺骗并使用`/* istanbul ignore next */`来完全忽略我的工作委托人的`[invoke](https://github.com/davesag/amqp-delegate/blob/1.0.3/src/makeDelegator.js#L50)` [功能](https://github.com/davesag/amqp-delegate/blob/1.0.3/src/makeDelegator.js#L50)。

在我的测试中，我添加了一点`TODO`注意:

```
// TODO: work out how to test the channel.consume callback
```

然后[我](https://github.com/davesag/amqp-delegate/blob/1.0.3/test/unit/makeDelegator.test.js#L112) `[skipped](https://github.com/davesag/amqp-delegate/blob/1.0.3/test/unit/makeDelegator.test.js#L112)` [测试](https://github.com/davesag/amqp-delegate/blob/1.0.3/test/unit/makeDelegator.test.js#L112)。

我想我已经用我的*集成*测试测试了这段代码，所以纠结于*单元*测试代码覆盖率只是浪费时间。我的代码有效，因此是好代码。

然后我读到了一个名为 [Stryker Mutator](https://stryker-mutator.io) 的突变测试库，我想我应该把它添加到我的库中，看看它能为我做些什么。

# 什么是突变测试。

突变测试是测试你的测试的一种方式。如上所述，通过跳过一些代码来欺骗你的测试覆盖报告是很容易的，但是有时你的单元测试并没有真正完成它们的工作也是不明显的。

变异测试以巧妙的方式破坏你的代码，将`false`改为`true`，更改`strings`和`numbers`的值，将`plus`改为`minus`，诸如此类，然后对你代码的每个变异一次又一次地运行你的测试。如果尽管对代码进行了更改，测试仍然通过，那么您的测试就被认为是失败的。

在一个理想的世界里*,你的测试没有一个能在你的代码被变异后存活下来。*

# 尝试一下

运行我的突变测试显示，我的终端中出现了大片红色，因为我告诉`[Istanbul](https://istanbul.js.org)`忽略的所有代码都被标记出来，还有一堆我认为测试良好的其他代码。

为了给出一个更详细的例子，下面是我上面提到的`[makeDelegator](https://github.com/davesag/amqp-delegate/blob/1.0.3/src/makeDelegator.js#L50)` [函数](https://github.com/davesag/amqp-delegate/blob/1.0.3/src/makeDelegator.js#L50)的代码。

具体来说，下面是`invoke`函数的样子。您可以看到为什么这很难进行单元测试。

```
const invoke = async (name, ...params) => {
  if (!channel) throw new Error(QUEUE_NOT_STARTED)
  const queue = await channel.assertQueue('', { exclusive: true })
  const buffer = Buffer.from(JSON.stringify(params))
  const correlationId = v4()
  const replyTo = queue.queue return new Promise((resolve, reject) => {
    channel.consume(
      replyTo,
      message => {
        if (message.properties.correlationId === correlationId) {
          try {
            const result = JSON.parse(message.content.toString())
            return resolve(result)
          } catch (err) {
            return reject(err)
          }
        } else return reject(WRONG_CORRELATION_ID)
      },
      { noAck: true }
    ) channel.sendToQueue(name, buffer, { correlationId, replyTo })
  })
}
```

该函数的工作方式是注册一个消息消费者，然后将数据发送到消息队列。消息消费者等待，直到得到正确的`correlationId`响应，然后才执行`resolve`或`reject`操作`promise`。

对`resolve`或`reject`的调用深埋在响应消息处理器中，使得单元测试非常困难。作为一个完美主义者，我不得不试一试。

## 重构

第一步是提取响应消息处理程序并单独测试。

处理函数需要访问总体承诺的`resolve`和`reject`函数以及`correlationId`函数，以便与`message`自己的`correlationId`进行比较。我创建了下面的 *curried* 实用函数:

`[src/utils/messageCorrelator.js](https://github.com/davesag/amqp-delegate/blob/develop/src/utils/messageCorrelator.js)`

```
const { WRONG_CORRELATION_ID } = require('../errors')const messageCorrelator = (correlationId, resolve, reject) =>
  message => {
    if (message.properties.correlationId === correlationId) {
      try {
        const result = JSON.parse(message.content.toString())
        return resolve(result)
      } catch (err) {
        return reject(err)
      }
    } return reject(WRONG_CORRELATION_ID)
  }module.exports = messageCorrelator
```

这很容易测试。只需传入`resolve`和`reject`函数的存根，并设置`correlationIds`匹配或不匹配的场景。此外，为了完整性，当响应消息的`content`不可解析为`JSON`时，加入一个测试。

```
const { expect } = require('chai')
const { stub, resetHistory } = require('sinon')const messageCorrelator =
  require('../../../src/utils/messageCorrelator')const { WRONG_CORRELATION_ID } = require('../../../src/errors')describe('utils/messageCorrelator', () => {
  const RESOLVED = 'resolved'
  const REJECTED = 'rejected'
  const resolve = stub().returns(RESOLVED)
  const reject = stub().returns(REJECTED)
  const correlationId = '123456'
  const content = { test: 'data', is: 'good data' } const correlate =
    messageCorrelator(correlationId, resolve, reject)
  let result context('given matching correlationId', () => {
    context('given unparsable message content', () => {
      const message = {
        properties: {
          correlationId
        },
        content: 'junk'
      } before(() => {
        result = correlate(message)
      }) after(resetHistory) it('invoked reject with a SyntaxError', () => {
        expect(reject).to.have.been.called
        const err = reject.firstCall.args[0]
        expect(err).to.be.instanceof(SyntaxError)
      }) it('returned the evaluated rejection', () => {
        expect(result).to.equal(REJECTED)
      })
    }) context('given parsable message content', () => {
      const message = {
        properties: {
          correlationId
        },
        content: JSON.stringify(content)
      } before(() => {
        result = correlate(message)
      }) after(resetHistory) it('invoked resolve with parsed content', () => {
        expect(resolve).to.have.been.calledWith(content)
      }) it('returned the evaluated rejection', () => {
        expect(result).to.equal(RESOLVED)
      })
    })
  }) context('given non-matching correlationId', () => {
    const message = {
      properties: {
        correlationId: 'some-other-id'
      },
      content: JSON.stringify(content)
    } before(() => {
      result = correlate(message)
    }) after(resetHistory) it('invoked reject with WRONG_CORRELATION_ID', () => {
      expect(reject).to.have.been.calledWith(WRONG_CORRELATION_ID)
    }) it('returned the evaluated rejection', () => {
      expect(result).to.equal(REJECTED)
    })
  })
})
```

现在我有了这个实用程序，我可以通过制作一个简单的 curried `[invoker](https://github.com/davesag/amqp-delegate/blob/develop/src/utils/invoker.js)` [函数](https://github.com/davesag/amqp-delegate/blob/develop/src/utils/invoker.js)从 invoke 函数中提取调用逻辑，如下所示:

```
const messageCorrelator = require('./messageCorrelator')const invoker = (correlationId, channel, replyTo) =>
  async (name, params) =>
    new Promise((resolve, reject) => {
      channel.consume(
        replyTo,
        messageCorrelator(correlationId, resolve, reject),
        { noAck: true }
      ) channel.sendToQueue(
        name,
        Buffer.from(JSON.stringify(params)),
        {
          correlationId,
          replyTo
        }
      )
    })module.exports = invoker
```

这使用了`messageCorrelator`，非常容易测试。这个函数中根本没有分支逻辑。该函数不关心得到的承诺是被解决还是被拒绝，所以测试非常简单。

在许多测试中，我使用测试工具创建一个假的`[channel](https://github.com/davesag/amqp-delegate/blob/develop/test/unit/fakes.js)`来代替真的`amqp channel`。

```
const fakeChannel = () => ({
  assertExchange: stub(),
  publish: stub(),
  close: stub(),
  assertQueue: stub(),
  purgeQueue: stub(),
  bindQueue: stub(),
  prefetch: stub(),
  consume: stub(),
  ack: stub(),
  nack: stub(),
  sendToQueue: stub()
})
```

我使用`[proxyquire](https://github.com/thlorenz/proxyquire)`来剔除`messageCorrelator`，因为我已经单独测试过了，所以[测试](https://github.com/davesag/amqp-delegate/blob/develop/test/unit/utils/invoker.test.js)看起来像这样:

```
const { expect } = require('chai')
const { stub, match } = require('sinon')
const proxyquire = require('proxyquire')const { fakeChannel } = require('../fakes')describe('utils/invoker', () => {
  const channel = fakeChannel()
  const messageCorrelator = stub()
  const correlationId = '12345'
  const name = 'some name'
  const param = 'some param'
  const replyTo = 'some replyTo address' const invoker = proxyquire('../../../src/utils/invoker', {
    './messageCorrelator': messageCorrelator
  }) const invocation = invoker(correlationId, channel, replyTo)
  const message = 'a message' before(() => {
    messageCorrelator.returns(message)
    invocation(name, [param])
  }) it('called the messageCorrelator with the right values', () => {
    expect(messageCorrelator).to.have.been.calledWith(
      correlationId,
      match.func,
      match.func
    )
  }) it('called channel.consume with the right values', () => {
    expect(channel.consume).to.have.been.calledWith(
      replyTo,
      message,
      { noAck: true }
    )
  }) it('called channel.sendToQueue with the right values', () => {
    expect(channel.sendToQueue).to.have.been.calledWith(
      name,
      Buffer.from(JSON.stringify([param])),
      { correlationId, replyTo }
    )
  })
})
```

然后我可以重构原始的`invoke`函数，使其更加简单:

```
const invoke = async (name, ...params) => {
  if (!channel) throw new Error(QUEUE_NOT_STARTED)
  const queue = await channel.assertQueue('', { exclusive: true })
  const correlationId = v4()
  const replyTo = queue.queue
  const invocation = invoker(correlationId, channel, replyTo)
  return invocation(name, params)
}
```

这更容易测试。

```
describe('invoke', () => {
  const invocation = stub() context('before the delegator was started', () => {
    before(() => {
      delegator = makeDelegator({ exchange })
    }) after(resetHistory) it('throws QUEUE_NOT_STARTED', () =>
     expect(delegator.invoke())
       .to.be.rejectedWith(QUEUE_NOT_STARTED))
  }) context('after the delegator was started', () => {
    const name = 'some name'
    const param = 'some param' before(async () => {
      queue = fakeQueue()
      delegator = makeDelegator({ exchange })
      channel = fakeChannel()
      connection = fakeConnection()
      channel.assertQueue.resolves(queue)
      connection.createChannel.resolves(channel)
      amqplib.connect.resolves(connection)
      await delegator.start()
      invocation.resolves()
      invoker.returns(invocation)
      await delegator.invoke(name, param)
    }) after(resetHistory) it('called channel.assertQueue with the right params', () => {
      expect(channel.assertQueue).to.have.been.calledWith('', {
        exclusive: true
      })
    }) it('called the invoker with the right params', () => {
      expect(invoker).to.have.been.calledWith(
        correlationId,
        channel,
        queue.queue
      )
    }) it('called invocation with the right params', () => {
      expect(invocation).to.have.been.calledWith(name, [param])
    })
  })
})
```

然后，我将同样的分解应用到代码的其他部分，这些部分以前很难测试。

# 结论

通过增加突变测试，我被迫返回并使我的代码更易于测试，结果我使它更加模块化，更容易推理。结果无疑是更好的代码，尽管它实际上并不比突变前的测试代码更好。

好处是，在过去的几个周末，我回顾了我维护的所有开源代码库，并为它们添加了变异测试，并修复了变异测试揭示的所有问题。

# 链接

*   *使用 NodeJS 和 AMQP 委托工作*’—[https://it next . io/delegateing-Work-using-NodeJS-and-amqp-4d 3c C1 f 62824](/delegating-work-using-nodejs-and-amqp-4d3cc1f62824)
*   `amqp-delegate`——[https://github.com/davesag/amqp-delegate](https://github.com/davesag/amqp-delegate)
*   `amqp-delegate`之前我加了突变测试—[https://github.com/davesag/amqp-delegate/tree/1.0.3](https://github.com/davesag/amqp-delegate/tree/1.0.3)
*   https://istanbul.js.org 的`Istanbul`
*   `Stryker Mutator`—[https://stryker-mutator . io](https://stryker-mutator.io)
*   【https://www.rabbitmq.com】——
*   `Mocha`—[https://mochajs.org](https://mochajs.org)
*   `Sinon`—[https://sinonjs.org](https://sinonjs.org)
*   `Proxyquire`——[https://github.com/thlorenz/proxyquire](https://github.com/thlorenz/proxyquire)

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。