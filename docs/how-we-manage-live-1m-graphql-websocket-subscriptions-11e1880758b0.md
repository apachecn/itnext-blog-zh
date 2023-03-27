# 我们如何管理实时 1M GraphQL Websocket 订阅

> 原文：<https://itnext.io/how-we-manage-live-1m-graphql-websocket-subscriptions-11e1880758b0?source=collection_archive---------0----------------------->

![](img/bc9486ef28607665bd20fd13d9caf10a.png)

GraphQL 订阅是我们环境的关键部分，因为它允许使用相同的基于模式/查询的概念，但通过多个 UI 组件进行实时 WebSocket 更新。特别是考虑到我们在[Hexometer.com](https://hexometer.com)进行 UI <> API 通信的用例，我们的一些网站分析工具可能会给出部分数据的响应，或者可能会花费大量时间。使用 WebSockets 允许我们在工具准备好响应时向 UI 发送数据。

除了 UI 便利性之外，使用基于 Queue + PubSub 的系统来处理用户请求实际上还有很多好处，因为一切都以异步方式运行，不需要增加新服务或分配更多资源来处理每个请求。然而，当然也有一些缺点，特别是管理 WebSocket 连接可能需要比标准的请求-响应周期更多的资源。

# 我们的基地设置

对我们基础设施的所有请求都通过我们的 Nginx 实例传递，然后该实例根据请求代理特定的服务。我们的 Node.js API 实例也是如此，它通过我们的 Apollo 服务器处理 GraphQL Websocket 订阅。

![](img/75aaa14776945bd9110a3a4db82a3df7.png)

基于这种结构和我们的基准，每个 WebSocket 连接平均需要大约 4Kb 的内存，这不是一个大问题，但如果你计算一下我们正在进行的活动连接和活动网站分析的数量，保持活动 WebSockets 连接会变得越来越昂贵。

WebSockets 的主要问题是资源使用呈指数级增长，很难预测如何与之匹配。有时，当连接关闭时，Node.js 不会释放内存，因为默认情况下有一个特定的超时选项。

# Apollo 服务器设置

一般来说，我们这里没有任何定制的东西，它是一个基本的 Apollo 服务器，带有一些中间件来处理连接、验证或通过认证过程。我们使用`cors`模块来保持我们的 API 端点作为子域，并从多个 UI 项目中访问它。

```
const server = new ApolloServer({
  resolvers: Resolvers,
  typeDefs,
  context: async ({ req }) => {
    // Authentication and Connection validation goes here
  },
  subscriptions: {
    keepAlive: 1000,
    onConnect: async (
      connectionParams: ConnectionParamsObject,
      websocket,
      context
    ) => {
      // Handling connection context here
    },
    onDisconnect: (websocket, context) => {
      // console.log("WS Disconnected! -> ", JSON.stringify(context));
    },
    path: '/api/ws'
  }
});const app = express();
app.use(cors());
server.applyMiddleware({ app, path: '/api/ql', cors: false });const httpServer = http.createServer(app);
server.installSubscriptionHandlers(httpServer);const port = process.env.HEXOMETER_PORT || 4000;httpServer.listen({ port }, () => {
  console.log(
    `🚀 Server ready at [http://localhost:${port}${server.graphqlPath}`](http://localhost:${port}${server.graphqlPath}`)
  );
  console.log(
    `🚀 Subscriptions ready at ws://localhost:${port}${
      server.subscriptionsPath
    }`
  );
});
```

正如您可能注意到的，我们使用一个`express`来进行一个基本的路由，并通过 Apollo 服务器进行连接，但是，如果我们需要一个外部 REST 端点(以防万一)，这只是为了进行一些路由。否则，我们在所有应用中只使用基于 GraphQL 的查询、变异和订阅。即使我们需要从其他服务向我们的主 API 发出请求，我们也是用 GraphQL 查询体发出基本的 HTTP 请求，遵循规则就是这么简单:)

# GraphQL 订阅

GraphQL 订阅背后的主要概念是发送基于特定类型的订阅请求，并保持无限异步迭代器活动，这允许在我们需要时从 API 端点发送回 UI 消息。为了实现所有这些过程，我们使用了定制的 PubSub 库，与 Apollo Server 默认提供的非常相似。

```
import { PubSub } from 'apollo-server-express';
....
export const pubsub = new PubSub();
....async subscribeResolver(parent: Object, args: Object, context: GrqphqlContext) {
  return pubsub.asyncIterator(topic);
}....// After some specific events, we can publish to that asyncIterator
// Which will send Websocket message to UI and will try to get another async iterator message
pubsub.publish(topic, data);
```

这个过程很酷的一点是，对于 GraphQL 订阅，您必须拥有一个异步迭代器来获取 UI 上发布的消息，否则这与拥有一个基本查询类型是一样的。

您可以将每个 PubSub 主题想象成一个队列，它以“先进先出”的概念运行，并且实际上是由所有客户共享的。起初，我们为每个订阅创建了新的 PubSub 对象，但它最终占用了大量资源，因为每个主题都分配了特定的内存大小，只有当您手动处理连接关闭和清理特定主题队列时，它才会释放内存。

我们基本上包装了默认的 PubSub 类，现在，只要 Websocket 连接关闭，异步迭代器停止迭代，它就会自动销毁自己。最酷的部分是，我们现在在整个服务中制作单个 PubSub 对象，但对于每个客户，我们都制作一个单独的主题，这允许我们跨多个模块传递全局`pubsub`对象并将对象发送到 UI，甚至不需要特定模块的功能。

# 1M WebSocket 问题

如果你能想象一下拥有 100 万个实时 WebSocket 连接意味着什么，你会发现这需要大量资源，这很难自动扩展，尽管我们所有的基础设施都在 Google Cloud Kubernetes 上。

实时 TCP 连接的核心概念是它需要

*   打开文件/套接字
*   为 IO 缓冲区分配的内存
*   管理数据流事件的 CPU 周期

所有这些都受到操作系统本身的极大限制。当然，这也取决于您的应用程序性能，但是如果您拥有我们所拥有的一切(基本的、默认的方式)，那么您的主要限制或性能问题实际上来自这 3 个限制。

例如，如果您将 Apache Server 作为主要的 Websocket 处理程序，而不是 Nginx，您可能会非常头疼，因为 Apache 不是异步的，并且会占用大量资源来保持单个连接的活动。因此，选择负载平衡软件至关重要，实际上没有负载平衡器是不行的，因为在任何情况下，您都希望将 API 实例从一个扩展到多个，这就是微服务的本质。

与其他软件负载平衡器相比，Nginx 实际上在资源使用方面要便宜得多，但是，我们必须对其他软件进行一些特定的调整，以使它在 WebSocket 连接方面更加可靠。

```
worker_processes auto;
events {
    use epoll;  # Using OS's base Event Loop for IO operations
    worker_connections 1024;
    multi_accept on;
}....
	location '/ws' {
     ......
     proxy_http_version 1.1;
     proxy_set_header Connection $connection_upgrade;
     proxy_set_header Upgrade $http_upgrade;
     proxy_read_timeout 999950s;  # Just a large number to wait for data
  }
