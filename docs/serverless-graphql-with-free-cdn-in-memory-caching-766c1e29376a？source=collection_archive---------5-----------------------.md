# 具有免费 CDN 和内存缓存的无服务器 GraphQL

> 原文：<https://itnext.io/serverless-graphql-with-free-cdn-in-memory-caching-766c1e29376a?source=collection_archive---------5----------------------->

声明:这是我的第一篇关于媒体的文章，我只是想和大家分享我的经验:)所以请不要介意我的写作技巧。

![](img/2ec08de59683f851324ba050e966a215.png)

自从我发现了 GraphQL，我就爱上了它，尤其是因为它有像 apollo-client 这样的客户端库。然而，正如每种技术一样，也有权衡(见本文[https://dev-blog . Apollo data . com/graph QL-vs-rest-5d 425123 e 34 b](https://dev-blog.apollodata.com/graphql-vs-rest-5d425123e34b))。我认为最大的缺点之一是缺少 CDN & HTTP 缓存。幸运的是，apollo-client 团队提出了一些解决方案: [Apollo 引擎](https://www.apollographql.com/docs/engine/)和[自动持久化查询](https://www.apollographql.com/docs/engine/auto-persisted-queries.html)。结合 Firebase 云功能，它可以解决所有的缓存问题。

# 阿波罗发动机的缺点

当我写这篇文章时(27/06/2018)，如果你想在 AWS Lambda 或 Google/Firebase Cloud Functions 这样的无服务器框架上使用 Apollo 引擎，它并不真正适合。主要的缺点是您需要托管一个代理服务器，在基础设施中引入了一个额外的节点，从而产生了一个单点故障。

问题在于 apollo-server 如何处理请求(至少是 apollo-server-express)。目前，只要请求来自客户端，single express 中间件就会处理查询解析、解析查询，并最终将请求发送回客户端。这就只剩下预处理和后处理请求的一个选项。是的，使用代理服务器…

幸运的是，我们可以将 apollo-server-express 包分成几个 express 中间件，这样我们就可以对请求进行预处理/后处理。

# 让我们写一些代码

是时候编码了:

首先，我们需要修改 apollo-server-express 包，以便能够使用我们的定制 express 中间件:

主要区别是这几行:

```
gqlResponse => {
        res['gqlResponse'] = gqlResponse;
        next();
},
```

我们将 gqlResponse 存储在 res['gqlResponse']变量中，并转到链中的下一个中间件，而不是直接发送给客户机。

现在让我们创建负责向用户发送响应的最终中间件:

并替换默认设置(假设您之前已经有了一些 bodyparser 中间件)

```
app.use('/graphql', graphqlExpress({ schema: myGraphQLSchema }));
```

随着

```
app.use(
 '/graphql',
 // resolve the request
 graphqlExpress({
  schema,
 }),
 // send the response
 graphqlResponseHandler
);
```

完成了。

# 让我们释放野兽

这个设计最好的部分是，我们可以在 graphql 处理请求之前和服务器发送响应之前添加中间件。实现缓存、跟踪、追踪等的完美技术。

最终，端点将拥有这些中间件:

storeCache 是一个 LTU 缓存库，但也可以是一个 redis 数据库。

这里定义了所有的中间件:

## 关于缓存“POST”请求的快速解释

首先，如果你使用 post 请求，CDN 缓存将不起作用。剩下的就是服务器上的响应缓存。

诀窍是从 post 主体生成一个惟一的散列队列。这是通过调用 hashPostBody 函数来完成的。

当 post 请求到达服务器时，getFromCacheIfAny 中间件检查查询是否被缓存(第 101 行)。如果是，它将从缓存中发送响应。否则，它继续到下一个中间件 graphqlHandler。graphql 服务器解析查询并将结果设置在 res["gqlResponse"]变量中，如上所述。

然后 storeInCache 中间件开始工作，在这里我们再次缓存 post 主体，检查整个查询的 maxDuration(在这里解释:[https://github.com/apollographql/apollo-cache-control](https://github.com/apollographql/apollo-cache-control))并将结果存储在带有 maxDuration 的缓存中。

在我们用 extensionsFilter 清理完扩展之后(您也可以在这里做一些日志记录)。最后，我们将请求发送回去。

# 让我们创造一个火箭

现在是时候设置 CDN 缓存了。要使用 CDN 缓存，我们需要发送 GET 请求。总之，apollo-client 散列查询(就像我们对 POST 请求所做的那样)并发送一个散列。服务器检查散列是否在高速缓存中，如果是，它发送高速缓存数据，否则它解析查询并将结果与散列关键字(也由 apollo-client 发送)一起存储在高速缓存中。有深度的文章和设置说明可以在这里找到:[https://www . apollographql . com/docs/engine/auto-persisted-queries . html](https://www.apollographql.com/docs/engine/auto-persisted-queries.html)，直接跳过引擎部分。

## 由 Firebase 支持的免费 CDN 缓存

Firebase 是托管无服务器后端的绝佳平台。最棒的是:如果你使用 Firebase 云功能，你可以获得免费的 CDN 缓存。详情见此:[https://firebase . Google . com/docs/hosting/functions # manage _ cache _ behavior](https://firebase.google.com/docs/hosting/functions#manage_cache_behavior)

要在 Firebase 云功能上启用 CDN 缓存，只需一行代码:

```
// set CDN caching
// line 120
res.setHeader('Cache-Control', `public, max-age=${durationLeft}, s-maxage=${durationLeft}`);
```

**火基云功能的缺点:**
火基云功能:冷启动…

但是，您不会被 firebase 或任何其他无服务器服务所束缚。

# 结论

我们已经创建了一个全功能的 graphql 服务器，它运行在 Firebase Cloud 函数上，并具有内置的内存和 CDN 缓存。最棒的是:它是免费的(*直到你每月达到 200 万次请求，CDN 响应不算*)。

完整的代码库可以在这里找到:[https://github.com/Rusfighter/firegraph](https://github.com/Rusfighter/firegraph)

要在本地运行:

```
cd functions && npm install && npm run watch
```

要部署到 firebase，请查看 firebase 云功能文档。

## 最后的话

就像我在免责声明中说的，这是我第一篇关于 medium 的文章。而且我的英语写作能力也没有我希望的那么好，所以如果你有一些备注或者需要更多的解释，请继续并留下评论:)。

# 奖金

由于我们应该提高冷启动性能，所以我们需要延迟加载我们只对特定云功能感兴趣的模块。为此，我们创建一个助手函数:

因此，如果函数被触发，我们就延迟加载我们只需要的模块。这将导致更好的冷启动性能，尤其是当您需要大量模块时。更多改进建议请见:[https://firebase.google.com/docs/functions/tips#performance](https://firebase.google.com/docs/functions/tips#performance)