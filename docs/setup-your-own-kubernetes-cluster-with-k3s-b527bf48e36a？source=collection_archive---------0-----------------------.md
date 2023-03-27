# 使用 K3s 设置您自己的 Kubernetes 集群

> 原文：<https://itnext.io/setup-your-own-kubernetes-cluster-with-k3s-b527bf48e36a?source=collection_archive---------0----------------------->

![](img/12708c549269c9d068c848981dc27ab1.png)

照片由[克里斯蒂娜@ wocintechchat.com](https://unsplash.com/@wocintechchat?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

对于本教程，使用了两台运行 Ubuntu 20.04.1 LTS 的虚拟机。

如果需要一个本地 Kubernetes 集群，那么 [K3s](https://k3s.io) 似乎是一个不错的选择，因为每个节点只需要安装一个小的二进制文件。

请注意，出于隐私原因，我已经删除了所有的域名、IP 地址等等。

## 安装集群

在将成为主节点的计算机上，安装 K3s 二进制文件:

```
curl -sfL https://get.k3s.io | sh -
```

获取下一步需要的节点令牌:

```
cat /var/lib/rancher/k3s/server/node-token
```

在将成为工作节点的每台计算机上:

```
curl -sfL https://get.k3s.io | K3S_URL=https://kubernetes01.domain.de:6443 K3S_TOKEN=<the-token-from-the-step-before> sh -
```

回到主节点，检查所有节点是否都在:

```
$ sudo k3s kubectl get node
NAME                     STATUS   ROLES                  AGE   VERSION
kubernetes01.domain.de   Ready    control-plane,master   18m   v1.20.0+k3s2
kubernetes02.domain.de   Ready    <none>                 93s   v1.20.0+k3s2
```

## 部署服务

对于第一个测试，一些 NGINX 实例被部署到集群中，清单如下:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 10
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

将其保存到文件`nginx-deployment.yml`中并安装:

```
kubectl apply -f nginx-deployment.yml
```

现在总共应该有 10 个 pods 运行一个 NGINX 实例。为什么是 10？只是为了好玩:-)适应你的喜好。

使用以下命令检查这些单元在哪里运行:

```
kubectl get pods -l app=nginx --output=wide
```

## 创建服务

这是使“内部”部署可从集群外部访问的两个步骤中的第一步。

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    run: nginx
spec:
  ports:
    - port: 80
      protocol: TCP
  selector:
    app: nginx
```

当上述清单到位时，例如`nginx-service.yml`，其适用于:

```
kubectl apply -f nginx-service.yml
```

检查是否有效:

```
$ sudo kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   00.0.0.0       <none>        443/TCP   4h9m
nginx        ClusterIP   00.00.000.000   <none>        80/TCP    13
```

但是需要负载均衡，对吧？

对！

## 带 HAProxy 的入口

创建服务后，可以通过创建基于 HAProxy 的入口从外部访问 NGINX 实例。

HAProxy 提供的原始清单相当长，因此这里不再重复。

无论如何，它可以直接从它的来源安装:

```
$ kubectl apply -f [https://raw.githubusercontent.com/haproxytech/kubernetes-ingress/v1.4/deploy/haproxy-ingress.yaml](https://raw.githubusercontent.com/haproxytech/kubernetes-ingress/v1.4/deploy/haproxy-ingress.yaml)
```

再次检查是否一切顺利:

```
$ sudo kubectl get ingress
NAME            CLASS    HOSTS   ADDRESS        PORTS   AGE
nginx-ingress   <none>   *       00.000.0.000   80      2m38s
```

此外，让我们与其中一个 pod“对话”:

```
$ curl 10.199.7.120/nginx
<html>
<head><title>404 Not Found</title></head>
<body bgcolor="white">
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.14.2</center>
</body>
</html>
```

那辆`404`很好。所有 NGINX 实例都是“空”的。

编辑 2021 年 2 月 15 日

404 不好。这里出了点问题。空的 nginx 应该显示一个欢迎页面。有些地方出错了，但是这篇文章已经过时了。

## 改变当地的 Kubernetes 环境

从一个本地终端控制集群要方便得多，而不必`ssh`进入主节点。

从主节点复制`/etc/rancher/k3s/k3s.yaml`的内容并添加到本地`~/.kube/config`。

将“localhost”替换为主 K3s 节点的 IP 或名称。

在本地机器上，用`kubectl set-context <yourcontext>`改变上下文

检查，例如通过检索所有窗格

```
kubectl get pods -o wide
```

如果你喜欢这篇文章，请给我买杯咖啡吧！

## 资源

*   [K3s 快速入门指南](https://rancher.com/docs/k3s/latest/en/quick-start/)
*   [使用部署运行无状态应用](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/)
*   [创建服务](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/#creating-a-service)
*   [如何将 HAProxy Kubernetes 入口控制器安装到您的 Kubernetes 集群中](https://www.haproxy.com/documentation/kubernetes/latest/installation/community/kubernetes/)
*   [使用 kubectl 从外部访问集群](https://rancher.com/docs/k3s/latest/en/cluster-access/)