# 管理 Kubernetes 上的入口控制器:第 2 部分

> 原文：<https://itnext.io/managing-ingress-controllers-on-kubernetes-part-2-36a64439e70a?source=collection_archive---------2----------------------->

## 实现它:各种入口控制器

![](img/e4ac61676535fb45dd0d73880a14eef6.png)

伊琳娜·布洛克在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

制造入口资源实际上并不建立任何路由能力。为此，我们需要一个入口控制器。与 Kubernetes 集群上部署的所有其他控制器一样，入口控制器监视入口资源的变化，然后使用规则定义来路由传入流量。

入口控制器通常作为 pod 在集群中实施。它们由社区和其他第三方提供。因此，有许多不同的实现。有些甚至提供类似的功能。这意味着要做出选择，在有选择的地方，有时很难找到最佳选择。

因此，为了帮助您做出选择，让我们比较一些更受欢迎的入口控制器，从 Nginx 入口控制器开始。

## Nginx 入口控制器

我们要看的第一个入口控制器是由 Kubernetes 社区维护的基于 Nginx 的入口控制器(以下简称为 [k8s-ingress-nginx 控制器](https://kubernetes.github.io/ingress-nginx/))。这不要与 Nginx 社区维护的基于 Nginx 的 Ingress 控制器，或基于 Nginx Plus 的商用 Ingress 控制器相混淆。它们在本质上是相似的，但是 k8s-ingress-nginx-控制器支持比 Nginx 社区支持的控制器更高级的配置，而 Nginx Plus 控制器必须通过订阅获得许可。

k8s-ingress-nginx 控制器基于 nginx 反向代理的开源版本，但在中编译了一些额外的第三方模块。让我们来看看使它与众不同的一些特征。

## 高度可配置

Nginx 是一个高度可配置的反向代理，许多不同的配置选项作为 k8s-ingress-nginx 控制器的一部分直接可用。控制器自带一个全面的[默认配置](https://github.com/kubernetes/ingress-nginx/blob/master/internal/ingress/controller/config/config.go)，包含在控制器本身中，可以用几种方式覆盖。

要微调 Nginx 配置，该配置将应用于与 k8s ingress-nginx 控制器相关联的任何入口资源定义中的规则相匹配的流量，必须将该配置定义和部署为 ConfigMap，控制器的命令行参数(— `configmap`)在其部署资源定义中引用该配置。例如，要将默认的`keep-alive`值从`75`更改为`60`，ConfigMap 必须包含以下代码片段:

```
data: keep-alive: "60"
```

如果偏好在入口资源定义级别而不是在控制器级别提供配置定制，则可以将注释应用于特定入口资源定义，指定相关配置。假设我们有一个入口资源定义，这是与 k8s-ingress-nginx 控制器相关联的众多定义之一，并且我们需要对访问规则中定义的服务后端的用户进行身份验证，我们可以使用以下注释作为入口资源定义的一部分:

```
metadata: name: important-service annotations: nginx.ingress.kubernetes.io/auth-type: basic nginx.ingress.kubernetes.io/auth-secret: users
```

应用配置的最后一种方法是通过使用模板文件，使用 Golang 文本/模板语法指定配置。这是一种更高级的定制技术，但在未来的控制器更新中容易出现重大变化。

总而言之，k8s-ingress-nginx 控制器提供了全面的入口功能，具有相同的默认配置，但能够微调该配置，以适应几乎任何使用情况。

## 高效的配置重新加载

在 Kubernetes 等动态的云原生环境中，工作负载会不断地被部署、扩展和删除。为了将流量路由到组成服务的 pod，k8s-ingress-nginx 控制器使用服务端点而不是其虚拟 IP 地址。这意味着底层 Nginx 配置需要不断刷新，以便控制器知道将流量路由到哪里。如果持续的端点变化迫使 Nginx 每次都重新加载，这可能会显著影响控制器的性能及其有效路由入口流量的能力。

幸运的是，k8s-ingress-nginx 控制器避免了端点变化时的这种需要，包括了控制器监视端点变化的 OpenResty `lua_nginx_module.` ，为 Lua 处理程序提供了一个列表，Lua 处理程序确定传入请求应该发送到哪个后端对等点或端点。Nginx 然后负责路由请求。

在部署了大量服务的集群中，无需不断重新加载配置，这使得 k8s-ingress-nginx 控制器成为管理入口的可行选择。需要注意的是，虽然 k8s-ingress-nginx 控制器从 0.12 版本开始就有动态配置重新加载功能，但从 0.18 版本开始，该功能仅默认开启。

## 支持 TCP/UDP 服务

Kubernetes Ingress 通常仅限于第 7 层 HTTP 路由，并不提供任何基于寻址端口和协议路由流量的“本地”功能。但是，k8s-ingress-nginx 控制器可以为基于 TCP 和 UDP 的服务配置 ConfigMap，这使它能够将面向客户端的端口映射到命名空间服务和端口。每个协议的配置图需要在启动时作为参数提供给控制器。公开服务的配置图片段可能如下所示:

```
data:
  6379: “default/redis:6379”
```

从技术上来说，这是对 Kubernetes Ingress API 对象中缺乏第 4 层负载平衡支持的一种变通办法，但尽管如此，这仍然是 k8s-ingress-nginx 控制器的一个有用特性。

## 会话关联性

我们已经讨论过 k8s-ingress-nginx 控制器使用端点 API 来监视 pods 的变化，而不是基于服务的虚拟 IP 地址来路由流量，这带来了一些有用的好处。通过 kube-proxy 绕过服务请求的正常代理，它允许对客户端连接使用会话亲缘关系，这是通过路由到服务的虚拟 IP 不可能实现的。

[![](img/c46c9bc5c32531dc4b9e02fbc12fd9f9.png)](https://info.giantswarm.io/guide-to-managing-ingress-controllers)

## 轮廓入口控制器

Contour 是一个开源的入口控制器，它使用了另一个广受好评的云原生代理；[特使](https://www.envoyproxy.io/)。除了作为反向代理之外，Envoy 还有许多应用，但它非常适合作为入口控制器。

通常，Contour Ingress controller 被部署为单个 pod，由一个 Envoy 容器和一个 Contour 容器组成，后者充当 Envoy 的“管理服务器”。轮廓组件的主要用途是双重的。首先，它监视与 Ingress 相关联的各种 Kubernetes API 对象的变化，然后将它们转换为 Envoy 使用的等效资源对象。其次，它通过实现 Envoy 使用的各种发现服务(xDS)API 来响应 Envoy 的轮询，并返回必要的配置供 Envoy 使用。Contour 通过 gRPC 提供 xDS APIs，Envoy 使用它们。该入口控制器的独特之处在于:

## 对象翻译

Kubernetes 和 Envoy 说的不是同一种语言。Kubernetes 依赖于入口、[服务](https://blog.giantswarm.io/basic-kubernetes-concepts-iii-services-give-abstraction/)、[端点](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-v-daemon-sets-and-jobs/)和[秘密](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-iv-secrets-and-configmaps/)等资源。Contour Ingress 控制器通过在这些 Kubernetes 资源和相应的 Envoy 资源之间执行转换来实现动态配置更改。

Envoy 集群大致相当于 Kubernetes 服务，Contour 和 Envoy 之间的通信是使用 Envoy 的集群发现服务(CDS) API 进行的。监听器发现服务(LDS) API 用于配置 Envoy 监听器，数据由包含 TLS 工件的入口对象和秘密对象提供。入口对象还提供路由配置，通过路由配置发现服务(RDS) API 进行通信。最后，使用端点发现服务(EDS) API 将 Kubernetes 端点对象转换为 Envoy ClusterLoadAssignment 对象。

## API 驱动的配置

虽然 k8s-ingress-nginx 控制器通过包含 OpenResty lua_nginx_ module 避免了配置重载，但 Contour Ingress 控制器受益于 Envoy 的动态 API 驱动配置。这意味着 Envoy 不需要重新加载配置，从而确保控制器保持高性能，即使在配置不断变化的情况下也是如此，正如我们在云原生的容器化环境中所预期的那样。

## IngressRoute 自定义资源定义

虽然 Contour Ingress controller 与标准 Ingress API 资源配合得非常好，但它也可以利用名为 IngressRoute 的自定义资源。Kubernetes 1.1 在 2015 年底引入了标准的入口 API 资源。自那以后，它的变化很小，并且没有完全模拟许多入口场景所需的行为。因此，许多入口控制器利用[注释](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)来克服默认入口 API 资源的不足。

为了改善这一点，CRD 入口路线旨在通过更全面的功能来丰富 Kubernetes 入口体验。一些额外的功能包括:加权和负载平衡策略的定义、特定路由到多个上游服务的负载平衡路由、上游服务粒度的健康检查等等。

Envoy 提供的 API 驱动配置非常适合入口控制器，Ingres route CRD 提供的功能使 Contour 入口控制器成为管理 Kubernetes 集群入口的强大资源。

现在让我们来看看这个系列的最后一个入口控制器，Traefik 入口控制器:

## Traefik 入口控制器

得知 Traefik 入口控制器基于 [Traefik](https://traefik.io/) 反向代理和负载均衡器并不奇怪。与 Envoy 非常相似，Traefik 是在最近的容器驱动的云原生运动中构思出来的。它是开源软件，由它的创造者 Containous 提供商业支持。Traefik 本质上是通用的，除了 Kubernetes 之外，它还支持各种不同的基础设施环境或“提供商”。

Traefik 入口控制器的工作方式与前面讨论过的入口控制器非常相似，它使用标准的入口 API 对象和包含在二进制文件中的控制器，后者是从最小 Docker 映像部署的。控制器根据被监视的 Kubernetes API 对象动态更新其配置，并提供了一个默认的内置 Traefik 配置，如果需要，可以覆盖该配置。

Traefik 入口控制器实现与入口 API 对象相关联的行为，为群集中部署的服务提供基于主机和路径的路由。然而，作为一个全面的反向代理，Traefik 入口控制器通过一组可选注释提供了许多有趣的配置选项。让我们来看看一些更有趣的功能。

## 限速

Traefik 入口控制器可以通过指定平均速率和突发速率来限制在给定时间段内发送到后端服务的客户端请求的数量。此限制适用于每个客户端 IP 地址。使用`traefik.ingress.kubernetes.io/rate-limit` 键指定速率限制，YAML 值指定请求的时间段、平均值和突发数。这对于保护后端服务不被请求淹没特别有用，并且可以防止 DDoS 攻击。

## 断路

断路是大多数现代代理的一个相当标准的特性，当后端被认为处于故障或失败状态时，它允许系统关闭到后端的路由。Traefik 入口控制器通过定义延迟和/或网络错误和/或 HTTP 响应代码比率的表达式来提供这种能力。断路器表达式是使用作为后端服务定义的一部分提供的`traefik.ingress.kubernetes.io/circuit-breaker-expression` 注释提供的。

## 会话关联性

粘性会话可以通过在后端服务上将`traefik.ingress.kubernetes.io/affinity` 注释设置为“true”来实现。另一个注释允许设置 cookie 的名称。

## 具有服务权重的流量分流

`traefik.ingress.kubernetes.io/service-weights`注释允许根据为每个服务定义的权重，在不同的后端服务之间分割寻址到特定主机和路径的流量。该键的值是 YAML，它为每个提供服务指定一个百分比。以这种方式进行流量分离的完美用例是执行 canary 版本，其中一部分流量可以被定向到 canary 服务，随着对新版本的信心增加而逐渐增加。服务权重是 Traefik 即将发布的 1.7 版本的一项功能，在此之前的版本中不可用。

这只是 Traefik 入口控制器提供的功能的一个子集，通过使用入口或服务对象注释，还可以获得许多更高级的功能。

> [**阅读第 3 部分:管理 SSL/TLS 的选项**](https://medium.com/@GiantSwarm/managing-ingress-controllers-on-kubernetes-part-3-2984b3616249)

由[Puja Abbas si](https://twitter.com/puja108)——开发者倡导者@ [巨型群体](https://giantswarm.io/)撰写

[](https://twitter.com/puja108) [## puja Abbas si @ # kube con # kube Khan(@ puja 108)| Twitter

### Puja Abbassi 的最新推文@ #KubeCon #KubeKhan (@puja108)。开发者倡导者@ GiantSwarm & Researcher

twitter.com](https://twitter.com/puja108) 

准备好将您的云原生项目投入生产了吗？只需在 https://giantswarm.io/[申请免费试用](https://giantswarm.io/)