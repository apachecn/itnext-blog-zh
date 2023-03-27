# 在 K3s 上设置私有注册表

> 原文：<https://itnext.io/setup-a-private-registry-on-k3s-f30404f8e4d3?source=collection_archive---------1----------------------->

![](img/292b2a2c06a94676eb7b5114fc349282.png)

我最近需要建立一个私人注册中心来展示 Kubernetes 的一些特性。轻量级 Kubernetes K3s 上的 Docker-registry 舵图很好地服务于我的目的。

## 1.创建证书

使用 [cfssl](https://github.com/cloudflare/cfssl) 工具创建一个自签名证书。

准备以下 myca.json 文件

```
{
   "CN": "k3s",
   "hosts": [ "k3s" ],
   "key": {
      "algo": "rsa",
      "size": 2048
   },
   "names": [
      {
          "C": "SG",
          "ST": "SG",
          "L": "Singapore"
      }
  ]
}
```

生成自签名根 ca 证书，

```
cfssl gencert -initca myca.json | cfssljson -bare myca
```

创建以下 serverRuest.json 文件，

```
{
   "CN": "registry",
   "hosts": [ "ubuntu" ],
   "key": {
      "algo": "rsa",
      "size": 2048
   }
}
```

使用下面的命令生成服务器证书，

```
cfssl gencert -ca=myca.pem -ca-key=myca-key.pem -config=ca-config.json -profile=server -hostname=ubuntu serverRequest.json | cfssljson -bare registry
```

现在我们已经创建了证书 registry.pem 和密钥文件 registry-key.pem。在 K3s 集群中创建 Kubernetes 的 TLS 秘密，

```
kubectl -n kube-system create secret tls registry-ingress-tls --cert=registry.pem --key=registry-key.pem
```

## 2.展开舵图

创建以下 helmchart CRD 注册表

```
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: docker-registry
  namespace: kube-system
spec:
  chart: stable/docker-registry
  targetNamespace: kube-system
  valuesContent: |-
    service:
      name: registry
      type: LoadBalancer
    persistence:
      enabled: true
    tlsSecretName: registry-ingress-tls
```

我使用的是最新的 K3s(v 0 . 6 . 1)。helm chart 的 CRD API 版本略有变化。*(感谢 K3s 团队，他们在我为 V0.6.0 报告后立即修复了舵图问题，并发布了 V0.6.1)*

应用它。我们正在运行 docker-registry。通过以下命令验证 docker 注册表是否正常工作，

```
curl -ks https://ubuntu:5000/v2/_catalog
```

由于我使用的是 K3s 负载平衡器，主机上的端口 5000 是可用的。

## 3.使用 Docker 推送图像

在我们将图像发送到私有注册表之前。我们需要使 CA 对 Docker 引擎是可信的。使用 CA 的证书更新设置，`myca.pem`

```
sudo mkdir -p /etc/docker/certs.d/ubuntu:5000
sudo cp myca.pem /etc/docker/certs.d/ubuntu:5000/ca.crt
```

现在我们可以构建映像并将映像推送到我们设置的私有注册中心。

## 4.使用 K3s 集群中的映像

类似地，要使用私有 Docker 注册表中的 K3s 映像，首先需要信任 CA 的证书。似乎目前 containerd 引擎无法从配置中定义受信任的 CA。

但是，可以通过将 CA 放入主机系统的可信 CA 链中来解决这个问题。运行以下命令。

```
sudo mkdir -p /usr/local/share/ca-certificates/myregistrysudo cp registry/myca.pem /usr/local/share/ca-certificates/myregistry/myca.crtsudo update-ca-certificates
```

注意，特定目录下的证书必须以扩展名`crt`命名。重新启动 K3s 服务以使更改生效。

现在我们设置注册表，里面的图片可以在 K3s 集群中引用并运行。