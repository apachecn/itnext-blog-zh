# K3s 上 Traefik 的入口

> 原文：<https://itnext.io/ingress-with-treafik-on-k3s-53db6e751ed3?source=collection_archive---------1----------------------->

![](img/f59a8b8b65e1502375d3a8fd224dc562.png)

照片由[乔纳森·艾利森](https://unsplash.com/@jallison154?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

这篇文章是一个关于如何使用 K3s 内置的入口控制器“Traefik”来暴露一个由 nginx 托管的网站的教程。

它基于我的上一篇文章

*   [使用 K3s 设置您自己的 Kubernetes 集群— Take 2 — k3sup](https://twissmueller.medium.com/setup-your-own-kubernetes-cluster-with-k3s-take-2-k3sup-a5d9453f709f)

这篇文章的结果是一个没有任何“有用”服务的“空”集群。在我关于安装 K3s 的第一篇文章中，我已经创建了一个基于 HAProxy 的入口控制器。当时我不知道 K3s 已经配备了一个入口控制器。

这一次，需要四种资源

*   配置映射
*   部署
*   服务
*   进入

配置映射是一种让 nginx 访问 html 文件的快速而肮脏的方法。这不是一个干净的网站托管方式，只是为了这个目的。

首先，必须创建`index.html`

```
<html>
<head><title>Hello World!</title>
  <style>
    html {
      font-size: 500.0%;
    }
    div {
      text-align: center;
    }
  </style>
</head>
<body>
  <div>Hello World!</div>
</body>
</html>
```

然后，使用以下命令创建配置映射

```
kubectl create configmap test-html --from-file index.html
```

除了部署中的卷声明之外，其余资源与上一教程中的类似

```
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    kubernetes.io/ingress.class: "traefik"
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: nginx-service
          servicePort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    run: nginx
spec:
  ports:
    - port: 80
      protocol: TCP
  selector:
    app:  nginx

---
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
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html-volume
        configMap:
          name: test-html
```

一切都适用于

```
kubectl apply -f nginx.yaml
```

让我们检查在哪个 IP 下可以到达网站

```
$ kubectl get ingress
NAME            CLASS    HOSTS   ADDRESS        PORTS   AGE
nginx-ingress   <none>   *       12.345.6.789   80      22m
```

用...试一试

```
wget 12.345.6.789
```

这将返回在本教程开始时添加到配置映射中的 html。

如果你喜欢这篇文章，请给我买杯咖啡。

# 资源

*   [K3s 联网](https://rancher.com/docs/k3s/latest/en/networking/)
*   [用 Treafik 指挥 Kubernetes 交通](https://opensource.com/article/20/3/kubernetes-traefik)