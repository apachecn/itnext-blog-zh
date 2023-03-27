# Node.js —使用 Pact.js 进行集成测试

> 原文：<https://itnext.io/node-js-integration-testing-with-pact-js-1a2ea8aa3116?source=collection_archive---------3----------------------->

![](img/8ad6aa8bed269916e4a7daeeeba04251.png)

在上一篇文章中，我们回顾了消费者驱动的契约(CDC)如何在微服务丰富的环境中帮助集成测试。

[](https://medium.com/nmc-techblog/scalable-integration-testing-for-microservices-deployments-e03e29dd1280) [## 微服务部署的可扩展集成测试

### 许多人在微服务上操之过急，它们在今天比以往任何时候都更普遍地用于实现面向服务…

medium.com](https://medium.com/nmc-techblog/scalable-integration-testing-for-microservices-deployments-e03e29dd1280) 

# 用契约解决它

pact 是一个开源项目，它实现了消费者驱动的契约测试，并使用 Pact 基金会组织为不同的语言和平台创建和协作 Pact 框架。

一些 Pact 实施包括:

*   红宝石
*   Java 语言(一种计算机语言，尤用于创建网站)
*   节点. js

# Pact.js 简介

虽然硬币有两面，但我们将只关注 Node.js 项目的消费者测试，并回顾 Ava.js 测试运行程序和框架的集成测试。

# 0.装置

将 pact npm 软件包作为开发依赖项安装:

```
npm install *--save-dev pact*
```

# 1.创建测试文件

根据您的 Node.js 框架和目录结构，创建一个测试(spec)文件，我们将在其中编写与 pact 的集成测试。

```
*// consumer-example-test.integration.test.js*
const path = require('path')
const test = require('ava')
const pact = require('pact')
const request = require('request')
```

# 2.定义契约服务器

```
 const MOCK_SERVER_PORT = 2202

  const provider = pact({
    consumer: 'TodoApp',
    provider: 'TodoService',
    port: MOCK_SERVER_PORT,
    log: path.resolve(process.cwd(), 'logs', 'pact.log'),
    dir: path.resolve(__dirname, 'pacts'),
    logLevel: 'INFO',
    spec: 2
  })
```

当我们说我们正在编写消费者测试时，不要被*提供者*变量定义误导。这是我们定义 pact 服务器的地方，它模仿我们的提供者，并响应我们对它的 API 请求。

1.  ***消费者*** 和 ***提供者*** 仅仅是名称，以便在调试、检查测试日志时更容易使用，并在最终生成的契约中使用。
2.  Pact 将启动一个监听端口 2202 的服务，将日志写到一个执行测试的 *logs/* 目录中，并将在一个目录 *pacts/* 中创建实际的 pact 契约文件，该目录位于您创建测试文件的位置旁边。
3.  Pact 将使用它支持的最新规范版本(v2)

# 3.测试设置

```
test.before('setting up pact', async t => {
  *// this is the response you expect from your Provider*
  const EXPECTED_BODY = [{
    id: 1,
    name: 'Project 1', 
    type: '1',
    due: '2016-02-11T09:46:56.023Z',
    tasks: [
      {id: 1, name: 'Do the laundry', 'done': true},
      {id: 2, name: 'Do the dishes', 'done': false},
      {id: 3, name: 'Do the backyard', 'done': false},
      {id: 4, name: 'Do nothing', 'done': false}
    ]
  }] await provider.setup()
    .then(() => {
      provider.addInteraction({
        *// The 'state' field specifies a "Provider State"*
        state: 'i have a list of projects',
        uponReceiving: 'a request for projects',
        withRequest: {
          method: 'GET',
          path: '/projects',
          headers: {'Accept': 'application/json'},
          query: {
            projectTypes: pact.Matchers.term({
              matcher: '\\d+',
              generate: '1' })
          }
        },
        willRespondWith: {
          status: 200,
          headers: {'Content-Type': 'application/json'},
          body: EXPECTED_BODY
        }
      })
    })
})
```

在我们的测试实际运行之前，我们需要启动 Pact 服务，并为它提供我们期望的交互。

> 这是我们定义期望的地方。预期交互之间的任何不匹配都将导致测试在断言时抛出错误。

1.  *withRequest* 和 *willRespondWith* 定义了 API 消费者和提供者之间的预期交互。
2.  您会注意到，请求部分的期望还定义了一个名为 *projectTypes* 的查询参数，消费者 API 将发送该参数，我们使用 pact 库中的一个称为 matchers 的功能，通过使用一个 number regex，允许提供者灵活地实现契约。

# 4.消费者测试

```
test.cb('should generate a list of TODOs for the main screen', t => {
  t.plan(1) *// const todoApp = new TodoApp()*
  *// todoApp.getProjects() // <- this method would make the remote http call*
  *//   .then((projects) => {*
  *//     expect(projects).to.be.a('array')*
  *//     expect(projects).to.have.deep.property('projects[0].id', 1)*
  *//*
  *//     // (5) validate the interactions occurred, this will throw an error if it fails telling you what went wrong*
  *//     return provider.verify()*
  *//   })* const reqOptions = {
    url: `http://localhost:${MOCK_SERVER_PORT}/projects?projectTypes=1`,
    method: 'GET',
    json: true
  } request(reqOptions, async (error, response, body) => {
    if (error) {
      t.fail()
    } // This is the important part, where we assert expected interactions with our Pact service
    await t.notThrows(provider.verify())
    t.end()
  })})
```

这是我们实际的消费者测试，我们使用 *request* 模块向 pact 库为我们创建的模拟 API 服务发出 HTTP 请求。

关于实际的消费者测试——被注释的顶部显示了在真实世界中的实际预期使用，其中您调用了一个内部逻辑，该逻辑在幕后预期向提供者发出一个 API 调用。另一种情况是，您用 *supertest* 向您自己的消费者内部的一个内部 API 端点模拟请求，它反过来向提供者发出那个 API 调用。

我们向提供者发出 API 调用的测试的底部只是为了一个工作示例。

**最重要的是**，我们用 *provider.verify()* 断言，通过确保它不抛出错误，确实所有预期的交互都已经完成，然后结束测试。

# 5.测试拆卸

```
test.always.after(async () => {
  *// (6) write the pact file for this consumer-provider pair,*
  *// and shutdown the associated mock server.*
  *// You should do this only _once_ per Provider you are testing.*
  await provider.finalize()
})
```

在运行测试之后，您可以参考详细的 pact.log 文件来更清楚地了解交互是如何工作的，更好的是，您现在在 *pacts/* 目录中有了一个 pact 文件，您可以与您的提供者协作！

# 摘要

*   Pact.js 让我们编写消费者测试变得非常容易，我们邀请你以开源精神与我们合作:[https://github.com/pact-foundation/pact-js](https://github.com/pact-foundation/pact-js)
*   注意您的测试设置——通常您将依赖于消费者可用的数据来测试提供者，所以在您编写消费者测试时要为此做好准备。

简而言之，测试代码本身是以简单的方式编写的，但是有更好的模式可以用来编写测试，例如在测试用例本身中定义交互及其期望，而不是在全局测试套件中。

在下一篇后续文章中，我们将更深入地回顾 Pact.js 框架生命周期，以及更好测试的测试模式。