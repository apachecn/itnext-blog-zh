# 如何在 K3s 上设置私人图像注册表

> 原文：<https://itnext.io/how-to-setup-a-private-registry-on-k3s-d9283906d16?source=collection_archive---------0----------------------->

![](img/18a3e7c3ec4a7c74509943e1004def6a.png)

由[马克西姆·卡哈里茨基](https://unsplash.com/@qwitka?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄的照片

将自己的容器映像推送到公共注册中心并不总是合适的。这篇文章展示了一种在 K3s Kubernetes 集群中创建私有映像注册表的快速方法。

请注意，对于以下清单，当从群集中删除注册表资源时，所有映像也将被删除。清单的最后一行有一个 TODO 来解决这个问题。

同样需要注意的是:注册表是不安全的。必须采取进一步的措施来保护它，如用户名/密码或证书，这不是本教程的范围。

## 库伯内特资源公司

在应用这个清单之前，`spec:rules:host`下的域名需要相应地更改。

```
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: docker-registry-ingress
  annotations:
    kubernetes.io/ingress.class: "traefik"
spec:
  rules:
  - host: registry.domain.de
    http:
      paths:
      - path: /
        backend:
          serviceName: docker-registry-service
          servicePort: 5000

---
apiVersion: v1
kind: Service
metadata:
  name: docker-registry-service
  labels:
    run: docker-registry
spec:
  selector:
    app: docker-registry
  ports:
    - protocol: TCP
      port: 5000

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-registry
  labels:
    app: docker-registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: docker-registry
  template:
    metadata:
      labels:
        app: docker-registry
    spec:
      containers:
      - name: docker-registry
        image: registry
        ports:
        - containerPort: 5000
          protocol: TCP
        volumeMounts:
        - name: storage
          mountPath: /var/lib/registry
        env:
        - name: REGISTRY_HTTP_ADDR
          value: :5000
        - name: REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY
          value: /var/lib/registry
      volumes:
      - name: storage
        emptyDir: {} # TODO -make this more permanent later
```

清单应用于:

```
kubectl apply -f registry.yaml
```

## 设置

在 K3s 节点上，需要用以下内容创建文件`/etc/rancher/k3s/registries.yaml`。域名需要与上述清单中的域名相匹配。

```
mirrors:
  "registry.domain.de":
    endpoint:
      - "http://registry.domain.de"
```

我在服务器和代理节点上这样做了。我不确定是否真的要在代理人身上做，我已经做了。

之后，需要重新启动服务器和代理。

在服务器上执行:

```
systemctl restart k3s
```

在代理节点上:

```
systemctl restart k3s-agent
```

如果应用的更改可以通过以下方式检查:

```
crictl info
```

有一个叫做`registry`的部分应该会列出新创建的私有注册表。

本地工作站也需要知道新的注册表。我用 Docker 桌面在 macOS 上工作。在“首选项-> Docker 引擎”下，必须使用以下条目扩展设置:

```
{
  ...

  "insecure-registries": [
    "registry.domain.de"
  ]
}
```

## 测试注册表

为了测试一个图像是否可以被推送到新的注册表，我用一个定制的 HTML 文件构建了一个仅包含 Nginx 的小图像:

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

新图像的 docker 文件:

```
FROM nginx:alpine
COPY index.html /usr/share/nginx/html
```

根据您的注册域构建并标记新映像:

```
docker build -t registry.domain.de/hello:latest .
```

最后图像可以被推送。

```
docker push registry.domain.de/hello:latest
```

希望这没有任何问题。

对于另一个测试，也可以从本地工作站删除映像，然后再从私有注册表中取出。

```
docker rmi registry.domain.de/hello:latest
docker pull registry.domain.de/hello:latest
```

如果你喜欢这篇文章，请随时给我买杯咖啡！

## 资源

*   [Kubernetes (K3s) —添加私有的不安全注册表](https://llimon.github.io/post/k3s-registry/)
*   [在 K3s 上设置私有注册表](/setup-a-private-registry-on-k3s-f30404f8e4d3)
*   [在 k3s 上安装 docker 注册表](https://carpie.net/articles/installing-docker-registry-on-k3s)