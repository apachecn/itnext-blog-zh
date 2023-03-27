# Kubernetes 的孔门

> 原文：<https://itnext.io/kong-gateway-in-kubernetes-ad86981f93a5?source=collection_archive---------2----------------------->

Kong 是一个构建在轻量级代理 Nginx 之上的开源 API 网关。在这篇文章中，我将向您展示如何在 Kubernetes 中设置和配置 Kong。有关孔的更多信息，请访问:

> 在 [Gengo](https://gengo.com/) ，我们每天处理上亿的 HTTP 请求。我们使用 Nginx 作为内部和外部流量到 Kubernetes 集群中托管的微服务的网关。我们自己管理这些代理变得很成问题，因为我们需要控制、监视和保护流量，而不增加现有设置中的代码更改和复杂性。当然，这是为了让我们能够快速扩展。

![](img/585a9e3766da24ccf89b4a8d6eecb6a0.png)

金刚来救援了。这是一个运行在 Nginx 中的 Lua 应用程序，这意味着我们可以使用插件享受更多的功能，同时仍然使用一个非常轻量级的代理。Nginx 类固醇=)

为了在 K8s 中使用孔，我们将使用[孔入口控制器](https://github.com/Kong/kubernetes-ingress-controller)。这允许我们以管理 Kubernetes 资源的类似方式配置 Kong。

# 测试设置

*   minikube 1 . 5 . 2 版
*   kubernetes 1 . 14 . 0 版

运行此命令以应用控制器。

```
kubectl apply -f [https://bit.ly/kong-ingress-dbless](https://bit.ly/kong-ingress-dbless)
```

> 请注意，这不是 Kong 的生产级部署。

这将为 Kong 创建所需的资源，包括名称空间、服务、部署、CustomResourceDefinitions(稍后将详细介绍)、ClusterRoles 和 ConfigMaps。

在 1.1 版本发布之前，Kong 的配置通常与数据库一起使用，但我们不希望这样，因为我们的目标是一个声明式设置。

要检查它是否工作，卷曲 kong-proxy 服务。在 minikube 中，必须可以在本地主机中访问 LoadBalancer 类型的服务。要做到这一点，运行

```
$ minikube tunnel
```

现在我们可以从本地主机访问我们的服务。

```
$ export KONG_HTTP=$(minikube ip):$(kubectl -nkong get svc kong-proxy --no-headers | awk '{print $5}' | cut -d / -f 1 | cut -d : -f 2)$ curl -i $KONG_HTTPHTTP/1.1 404 Not Found
Date: Sun, 08 Dec 2019 12:10:23 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 48
Server: kong/1.3.0{"message":"no Route matched with those values"}%
```

恭喜你！孔正在工作。但是它还不知道如何代理请求。

## 让我们建立一个简单的应用程序，

```
$ curl -sL bit.ly/echo-server | kubectl apply -f -service/echo created
deployment.apps/echo created
```

## 基本路由配置

kong-ingress-controller 监视入口控制器的变化，以建立基本路由配置。

入口/api

您应该在日志中看到类似这样的内容

```
$ kubectl -n kong logs -f $(kubectl -n kong get pod — no-headers | awk ‘{print $1}’) --tail=2 ingress-controllerI1208 10:10:30.885602       1 status.go:342] updating Ingress default/api status to [{10.110.88.230 }]
W1208 10:10:30.902533       1 controller.go:135] successfully synced configuration to Kong
```

现在用 api.gengo.dev 的主机头再次卷曲端点

```
$ curl -i -H "Host: api.gengo.dev" $KONG_HTTP/HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Date: Sun, 08 Dec 2019 12:20:14 GMT
Server: echoserver
X-Kong-Upstream-Latency: 14
X-Kong-Proxy-Latency: 3
Via: kong/1.3.0Hostname: echo-758859bbfb-bx9txPod Information:
 node name: minikube
 pod name: echo-758859bbfb-bx9tx
 pod namespace: default
 pod IP: 172.17.0.7...
```

恭喜你！您已经有了基本的路由配置。现在让我们注射一些类固醇。

# 自定义资源定义

由于 Kong 提供了比我们刚才设置的基本路由配置更多的功能，它允许我们通过使用自定义资源定义来进行进一步的配置。

*   **KongPlugin:** 在 Kong 中安装和配置插件
*   **KongIngress:** 作为入口资源的“扩展”,用于流量的细粒度控制(路由、负载平衡，甚至健康检查)
*   **孔消费者:**映射到孔[消费者](https://getkong.org/docs/latest/admin-api/#consumer-object)实体。
*   **KongCredential:** 允许设置将与 KongConsumer 关联的凭证。

## 孔插件

这就是我真正喜欢孔的地方。轻量级 Nginx 的功能可以通过使用插件轻松扩展。如果你想查看不同功能(认证、安全、无服务器、监控、日志等)的可用插件列表，有一个 [KongHub](https://docs.konghq.com/hub/) 。)

我最喜欢使用的插件之一是**无服务器函数**，我们可以在访问阶段动态运行 Lua 代码。插件有两种。在插件链中，每一个都有不同的优先级。

*   **前置功能** —在其他插件之前运行
*   **后置功能** —在其他插件之后运行

下面的插件举例说明了如何在 Lua 中读取 JWT 令牌的每一段。

孔插件/读-jwt-令牌

并且我们也可以为 Lua 使用 [Nginx API](https://github.com/openresty/lua-nginx-module#nginx-api-for-lua) 。多酷啊。

要使用它，我们需要通过使用注释将它与入口关联起来。

例如(假设我们有另一个名为 http-jwt 的插件，它在读取令牌之前检查它的有效性)

```
annotations:
  plugins.konghq.com: http-jwt,read-jwt-token
```

## KongIngress Man！！

在大多数情况下，入口资源是足够的，但如果你想更多地控制路由，孔入口是你的男人。使用这个可以修改与入口资源相关的[上游](https://getkong.org/docs/latest/admin-api/#upstream-objects)、[服务](https://getkong.org/docs/latest/admin-api/#service-object)和[路由](https://getkong.org/docs/latest/admin-api/#route-object)实体的所有属性。

供您参考，这是完整的规格，

孔入口/全规格

所有配置都是不言自明的，如果不是，请参考[文档](https://docs.konghq.com/1.4.x/admin-api/#upstream-object)。

要使用它，我们需要通过使用注释将其与入口或服务相关联。

```
annotations:
  configuration.konghq.com: kong-ingress-resource-name
```

它可以与入口或服务相关联，但是我们如何知道是哪一个呢？

如果您在上游或代理节点中进行配置更改，它需要与服务相关联，而路由节点与入口相关联

```
upstream: # Associate with Service
  hash_on: none
  hash_fallback: noneproxy: # Associate with Service
  protocol: http
  path: /
  connect_timeout: 10000
  retries: 10
  read_timeout: 10000
  write_timeout: 10000route: # Associate with Ingress
  methods:
  - POST
  - GET
```

## 孔消费者

有些情况下，我们希望对访问端点的每个用户/消费者使用不同的规则。例如，web 客户端上的速率限制条件不同于以编程方式访问 API 的机器。这可以用孔消费者来实现。

下面是创建消费者的清单文件。

```
apiVersion: configuration.konghq.com/v1
kind: KongConsumer
metadata:
  name: api-consumer
  annotations:
    plugins.konghq.com: api-rate-limit
username: api
custom_id: api-1---
apiVersion: configuration.konghq.com/v1
kind: KongConsumer
metadata:
  name: web-consumer
  annotations:
    plugins.konghq.com: web-rate-limit, web-cors
username: web
custom_id: web-1
```

在这个例子中，我们创建了两个消费者，每个消费者根据他们的需求与不同的插件相关联。

只需使用注释将 KongPlugins 与您的消费者关联起来。

对于孔来说，消费者是一个不透明的概念，因为它可以根据不同的使用情况进行不同的处理。这真的很令人困惑。

例如，你可以和你的用户进行一对一的交流，或者像上面的例子一样。核心原则是你可以给它们附加插件，从而定制请求行为。

在大多数情况下，消费者是通过身份验证插件来识别的，所以凭证部分就来了。

## 孔凭据

KongCredentials 是可以分配给 KongConsumer 的不同类型的独立实体。

*   **key-auth** 用于[密钥认证](https://docs.konghq.com/plugins/key-authentication/)
*   **基本认证**为[基本认证](https://docs.konghq.com/plugins/basic-authentication/)
*   **hmac-auth** 用于 [HMAC 认证](http://docs.konghq.com/plugins/hmac-authentication/)
*   **jwt** 用于[基于 jwt 的认证](http://docs.konghq.com/plugins/jwt/)
*   **oauth2** 为 [Oauth2 客户端凭证](https://docs.konghq.com/hub/kong-inc/oauth2/)
*   **acl** 用于 [ACL 组关联](https://docs.konghq.com/hub/kong-inc/acl/)

请注意，一个消费者可以有许多凭证。

```
apiVersion: configuration.konghq.com/v1
kind: KongCredential
metadata:
  name: web-apikey
consumerRef: web-consumer
type: key-auth
config:
  key: super-secret-key
```

在我们的例子中，我们使用集成到 Auth0 中的 JWT 插件，每个 Auth0 租户只需要 1 个消费者和 1 个凭证。清单文件如下所示

孔-auth0-integration.yaml

以上就是孔在库伯内特斯的故事。我希望你在 Kubernetes 学会了如何使用孔。如果你有意见/建议/问题，请在下面的评论区告诉我。

感谢阅读:)

> 我乐于接受反馈和建议，这样我就可以改进。:)

参考资料:

[](https://docs.konghq.com/?itm_source=website&itm_medium=nav) [## 开源 API 管理和微服务管理

### 使用插件保护、管理和扩展您的 API 或微服务，用于身份验证、日志记录、速率限制…

docs.konghq.com](https://docs.konghq.com/?itm_source=website&itm_medium=nav) [](https://github.com/Kong/kubernetes-ingress-controller) [## kong/kubernetes-入口控制器

### 使用孔作为 Kubernetes 入口。在 Kong 为 Kubernetes 配置插件、健康检查、负载平衡等等…

github.com](https://github.com/Kong/kubernetes-ingress-controller)