....
```

这里最重要的部分是添加`use epoll`配置，这意味着 Nginx 将维护基本 Linux Epoll 事件循环系统上的所有连接，为所有硬件驱动程序管理整个 IO。这两个字实际上使我们的 CPU 和内存使用率下降了 4-5 倍，因为此后 Nginx 不再为特定套接字维护数据缓冲，而是由 OS Epoll 自己完成，然后它使用基本的事件原理将缓冲区和套接字状态转移到 Nginx。

# 为什么 WebSockets 没有那么流行？

我认为大多数公司仍然使用基本 HTTP 请求的原因是他们认为很难维护。但是使用 GraphQL 订阅，就完全抽象掉了，开发者其实感觉不到任何区别，因为是同样的 GraphQL 类型，同样的解析器。唯一的区别是，开发人员应该理解拥有 PubSub 系统的基本概念，以及所有消息/数据流通过队列原则(先进先出)。将 GraphQL 订阅 PubSub 连接到 Redis 或任何其他队列(DB)系统实际上是可能的，但这取决于特定的用例。

我们继续将越来越多的内容从基本的查询转移到订阅，为我们的用户提供更多的交互式 UI，但我同意它不会完全取代标准的请求响应。

通过分享和鼓掌来帮助制作更多这种类型的内容👏👏