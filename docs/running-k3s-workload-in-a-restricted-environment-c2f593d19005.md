# 在受限环境中运行 K3S 工作负载

> 原文：<https://itnext.io/running-k3s-workload-in-a-restricted-environment-c2f593d19005?source=collection_archive---------1----------------------->

![](img/a9dd9311162296bda4e71f7dda451779.png)

我需要在受限环境中运行一些监控。它是受限制的，首先，它有气隙，没有互联网连接。其次，环境是锁定的，不能安装额外的依赖模块。

轻量级的 Kubernetes 发行版 K3s 及其独立的依赖项完美地满足了这一需求。本文将展示我是如何在上述受限环境中设置普罗米修斯的。

## 在 airgap 服务器中安装 K3s

K3s 的气隙设置在[有很好的记录](https://rancher.com/docs/k3s/latest/en/installation/airgap/)。下面是需要下载到面向 internet 的服务器上的包和脚本的快速命令列表。

```
curl -LO [https://github.com/k3s-io/k3s/releases/download/v1.21.1%2Bk3s1/k3s-airgap-images-amd64.tar.gz](https://github.com/k3s-io/k3s/releases/download/v1.21.1%2Bk3s1/k3s-airgap-images-amd64.tar.gz)curl -LO [https://github.com/k3s-io/k3s/releases/download/v1.21.1%2Bk3s1/k3s](https://github.com/k3s-io/k3s/releases/download/v1.21.1%2Bk3s1/k3s)curl -sfL [https://get.k3s.io](https://get.k3s.io) > install.sh
chmod a+rx install.sh
```

将这些文件复制到受限节点。我的 K3s 设置在一个节点中。运行以下命令准备文件，

```
sudo mkdir -p /var/lib/rancher/k3s/agent/images/
sudo cp k3s-airgap-images-amd64.tar.gz /var/lib/rancher/k3s/agent/images/

sudo cp k3s /usr/local/bin
sudo chmod a+rx /usr/local/bin/k3scd /var/lib/rancher/k3s/agent/images/
sudo gunzip k3s-airgap-images-amd64.tar.gz
```

安装 K3s，

```
INSTALL_K3S_SKIP_DOWNLOAD=true INSTALL_K3S_EXEC="--write-kubeconfig-mode 644"  INSTALL_K3S_SKIP_SELINUX_RPM=true INSTALL_K3S_SELINUX_WARN=true ./install.sh
```

请注意，底层 CentOS 禁用了 SELinux，因此我们安装 K3s 时没有 SELinux 选项。这避免了安装额外的 rpm。

K3s 现在已经上线运行了，就这么简单。

## 填充注册表容器图像

让我们为 airgap K3s 映像使用一个更通用的解决方案，为 K3s 设置一个私有注册表。

然而，这似乎是一个鸡和蛋的问题。注册表本身需要首先填充容器映像…

我们在这里可以做的是使用 containerd 命令行工具`ctr`，将容器映像导入本地文件系统。

在没有设置 Docker 守护进程的情况下，在面向互联网的服务器上安装`skopeo`工具，下载私有注册中心的容器映像，

```
skopeo copy docker://docker.io/library/registry:2 docker-archive:./registry.tar:library/registry:2
```

上面的命令基本上将容器映像从 docker hub 复制到 docker tar 格式文件中。请注意，我们在 tar 文件名后面附加了图像名称，以便让 tar 文件内容具有图像的名称，否则在稍后导入 containerd 时，未命名的图像将被忽略。

将 tar 文件传输到受限的 K3s 服务器，将其加载到 containerd 文件系统中

```
sudo $(which k3s) ctr images import registry.tar
```

如果您列出图像，您将看到注册表图像现在是可用的。

```
sudo $(which k3s) crictl images | grep registrydocker.io/library/registry                               2                      1fd8e1b0bb7ef       26.8MB
```

我们准备在 K3s 中运行私有注册表。但在此之前，让我们准备必要的工具。

## 填充工具图像

我们需要工具，比如`skopeo`，将工作负载映像加载到私有注册表中。

由于环境是封闭的和受限制的，我们不能自由地安装所需的工具和所有的依赖模块或库。然而，事实上，我们不一定需要搞乱主机 OS，我们可以将容器映像用于我们的工具。

让我们将图像工具构建为一个容器图像。创建以下 Dockerfile 文件，

```
FROM alpine
RUN  apk --no-cache add curl && \
     apk --no-cache add skopeo --repository=[http://dl-cdn.alpinelinux.org/alpine/edge/community](http://dl-cdn.alpinelinux.org/alpine/edge/community)
```

选择 alpine 作为基础映像，添加来自 edge 社区 repo 的`skopeo`包。

用 podman 来构建，不需要后台像 Docker 一样运行守护进程。

```
podman build -t tools:v1.0 .
```

使用相同的方法将本地映像复制到一个 tar 文件中，

```
skopeo copy containers-storage:localhost/tools:v1.0 docker-archive:./tools.tar:tools:v1.0
```

传输文件，并导入到容器中，

```
sudo $(which k3s) ctr images import tools.tar
```

现在，注册表和工具的容器映像都准备好了。

## 在 K3s 中运行私有注册表

我们将使用 htpasswd 身份验证、启用 https、持久化部署来运行私有注册表。

***创建命名空间***

```
kubectl create ns registry
```

***创建 htpasswd 秘密***

在您拥有完全控制权的服务器上安装 htpasswd 工具，或者使用以下 golang 代码片段来准备 htpasswd.txt 文件

```
encrypted, err := bcrypt.GenerateFromPassword([]byte("password"), bcrypt.DefaultCost)
```

htpasswd.txt 文件应该如下所示，

```
admin:{{ .encrypted }}
```

将 htpasswd 文件传输到 K3s 节点，创建密码

```
kubectl -n registry create secret generic htpasswd --from-file=htpasswd=htpasswd.txt
```

***创建证书和秘密***

使用 cfssl 创建证书和密钥对，如[我过去的论文](/setup-a-private-registry-on-k3s-f30404f8e4d3)中所述，或者使用您最喜欢的工具。请注意，证书中的 SAN 需要包括 K3s 节点的主机名。您还可以考虑将 K8s 服务名称添加为 SAN 名称的一部分。

创造 K8s 秘密，

```
kubectl -n registry create secret tls tls-registry --cert=registry.pem --key=registry-key.pem
```

***创建 PVC***

应用以下 YAML 创建 PVC

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-registry
  namespace: registry
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

***创建部署***

应用以下部署，

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: registry
  namespace: registry
  labels:
    app: registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: registry
  template:
    metadata:
      labels:
        app: registry
    spec:
      containers:
      - name: registry
        image: registry:2
        **imagePullPolicy: IfNotPresent**
        ports:
        - containerPort: 5000
        env:
        - name: REGISTRY_AUTH
          value: htpasswd
        - name: REGISTRY_AUTH_HTPASSWD_REALM
          value: Registry Realm
        - name: REGISTRY_AUTH_HTPASSWD_PATH
          value: /auth/htpasswd
        - name: REGISTRY_HTTP_TLS_CERTIFICATE
          value: /certs/tls.crt
        - name: REGISTRY_HTTP_TLS_KEY
          value: /certs/tls.key
        volumeMounts:
        - name: registry
          mountPath: /var/lib/registry
        - name: tls-registry
          mountPath: /certs
        - name: htpasswd
          mountPath: /auth
      volumes:
      - name: registry
        persistentVolumeClaim:
          claimName: pvc-registry
      - name: tls-registry
        secret:
          secretName: tls-registry
      - name: htpasswd
        secret:
          secretName: htpasswd
```

查看环境变量如何用于注册表配置。

注意，我们将 imagePullPolicy 设置为 IfNotPresent。由于我们已经将图像填充到文件系统中，kubelet 不会从互联网上获取图像。

***揭露服务***

应用以下 K3s 负载平衡器服务，

```
apiVersion: v1
kind: Service
metadata:
  name: svc-registry
  namespace: registry
spec:
  type: LoadBalancer
  ports:
    - name: http
      port: 5000
      targetPort: 5000
  selector:
    app: registry
```

***信任 CA***

让操作系统信任注册表的 CA，myca.pem

```
sudo cp myca.pem /etc/pki/ca-trust/source/anchors/myca.crt
sudo update-ca-trust
```

现在，如果您对注册表进行 curl 操作，您应该会看到一个空的图像列表，

```
curl -u admin:password [https://centos7:5000/v2/_catalog](https://centos7:5000/v2/_catalog)
{"repositories":[]}
```

## 配置专用 K3S 注册表

一旦注册表开始运行，我们就可以像在配置文件`/etc/rancher/k3s/registries.yaml`中一样更新 k3s 注册表设置，

```
mirrors:
  quay.io:
    endpoint:
      - "[https://centos7:5000](https://centos7:5000)"
  docker.io:
    endpoint:
      - "[https://centos7:5000](https://centos7:5000)"
  k8s.gcr.io:
    endpoint:
      - "[https://centos7:5000](https://centos7:5000)"
configs:
  "centos7:5000":
    auth:
      username: admin
      password: password
    tls:
      cert_file: /etc/rancher/k3s/certs/registry.pem
      key_file: /etc/rancher/k3s/certs/registry-key.pem
      ca_file: /etc/rancher/k3s/certs/myca.pem
```

请注意镜像是如何定义的，以便将外部注册表映射到 K3s 私有注册表。重新启动 K3s 服务以使更改生效。

## 填充工作负载映像

以 Prometheus-operator 为例，我们需要以下图像

```
quay.io/prometheus/prometheus:v2.26.0
quay.io/prometheus/node-exporter:v1.1.2
quay.io/prometheus/blackbox-exporter:v0.18.0
quay.io/prometheus/alertmanager:v0.21.0
quay.io/prometheus-operator/prometheus-operator:v0.47.0
quay.io/prometheus-operator/prometheus-config-reloader:v0.47.0
quay.io/brancz/kube-rbac-proxy:v0.8.0
k8s.gcr.io/kube-state-metrics/kube-state-metrics:v2.0.0
jimmidyson/configmap-reload:v0.5.0
grafana/grafana:7.5.4
directxman12/k8s-prometheus-adapter:v0.8.4
```

下载 skopeo 作为 tar 文件的图像。举个例子，

```
skopeo copy docker://quay.io/prometheus/prometheus:v2.26.0 docker-archive:./prometheus.tar:prometheus:v2.26.0
```

将 tar 文件传输到 K3s 节点。首先，将工具映像作为 Pod 运行，

```
kubectl run loader --image=tools:v1.0 --command -- sh -c 'while true; do sleep 10; done'
```

将镜像加载到 K3s 中运行的私有注册表中。举个例子，

```
kubectl cp prometheus.tar loader:/tmpkubectl exec -i loader -- skopeo copy --dest-creds admin:password --dest-tls-verify=false --insecure-policy docker-archive:/tmp/prometheus-operator.tar docker://centos7:5000/prometheus-operator/prometheus-operator:v0.47.0
```

一旦所有映像都被填充到 K3s 私有注册表中，您就可以开始部署 Prometheus operator 网站中记录的工作负载。