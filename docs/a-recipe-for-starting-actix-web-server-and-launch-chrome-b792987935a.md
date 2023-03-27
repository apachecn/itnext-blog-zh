# 开始使用 Rust actix-web 和启动 chrome 的方法🚀

> 原文：<https://itnext.io/a-recipe-for-starting-actix-web-server-and-launch-chrome-b792987935a?source=collection_archive---------3----------------------->

向 tsttgen 问好

![](img/eebfb3518a591cdc752444a870d8e7f2.png)

几周前，我开始在 Rust 中开发一个名为 [tsttgen](https://github.com/LironHazan/tstt_gen) 的 CLI 工具，它代表 Typescript 测试模板生成器。

简而言之，tsttgen——我最初的动机当然是提高我的 Rust 技能，但该工具背后的想法是自动生成包含测试格式的 typescript 文件中的样板文件创建部分，这些测试格式在使用 [TestCafe](https://devexpress.github.io/testcafe/) 在 typescript 中开发 e2e 时使用。

随着时间的推移，我希望继续开发 tsttgen，我认为提供 tsttgen 运行的静态报告，甚至在将来运行可以执行更多操作的 web 界面，这将是非常好的。

tsttgen struct (TGenerator)实现包含生成测试文件的方法，每个文件代表一个套件，可以包含多个测试。

TGenerator 结构还包含了关联函数`new()`,它在这篇文章中起着重要的作用，因为它负责构建带有`suites_map` 的 TGenerator，这个函数将来会作为初始共享状态传递给我们的服务器🎁。

## 为什么 [actix-web](https://github.com/actix/actix-web) ？

> " Actix web 是一个强大、实用、速度极快的 fast web 框架."

我非常喜欢[文档](https://actix.rs/docs/)，很多概念让我想起了 nodejs express 框架，比如[中间件系统](https://actix.rs/docs/middleware/)。

我知道这个项目的创建者/主要维护者决定停止维护它或做任何开源的事情，最近，我在某处读到社区继续开发它+我没有为生产开发任何东西…

有一个很好的 [repo](https://github.com/actix/examples) ，我正在寻找它来缩短学习曲线…如果我有时间，我甚至可以提供在启动服务器时启动 chrome 的例子。

# 食谱

让我们深入研究一些代码。

我喜欢保持 **main.rs** 文件不受服务器引导逻辑的影响，因此`main`函数如下所示:

tsttgen 运行的结果被传递给负责使用`results`作为初始状态启动服务器的`serve_report`函数，您很快就会看到。

## 总体项目结构📂

在 **src** 下，我创建了 **server** 文件夹，其中包含我的 actix-web 应用程序 **routes.rs** 和 **server.rs** 文件。

![](img/856c3777fc6e547acdf539a76be315e8.png)

**server.rs** 包含以下`serve_report`函数:

`serve_report`函数启动传递 tsttgen 结果的本地服务器，并在 localhost:8080 上打开 chrome，当前为一个静态 index.html 提供文本承诺，我将很快开发 web 应用程序:)

我最喜欢的一句有趣的台词是:

```
tokio::join!(server, open_web_app(server_address));
```

我使用了 [tokio](https://docs.rs/tokio/0.2.11/tokio/macro.join.html#:~:text=macro%20must%20be%20used%20inside,multiplexed%20on%20the%20current%20task.) join 宏，它接受一个异步表达式列表，并在同一任务中同时对它们求值，以便服务器启动，浏览器启动！🌐

让我们来回顾一下`start_server`功能:

如您所见，服务器将以一个包含来自 tsttgen 部分的结果的`AppState`结构启动，并使用我们的路由进行配置。🧰

**顺便说一下，这种形式的 start_server 不包括 CORS 支持。

**routes.rs** 包含`init_routes`函数，该函数目前只配置了 2 条路由，我们当前关心的那条调用了指向静态`index.html`位置的`static_files`。

🌐启动 chrome 的函数也位于 server.rs 文件中，名为`open_web_app`

正如你看到的，我在这里使用了文件系统命令，我在 mac 上运行，所以我想你必须为 windows 调整这个功能。

## 🏁就这些了，祝你货运愉快:)

我希望你喜欢我的第一个帖子，希望不是最后一个帖子。

*如果你喜欢这个帖子，请* [*星我的回购！*](https://github.com/LironHazan/tstt_gen)

干杯！

利伦。