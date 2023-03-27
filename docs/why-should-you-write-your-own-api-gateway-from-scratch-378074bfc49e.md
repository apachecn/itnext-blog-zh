# 为什么您应该从头开始编写自己的 API 网关

> 原文：<https://itnext.io/why-should-you-write-your-own-api-gateway-from-scratch-378074bfc49e?source=collection_archive---------1----------------------->

![](img/8c545c8d48d8ac9d02f80a11d549dd24.png)

照片由[莱拉·格布哈德](https://unsplash.com/@lailagebhard?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

每个拥有微服务的组织都需要控制谁可以在什么条件下访问这些服务。API 网关是这个问题不可避免的解决方案。

但是，您是否应该使用现有的可配置代理，如 [Envoy](https://www.envoyproxy.io/) 、 [Ngnix](https://www.nginx.com/) 、 [Zuul](https://github.com/Netflix/zuul) 、 [Kong](https://docs.konghq.com/enterprise/) 、 [aws API gateway](https://aws.amazon.com/api-gateway/) (这个列表还可以继续下去)？这些项目中的每一个都有其优点和缺点，都有自己的配置语言、用户社区、书籍、文档和教程。

在这篇文章中，我认为你不需要，你需要的一切都可以通过几条 golang 线来实现。

这是可能的，因为`"net/http/httputil"`里的好东西。这个包是`"net/http"`的扩展，包含网络实用程序。

其中之一是`httputil.ReverseProxy`，顾名思义，可以处理透明转发 HTTP 请求所需的所有网络相关的东西，只需一行代码:`Proxy(targetUrl).ServeHTTP(ctx.Writer, ctx.Request)`

它可以与任何 golang HTTP 框架一起工作，如 [**gin**](https://github.com/gin-gonic/gin) 或 [**fast-http**](https://github.com/valyala/fasthttp) ，现有最快的服务器框架之一。

反向代理不限于透明的反向代理。它可用于修改拦截的请求及其响应。

例如，假设一个常见的场景，每个请求都需要:

*   笨拙的
*   一个公制发射
*   根据请求路径中的约定，路由到特定的微服务
*   已验证(用未加密的用户详细信息设置请求头)
*   经授权的
*   速率受限
*   在 staging env 上，如果是 500，响应主体需要修改并包含错误消息，以改善开发体验
*   在 prod env 上，它必须包含错误的散列，以便以后在搜索日志时使用。

配置 NGNIX，ZUUL，或者 ENVOY 做上面的列表可能是可以的，我承认我没有尝试过。但肯定不会是在公园散步，无论上面的代理有多么丰富的功能，它都无法与自定义代码的灵活性相比。

下面的例子包含了 87 行 golang，其中除了认证/授权/速率限制部分之外，还包含了上述所有内容，这部分内容不在讨论范围之内，但本质上将作为请求中间件来实现。

我们来深究一下，说说有意思的部分。

完整的 api 网关示例

*   `Target`函数返回被寻址微服务的内部 k8s 地址。例如，对`/api/product/frontend/users`的请求将变成`http://svc.product.frontend.cluster.local:10000/api/users`(一个内部 k8s DNS)。这里的约定是请求路径包括服务名和服务名称空间，k8s 服务名是`svc-NAME`。
*   `p.ModifyResponse`包含修改响应和记录错误的代码，
*   `models.FailedToFindProxyTarget.Inc()`指标报告
*   `Proxy(targetUrl).ServeHTTP(ctx.Writer, ctx.Request)` -请求转发(重写)

就是这样。如果你想要任何自定义行为，可以在这里介绍。通过一些额外的 golang 魔法，上述内容还可以作为 gRPC 和 graphql 请求的网关。

一点也不麻烦，性能稳定，网关可以轻松地扩展到您需要的任意数量的实例，并以无限种方式进行扩展。并且为它的逻辑编写测试根本不需要动脑筋。

如果这对你有用，请在下面的评论中告诉我。

干杯！