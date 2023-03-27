# 管理 Kubernetes 上的入口控制器:第 3 部分

> 原文：<https://itnext.io/managing-ingress-controllers-on-kubernetes-part-3-2984b3616249?source=collection_archive---------2----------------------->

## 管理 SSL/TLS 的选项

![](img/1c824515f1a0105dc3485bc9d2e61af4.png)

由[本杰明·沃罗斯](https://unsplash.com/@vorosbenisop?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

当我们谈论到达我们集群的流量时，安全性是一个重要的考虑因素。幸运的是，入口 API 支持为服务配置传输层安全性(TLS)。这样做可以避免以明文形式与服务进行通信。客户也更加确信他们实际上是在与预期的服务进行对话。让我们看看如何在入口资源中配置 TLS。

## 基本配置

主机的 TLS 工件(X.509 证书和私钥)需要存储在一个秘密中，然后在入口对象的定义中引用该秘密:

```
apiVersion: v1data: tls.crt: <base64 encoded string> tls.key: <base64 encoded string>kind: Secretmetadata: name: acme.com.tls namespace: defaulttype: kubernetes.io/tls---apiVersion: extensions/v1beta1kind: Ingressmetadata: name: acme-app-ingressspec: tls: — hosts: - acme.com secretName: acme.com.tlsrules:- host: acme.com http: paths: - path: /foo backend: serviceName: svc1 servicePort: 80
```

对于 TLS 的简单默认配置，有几件事情需要考虑。首先，入口实现假设 TLS 在入口点终止，流量使用普通 HTTP 路由到服务后端。稍后会有更多的介绍。第二，入口 API 仅支持端口 443 上的 TLS，并且如果在入口定义的 TLS 组件中指定了多个主机，则借助于用服务器名称指示(SNI) TLS 扩展指定的主机名，流量在同一端口上被多路复用，这必须由所讨论的特定入口控制器支持。

[![](img/c46c9bc5c32531dc4b9e02fbc12fd9f9.png)](https://info.giantswarm.io/guide-to-managing-ingress-controllers)

## SSL/TLS 直通

上面的例子表明，TLS 在入口点终止。在入口点终止 TLS 将后端服务单元从解密流量的高成本任务和证书管理负担中解放出来。但是，如果需要的话，可以将通信一直加密到服务单元。并不是所有的入口控制器都支持这个特性，它们以不同的方式处理它。

例如，Traefik 入口控制器检查入口定义中的服务端口，如果端口是 443，它将尝试使用 TLS 与每个 pod 端点通信。k8s-ingress-nginx 控制器使用`nginx.ingress.kubernetes.io/ssl-passthrough`(必须设置为“真”)和它的`— enable-ssl-passthrough`命令行标志。

如果不是必需的，那么采用入口 API 的默认行为并在入口点终止 TLS 会更方便，也更少麻烦。

## 其他 SSL/TLS 注意事项。小心 HSTS 的头球！

默认情况下，k8s-ingress-nginx 控制器在相关入口对象上配置了 TLS 的响应中自动应用 HTTP 严格传输安全(HSTS)报头。这指示客户端在六个月的默认时间段内仅使用 HTTPS 与后端服务通信。

应特别小心，确保不要求客户端使用普通 HTTP 访问相关域的资源，否则客户端可能在六个月内无法访问这些资源！

这里有一个例子:网站可能会利用由第三方托管的资产或服务。自定义子域可以与指向托管资产的 DNS CNAME 记录结合使用，但是如果子域没有 TLS 证书，则资产将使用普通 HTTP 提供服务。当 HSTS 作为入口定义的一部分启用时，所有 HTTP 请求都将被客户端升级到 HTTPS，导致内容无法按预期呈现。在 HSTS 最大年龄参数持续期间，修复服务器端的问题对有问题的客户端没有影响。明智的预防措施可能是首先禁用此功能(这是 Traefik 入口控制器的默认行为)，直到与任何后端服务相关的通信要求都已完全测试完毕。

最后，手动管理与入口对象相关联的 TLS 证书意味着巨大的操作开销。 [Jetstack 的 cert-manager](https://github.com/jetstack/cert-manager) 控制器可用于从外部认证机构(如 [Let's Encrypt](https://letsencrypt.org/) )自动获取和更新 TLS 证书。

> [**阅读第 4 部分: *D* 部署多个入口控制器**](https://medium.com/@GiantSwarm/managing-ingress-controllers-on-kubernetes-part-4-9f201296db28)

由 [Puja Abbassi](https://twitter.com/puja108) 撰写——开发者倡导者@ [巨型群体](https://giantswarm.io/)

[](https://twitter.com/puja108) [## puja Abbas si @ # kube con # kube Khan(@ puja 108)| Twitter

### Puja Abbassi 的最新推文@ #KubeCon #KubeKhan (@puja108)。开发者倡导者@ GiantSwarm & Researcher

twitter.com](https://twitter.com/puja108) 

准备好将您的云原生项目投入生产了吗？只需在 https://giantswarm.io/[申请免费试用](https://giantswarm.io/)