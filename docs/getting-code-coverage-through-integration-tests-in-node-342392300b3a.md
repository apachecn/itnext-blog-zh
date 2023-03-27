# 通过节点中的集成测试获得代码覆盖率

> 原文：<https://itnext.io/getting-code-coverage-through-integration-tests-in-node-342392300b3a?source=collection_archive---------1----------------------->

大约两年前，我开始了一个项目，由于时间限制，我必须快速开发。该项目本身从一个由文件数据库支持的 CLI 工具变成了一个具有 API 支持的后端但仍然是文件数据库的 web 应用程序。在某个时候，我决定将数据转移到持久存储中，这也是分两步进行的，首先是 mlab 上的 MongoDB，然后转移到 MongoDB Atlas。

![](img/0406f2d551aac87ca42fc07c3c6f113a.png)

我知道我将很快面临技术债务——特别是在那些重写期间——但是只要我不确定这个项目是否有潜力公开发布，我就不能在编写测试上花费太多时间。

它现在在[qzmkr.com](https://www.qzmkr.com)直播，可以根据邀请为公众或封闭观众进行多项选择测试。

## 测试策略

如前所述，我看到了 npm 的本杰明·科的 [c8](https://github.com/bcoe/c8) 工具。他的工具使得利用设置 NODE_V8_COVERAGE 路径和直接从 V8 输出代码覆盖信息以及添加伊斯坦布尔报告器支持成为可能。无需特殊的伊斯坦布尔/纽约或其他覆盖工具，就可以针对您的节点服务运行一个测试套件，并获得详细的覆盖数据。
我已经维护了一个 postman 集合来测试我的 ReSTful API，它也可以通过 [newman](https://npmjs.com/package/newman) 从 cli 用作测试场景。
最后剩下的部分是用 c8 启动服务器，一旦启动，运行 testsuite 并再次关闭服务器，返回 testsuite 的退出代码，以便能够在 CI 中运行它。

## **重要！**

为了使对环境变量 NODE_V8_COVERAGE 的支持生效，它至少需要节点 10.12

值得一提的是，nodejs 服务需要正常关闭，这样 V8 就可以完成覆盖报告的编写，而不会突然退出流程。

为此，我开始监听进程上的 SIGINT 和 SIGTERM，断开数据库连接，然后优雅地停止服务器。

# 操作方法

```
npm install --save-dev newman c8

# run your service (change your js accordingly)
c8 node server.js

# run your tests with newman
newman -e postman-environment.json run testsuite.postman-collection.json
```

# 结果

现在，我能够保持一定程度的稳定性，因为我让我的 postman 集合保持最新，并在 CI 中运行我的集成测试套件。对于复杂的算法和逻辑，我仍然保留我的单元测试。但是对于大多数与 API 相关的代码，我不必再去模仿这个世界，仍然可以得到有用的覆盖率输出，从而获得信心。

# 密码

示例代码可以在 [github](https://github.com/chrkaatz/coverage-test) 上找到

# 链接

[Benjamin Coe 的初始帖子](https://blog.npmjs.org/post/178487845610/rethinking-javascript-test-coverage)