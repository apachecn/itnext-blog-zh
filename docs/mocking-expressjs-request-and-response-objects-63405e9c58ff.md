# 模仿 ExpressJS 请求和响应对象

> 原文：<https://itnext.io/mocking-expressjs-request-and-response-objects-63405e9c58ff?source=collection_archive---------1----------------------->

![](img/79157eb9d56b247117da4677767b7703.png)

新德里街道上的请求和响应(图片由作者的照片生成)

当单元测试`[ExpressJS](https://expressjs.com)`路由控制器和中间件时，你需要创建虚拟的`req`和`res`对象作为参数传入。`req` param 需要一堆属性，最典型的是`body`、`query`和`params`对象，以及用于访问头的`get`函数。`res`参数通常需要`end`、`json`、`send`和`status`函数，以及您的函数使用的任何其他东西。在`status`函数的情况下，你的假`res`对象需要确保它的`status`函数是可链接的。

为了简单起见，我发布了`[mock-req-res](https://www.npmjs.com/package/mock-req-res)` [，](https://www.npmjs.com/package/mock-req-res)，它们公开了`mockRequest`和`mockResponse`函数，您的单元测试可以使用它们来生成一致且有用的模拟`req`和`res`对象。

# 模拟请求

```
const req = mockRequest(options)
```

`options`可以是你想要的任何东西。默认值为:

```
body: {},
cookies: {},
query: {},
params: {}
```

`get`功能作为`[sinon](https://sinonjs.org)`T24 提供。

# 模拟回应

```
const res = mockResponse(options)
```

默认的`options`有:

```
clearCookie: spy(),
cookie: spy(),
download: spy(),
end: spy(),
format: spy(),
json: spy(),
jsonp: spy(),
redirect: spy(),
render: spy(),
send: spy(),
sendFile: spy(),
sendStatus: spy(),
set: spy(),
type: spy()
```

在这种情况下，大多数默认函数都被设置为`[sinon](https://sinonjs.org)` `spies`，所以您的测试可以简单地使用`expect(res.end).to.have.been.calledOnce`。就像`request`对象一样，`get`函数被存根化，允许您的测试控制`get`返回什么。

`status`和`vary`函数都被存根化并设置为返回`response`对象，允许链接。

# 选择

`mockRequest`和`mockResponse`函数都返回包含最常用默认值的对象。可以向它们传递一个`options`对象，该对象的根级别值只是与那些默认值合并，任何重复的值都会覆盖默认值。因此，如果你需要一个`request`对象的特定属性，那么只需将它作为`option`传入即可。

# 用法示例

假设你有一个`ExpressJS`路径控制器功能如下:

```
const save = require('../../utils/saveThing') // assume this exists.const createThing = (req, res) => {
  const { name, description } = req.body
  if (!name || !description) throw new Error('Invalid Properties')
  const saved = save({ name, description })
  res.json(saved)
}
```

要对此进行单元测试，您可以使用`[Mocha](https://mochajs.org)`、`[Chai](http://www.chaijs.com)`、`[Sinon](https://sinonjs.org)`和`[Proxyquire](https://github.com/thlorenz/proxyquire)`，如下所示:

```
const { expect } = require('chai')
const { stub, match } = require('sinon')
const { mockRequest, mockResponse } = require('mock-req-res')
const proxyquire = require('proxyquire')describe('src/api/things/createThing', () => {
  const mockSave = stub() const createThing = proxyquire('../../src/api/things/createThing', {
    '../../utils/saveThing': mockSave
  }) const res = mockResponse() const resetStubs = () => {
    mockSave.resetHistory()
    res.json.resetHistory()
  } context('happy path', () => {
    const name = 'some name'
    const description = 'some description' const req = mockRequest({ body: { name, description }})
    const expected = { name, description, id: 1 }

    before(() => {
      save.returns(expected)
      createThing(req, res)
    }) after(resetStubs) it('called save with the right data', () => {
      expect(save).to.have.been.calledWith(match({
        name,
        description
      }))
    }) it('called res.json with the right data', () => {
      expect(res.json).to.have.been.calledWith(match(expected))
    })
  }) // and also test the various unhappy path scenarios.
})
```

# 摘要

当您需要测试的函数数量增加时，或者您发现自己正在编写大量的微服务时，使用标准库来模仿`request`和`response`对象可以为您节省大量令人厌烦的重复工作。

`mock-req-res`库尽可能的小，只需要`sinon`作为对等依赖。

# 链接

*   `[ExpressJS](https://expressjs.com)`:用于`NodeJS`的标准 web 服务器框架，
*   `[Sinon](https://sinonjs.org)`:强大的嘲讽库，
*   让你的测试富有表现力，
*   `[Proxyquire](https://github.com/thlorenz/proxyquire)`:将你自己的模拟对象注入到你正在测试的模块中，
*   `[Mocha](https://mochajs.org)`:强大的测试自动化库，
*   NPM 的图书馆，还有
*   GitHub 中的`[mock-req-res](https://github.com/davesag/mock-req-res)`源代码。

—

像这样但不是订户？您可以通过 davesag.medium.com 的[加入来支持作者。](https://davesag.medium.com/membership)