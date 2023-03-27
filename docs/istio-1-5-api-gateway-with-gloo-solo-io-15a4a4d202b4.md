# 带有 Gloo — Solo.io 的 Istio 1.5 API 网关

> 原文：<https://itnext.io/istio-1-5-api-gateway-with-gloo-solo-io-15a4a4d202b4?source=collection_archive---------1----------------------->

![](img/fd84fe6aa89b10ab2cc79111d51e2222.png)

Gloo 是一个构建在 Envoy Proxy 之上的 API 网关，它高度补充了像 Istio 这样的服务网格，具有像[转换](https://docs.solo.io/gloo/latest/guides/traffic_management/request_processing/transformations/)、 [OIDC 认证](https://docs.solo.io/gloo/latest/guides/security/auth/oauth/)、 [OPA 授权](https://docs.solo.io/gloo/latest/guides/security/auth/opa/)、 [Web 应用防火墙(WAF)](https://docs.solo.io/gloo/latest/guides/security/waf/) 、[等边缘功能](https://www.solo.io/products/gloo/)。我们的许多 Solo.io 客户将这两者结合起来取代传统的 API 管理供应商。我已经写了很多关于 API 网关和服务网格的重叠和互补的角色。在之前的博客文章中，我们也探讨了将 Istio 和 Gloo 结合起来的[。](https://www.solo.io/blog/using-gloo-as-an-ingress-gateway-with-istio-and-mtls-updated-for-istio-1-1/)

在 [Istio 1.5 版本](https://istio.io/news/releases/1.5.x/announcing-1.5/)中，随着人们不再部署和管理 Istio，许多架构方面的考虑发生了变化。Istio 中实现 mTLS 的方式也发生了一些变化。例如，在过去，Istio 会为每个服务帐户创建秘密，并将这些秘密装载到工作负载中，以取得它们的身份。这种情况现在已经改变了。现在，Istio 1.5 默认使用 [Envoy 的 SDS](https://www.envoyproxy.io/docs/envoy/latest/configuration/security/secret) 来分发工作负载身份/证书。

现在[我们已经与 Istio SDS 集成了一段时间](https://docs.solo.io/gloo/latest/guides/integrations/service_mesh/gloo_istio_mtls/)，同时提供了使用 SDS(更安全)或秘密安装方法的选项，但现在随着 Istio 1.5 的改变，在它实现 SDS 的一些其他方式中，我们已经更新了我们的 Gloo 文档，以显示如何让 Gloo 与 Istio 1.5 一起工作。

有两种不同的方法可以做到这一点。Gloo OSS 支持的方式是加载一个 Istio 代理(和 *istio-agent* )来连接到网格，下载证书并允许上游使用。不幸的是，这需要在 Gloo 代理(也基于 Envoy)旁边运行另一个除了下载证书之外什么也不做的 Envoy。这是[推荐的完成 SDS 和 Istio 集成](https://istio.io/blog/2020/proxy-cert/)的方式。然而，对于 GlooEE，我们觉得我们可以做得更好。我们有一个定制的 *istio-agent* ，可以为 istio 提供 SDS 服务，而不需要运行一个完全独立的特使。这简化了与 Istio 集成时 Gloo API 网关的部署。在 [Slack 上联系我们，或者通过 Solo.io 网站了解更多关于这个](https://www.solo.io/products/gloo/)的信息。

看这个 Gloo + Istio 1.5 的短视频演示

从文档开始，查看结合 Gloo 和 Istio 的[。加入我们的 slack](https://docs.solo.io/gloo/latest/guides/integrations/service_mesh/gloo_istio_mtls/)[【https://slack . solo . io】](https://slack.solo.io)如果你有任何问题想要讨论(任何服务网格、API 网关、特使、Web 组装等等！)

*原载于 2020 年 4 月 10 日*[*https://www . solo . io*](https://www.solo.io/blog/istio-1-5-api-gateway-with-gloo/)*。*