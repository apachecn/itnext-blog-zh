# 将 Ambassador Edge 堆栈部署为带 TLS 的 Kubernetes 的入口控制器

> 原文：<https://itnext.io/deploying-the-ambassador-edge-stack-as-an-ingress-controller-for-kubernetes-with-tls-cdc85495e3ac?source=collection_archive---------3----------------------->

## 辅导的

## 在不到 5 分钟的时间内将流量引入您的 Kubernetes 集群

我以前写过如何将外部流量路由到您的 Kubernetes 集群。在那篇文章中，我定义了[三个主要策略](https://blog.getambassador.io/kubernetes-ingress-nodeport-load-balancers-and-ingress-controllers-6e29f1c44f2d):使用类型为 [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) 的 Kubernetes 服务，使用类型为 LoadBalancer 的 Kubernetes 服务，或者使用 Kubernetes 入口资源。

![](img/febc54b6a48f3626403e6847014e6133.png)

在本教程中，我将展示如何使用 Ambassador Edge 堆栈作为 Kubernetes 的入口控制器，并通过 TLS 自动公开您的应用程序。

大使边缘堆栈是一个入口控制器(然后一些！)由 Envoy 代理提供支持，并通过 Kubernetes CRDs 进行声明式配置。Ambassador Edge 堆栈还集成了自动证书管理环境(ACME)支持，[为每个人实现自动 HTTPS](https://blog.getambassador.io/ambassador-integrates-automatic-https-for-kubernetes-ingress-43e738b48729)。

# 大使边缘堆栈作为入口控制器

1.  安装大使边缘堆栈作为您的入口控制器。

```
kubectl apply -f [https://www.getambassador.io/yaml/aes-crds.yaml](https://www.getambassador.io/yaml/aes-crds.yaml) && \ 
kubectl wait --for condition=established --timeout=90s crd -lproduct=aes && \ 
kubectl apply -f [https://www.getambassador.io/yaml/aes.yaml](https://www.getambassador.io/yaml/aes.yaml) && \
kubectl -n ambassador wait --for condition=available --timeout=90s deploy -lproduct=aes
```

2.我们现在将在 Kubernetes 中部署一个简单的服务，即`quote`服务。将下面的 YAML 保存到一个名为`quote.yaml`的文件中。

```
---
apiVersion: v1
kind: Service
metadata:
  name: quote
  namespace: ambassador
spec:
  ports:
  - name: http
    port: 80
    targetPort: 8080
  selector:
    app: quote
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quote
  namespace: ambassador
spec:
  replicas: 1
  selector:
    matchLabels:
      app: quote
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: quote
    spec:
      containers:
      - name: backend
        image: quay.io/datawire/quote:0.2.7
        ports:
        - name: http
          containerPort: 8080
```

3.在集群上部署`quote`服务:

```
kubectl apply -f quote.yaml
```

4.现在，创建一个入口资源，告诉大使(和我们的外部用户)如何访问`quote`服务。将下面的内容保存到一个名为`quote-ingress.yaml`的文件中。

```
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: ambassador
  name: quote-ingress
  namespace: ambassador
spec:
  rules:
   - http:
       paths:
       - path: /quote/
         backend:
           serviceName: quote
           servicePort: 80
```

4.将入口资源应用到集群:

```
kubectl apply -f quote-ingress.yaml
```

5.获取您的群集的 IP 地址:

```
kubectl get -n ambassador service ambassador -o 'go-template={{range .status.loadBalancer.ingress}}{{print .ip "\n"}}{{end}}
```

6.Ambassador Edge Stack 的一个特性是默认配置 TLS。因此，使用上一步中的 IP 地址，通过`https`向您的系统发送一个`curl`:

```
curl -k [https://YOUR_IP_ADDRESS/quote/](https://130.211.200.120/quote/)
```

不要忘记`-k`选项 Ambassador Edge 堆栈会自动安装自签名证书，因此我们需要`curl`来自动接受证书。

恭喜你！您已经安装了 Ambassador Edge 堆栈，并拥有一个成熟的入口控制器。

# 边缘策略控制台

您可以访问大使边缘堆栈边缘策略控制台 UI 来配置大使的其他方面。例如，UI 让您自动从 Let's Encrypt 获得您自己的证书。

在您的浏览器中，输入与上面相同的 IP 地址，并按照说明访问 UI。同样，在您配置自己的证书之前，您需要确保您的浏览器接受自签名证书。

# 有问题吗？

如果您对本教程或大使边缘堆栈有任何疑问，请[加入我们的 Slack 频道](http://d6e.co/slack)，或[联系我们](https://www.getambassador.io/contact)。

如果本教程解决了你的需要，我们很乐意听到它。请在下面的评论中给我们留言，或者在 Twitter 上发 [@getambassadorio](https://www.twitter.com/getambassadorio?source=post_page---------------------------) 。