# Node.js APIs 缓存变得简单！

> 原文：<https://itnext.io/node-js-apis-caching-made-simple-f79c22b2c2fa?source=collection_archive---------4----------------------->

![](img/792acc65ccf4d755843800725fe289d2.png)

在本文中，我们将讨论如何在分布式解决方案中轻松实现 API 缓存。
描述了 Node.js 的实现，具体到伟大的[*http-cache-middleware*](https://github.com/jkyberneees/http-cache-middleware)模块:

# 但是，什么是缓存？

> **高速缓存**是存储数据的硬件或软件组件，以便将来对该数据的请求可以得到更快的服务；存储在缓存中的数据可能是早期计算的结果，也可能是存储在其他地方的数据的副本。当请求的数据可以在缓存中找到时，发生缓存命中，而当不能找到时，发生缓存未命中。通过从缓存中读取数据来服务缓存命中，这比重新计算结果或从较慢的数据存储中读取要快；因此，缓存可以处理的请求越多，系统的执行速度就越快。
> 
> HTTPS://EN。WIKIPEDIA.ORG/WIKI/CACHE_(计算)

在本文的上下文中，我们将重点讨论软件缓存，特别是 API 响应缓存，以及它如何显著改善系统的以下方面:

*   潜伏
*   CPU 要求
*   网络带宽

```
Again, statics assets caching is out of scope of this article!
```

# 分布式系统的问题

当实现诸如微服务之类的分布式系统时，我们倾向于使用几十个(如果不是几百个的话)内部具有良好定义的接口和最小业务逻辑的微小服务。一方面，这有利于维护，允许运行时多样性并提高开发速度，然而这也给开发团队带来了许多架构问题，包括: ***数据聚合的成本！***

> 在细粒度的分布式系统中，您将需要几个 API 调用来符合单个响应有效负载，从而暴露出您的体系结构中的依赖性管理和网络延迟问题。

## “A”依赖于“B”，“B”依赖于“C”

在分布式系统中我们不会有单一的数据库，因此为一个响应负载聚合数据将需要从不同的服务获取记录，从而并行和顺序地触发许多 API 调用。

# 缓存是你的朋友！

如果您的记录集检索次数多于更新次数，那么您应该使用缓存。
这将防止您一遍又一遍地重新计算您的数据，还会将 API 调用和网络往返次数降至最低。

## “http-cache-middleware”拯救世界:

> Node.js 的高性能类似连接的 HTTP 缓存中间件。因此您的延迟可以减少到个位数毫秒:[https://www.npmjs.com/package/http-cache-middleware](https://www.npmjs.com/package/http-cache-middleware)
> 
> 内部使用`cache-manager`作为缓存层，因此支持多种存储引擎，即:Memory、Redis、Memcached、Mongodb、Hazelcast…

这个中间件与类似 connect 的框架兼容，例如 [*express*](https://expressjs.com/) ， [*restana*](https://github.com/jkyberneees/ana#readme) 和许多其他框架…，使得在您的架构上实现本地和全局缓存策略变得非常容易: ***只需使用 HTTP 头，就这么简单！***

缓存一周的 *GET /tasks* 端点响应:

如果状态改变，使高速缓存无效:

# 全部缓存

正如我们提到的，分布式体系结构经常包括不同语言和运行时的服务，那么我们如何应用一个适用于所有这些服务的缓存层呢？

***http-cache-middleware***模块也与[***fast-gateway***](https://www.npmjs.com/package/fast-gateway#gateway-level-caching)兼容，通过这种组合，您可以一劳永逸地实现它，无论您的服务使用什么运行时或框架:

# 结论

在本文中，我们通过使用用于 Node.js 框架的***http-cache-middleware***中间件(如 express 或 restana……

而你，你是如何使用缓存的？