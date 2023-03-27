# 在 Express 中使用异步路由

> 原文：<https://itnext.io/using-async-routes-with-express-bcde8ead1de8?source=collection_archive---------1----------------------->

![](img/49169b6cd83215432daa13ec7b143ca4.png)

现在我该去哪里？(作者照片，摄于瑞士曼尼利切附近)

一段时间以来，`NodeJS`已经支持了`async` / `await`语法，允许你避免`Promises`和`Generators`的许多问题。然而，开箱即用的`[Express](https://expressjs.com)`并不能很好地处理异步路由控制器。错误处理尤其成问题。然而，通过一些简单的函数组合，实际上很容易解决这个问题。

# 一个例子

假设您有一个需要调用和`await`一个名为`someAsync`的外部异步助手函数的路由。它获取该函数的结果，并通过`res.json`返回，如下所示:

```
const someAsync = require('./helpers/someAsync')module.exports = async (req, res) => {
  const result = await someAsync(req.body)
  res.json(result)
}
```

这将作为一个`Express`路由控制器工作得很好，但是如果在`someAsync`中有什么东西失败了，你永远也不会知道，因为没有什么可以捕捉错误。当然，你可以在你的路由控制器中添加`try` / `catch`块，但是这很快就会变得冗长和烦人。

一个简单的解决方案是使用函数组合来包装异步路由:

```
const asyncRoute = route => (req, res, next = console.error) =>
  Promise.resolve(route(req, res)).catch(next)
```

然后，您可以按如下方式使用它:

```
const someAsync = require('./helpers/someAsync')const myRoute = async (req, res) => {
  const result = await someAsync(req.body)
  res.json(result)
}module.exports = **asyncRoute**(myRoute)
```

所以现在，如果有错误，包装器将`catch`它并将错误传递给`next`函数。默认情况下，该函数只是`console.error`，但是您可以使用自己的自定义错误处理程序来覆盖它。

# 测试您的异步路由

使用`[sinon](https://sinonjs.org)`和`[proxyquire](https://github.com/thlorenz/proxyquire)`可以如下测试路线:

```
const { expect } = require('chai')
const sinon = require('sinon')
const proxyquire = require('proxyquire')describe('src/routes/myRoute', () => {
  const mockSomeAsync = sinon.stub() const myRoute = proxyquire('../../src/routes/myRoute', {
    './helpers/someAsync': mockSomeAsync
  } const req = { body: 'some body' }
  const res = { json: sinon.stub() }
  const next = sinon.spy() const resetStubs = () => {
    res.json.resetHistory()
    next.resetHistory()
  } context('no errors', () => {
    const result = 'some result' before(async () => {
      mockSomeAsync.resolves(result)
      await myRoute(req, res, next)
    }) after(resetStubs) it('called someAsync with the right data', () => {
      expect(mockSomeAsync).to.have.been.calledWith(req.body)
    }) it('called res.json with the right data', () => {
      expect(res.json).to.have.been.calledWith(result)
    }) it("didn't call next", () => {
      expect(next).not.to.have.been.called
    })
  }) context('has errors', () => {
    const error = 'some error' before(async () => {
      mockSomeAsync.rejects(error)
      await myRoute(req, res, next)
    }) after(resetStubs) it('called someAsync with the right data', () => {
      expect(mockSomeAsync).to.have.been.calledWith(req.body)
    }) it("didn't call res.json", () => {
      expect(res.json).not.to.have.been.called
    }) it('called next with the error', () => {
      expect(next).to.have.been.calledWith(error)
    })
  })
})
```

# 密码

我已经将这些代码打包到一个 npm 库中。要在您的代码中使用它，只需运行:

```
npm i route-async
```

然后在您的代码中:

```
const asyncRoute = require('route-async')const yourRouteHere = async (req, res) => {
  // ...
}
module.exports = asyncRoute(yourRouteHere)
```

# 链接

*   `[route-async](https://www.npmjs.com/package/route-async)` [上 npm](https://www.npmjs.com/package/route-async)
*   `[route-async](https://github.com/davesag/route-async)` [来源于 GitHub](https://github.com/davesag/route-async)
*   [expressjs](https://expressjs.com)
*   [摩卡酒](https://mochajs.org)
*   [柴吉斯](http://www.chaijs.com)
*   sinonjs
*   [代理查询](https://github.com/thlorenz/proxyquire)

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。