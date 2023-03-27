# 高级 NestJS 技术—第 3 部分—记录弹性搜索查询

> 原文：<https://itnext.io/advanced-nestjs-techniques-part-3-logging-elasticsearch-queries-ae118a7a9d4c?source=collection_archive---------1----------------------->

![](img/868c4c1e11a95b903f93beecb18e0c23.png)

在本文中，我将向您展示如何很好地记录 NestJS 应用程序对 Elasticsearch 进行的每个 HTTP 查询。

# 目标

本系列的第 2 部分是关于记录对外部 HTTP 服务的 HTTP 请求，无论它们是您的堆栈中的其他服务还是公共 API。

记录业务逻辑很容易(或者至少开始很容易):创建一个记录器或者从某个地方获得它(实际上我更喜欢从工厂获得一个新的实例)，然后开始记录与您的业务逻辑相关的任何东西。

> 但是我有一个更大的目标，那就是记录我的应用程序所依赖的服务的每个调用(HTTP 或其他任何东西)。

例如，使用 Postgres 和 Sequelize ORM 记录 SQL 查询很容易:

将 HTTP 查询记录到 Elasticsearch 有点棘手，因为正如这里的[所描述的](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/7.x/observability.html)，Elasticsearch 没有提供默认的记录方式，我们必须通过一个关联 ID 来关联请求和响应。所以让我们开始吧。

# 警告

如果你担心创建一个无限循环，因为你的应用程序已经向 Elasticsearch 发送日志，那么我建议你停止这样做。

一个[十二要素应用](https://12factor.net/)的 12 个核心原则之一是，将日志发送到特定存储不应该是应用本身的责任。应用程序通常应该将日志输出到 stdout / stderr，底层基础架构应该路由、增加相关元数据并在适当的位置存储这些日志。

这些原则的目标是拥有一个无需重大更改即可轻松部署到任何现代云平台的应用程序。

这是我们将通过日志记录改进的基本应用程序:

# 探测

下面是观察 Elasticsearch 客户端在不同情况下如何表现的第一步:

我们做了两件事:

*   用 UUID 替换默认生成的 ID(一个递增的数字)。如果/当我们的服务器重启时，它将避免重复使用 id
*   将此 ID 记录到控制台

我们应该已经观察到:

*   如果请求成功，事件“请求”被触发一次，事件“响应”被触发一次
*   如果请求因为 Elasticsearch 关闭而失败，客户端将重试几次，我们将得到多个“请求”事件，但只有一个“响应”事件
*   如果请求无效(“索引 _ 未找到 _ 异常”等)。)，没有重试，我们得到每个事件的单个事件

我们可以利用这一点。

# 履行

非常简单的实现。如果需要，我们可以通过查看响应主体的内容来记录更多的内容，比如返回的点击次数、碎片等。

# 结论

如果你一直在关注第一部分的[和第二部分](https://medium.com/@paztek/advanced-nestjs-techniques-part-1-custom-decorators-aa6d7f85c5b6)的[，你可能会注意到出现了两种模式:](https://medium.com/@paztek/advanced-nestjs-techniques-part-2-logging-outgoing-http-requests-3c75d47c5768)

1.  当我们需要在应用程序开始时连接一些东西，并且不再与它交互时，看起来模块生命周期的“onModuleInit()”方法是放置代码的最佳位置。
2.  为了避免 AppModule 上拥挤的“onModuleInit()”方法，我声明了像 HttpModule 或 ElasticsearchModule 这样的专用模块来处理这些初始化步骤。

我喜欢这些模式，但如果您对在哪里/如何进行这些初始化步骤有其他意见，我会更喜欢受到挑战。

像往常一样，我留给你完整资源库的链接:【https://github.com/paztek/nestjs-elasticsearch-example 

第 4 部分将讨论记录对 Redis 的调用。敬请关注。