# 完成循环(K8s)

> 原文：<https://itnext.io/completing-the-kubernetes-setup-7462383ab3c?source=collection_archive---------7----------------------->

## 通过一些额外的附加组件使 Kubernetes 集群发挥作用

![](img/30661624e6714d8ee1d9c0f18b5725b1.png)

JR Korpa 在 [Unsplash](https://unsplash.com/s/photos/monitoring?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

在上一篇文章中，我解释了如何设置 HA K8s 集群。

[](https://medium.com/@anisinanaj/on-premise-ha-kubernetes-cluster-15e41f18bd12) [## 内部 HA Kubernetes 集群

### 那么为什么是 9 台服务器呢？到目前为止，我们可以把构成星团的部分分成 3 个部分。我们有存储，控制…

medium.com](https://medium.com/@anisinanaj/on-premise-ha-kubernetes-cluster-15e41f18bd12) 

但这只是 Kubernetes 的裸装。

# 少了什么

设置好集群后，除了 K8s，上面不会有太多。为了让它工作，我们需要部署一些额外的应用程序。

## CNI(集装箱网络接口)

第一个也是最重要的一个是 CNI。我们将 Kubernetes 配置为使用法兰绒，所以现在我们必须将它添加到集群中，否则什么都不会起作用。事实上，检查节点将导致“未就绪”。

```
kubectl get nodes -o wide
```

添加法兰绒的命令非常简单。

```
kubectl apply -f [https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml](https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml)
```

> 要从您自己的计算机使用`*kubectl*`，请从第一个控制平面将*/etc/kubernetes/admin . conf*复制到 *~/。kube/配置*

## 集装箱存储接口

现在我们已经配置了网络，我们可以添加存储资源调配器了。

在[上一篇文章](https://medium.com/@anisinanaj/on-premise-ha-kubernetes-cluster-15e41f18bd12)中，我假设有一个 Ceph 集群，因此我将为此部署 provisioners，记住它已经为 K8s 进行了配置。

我将从克隆存储扩展的官方存储库开始。

```
git clone git@github.com:kubernetes-incubator/external-storage.git
```

我将重点关注 *ceph* 文件夹，其中有两个不同的资源调配器，一个用于 RBD，代表数据块存储，另一个是 FS，只是 iSCSI。

我喜欢两者都有，因为它们以不同的方式工作，并且在不同的情况下都很有用。

**RBD** 首先要做的是创建两个秘密资源，它们将保存访问 Ceph 集群的密钥。文件已经存在，我们只需要填写`./ceph/rbd/examples/secrets.yaml`中的信息

该文件应该如下所示。默认情况下，名称空间是`kube-system`，但是我把它改成了`storage`，如果你按照我的例子做，请确保也改变了`./ceph/rbd/deploy/rbac/rolebinding.yaml`和`./ceph/rbd/deploy/rbac/clusterrolebinding.yaml`上的名称空间

```
apiVersion: v1
kind: Secret
metadata:
  name: ceph-admin-secret
  namespace: **storage**
type: "kubernetes.io/rbd"
data:
  key: **QVFDMHFJWmNSay9ZSnhBQXlhSFhTOFo5a3hKODE1ZUdQWVRYYmc9PQ==**
---
apiVersion: v1
kind: Secret
metadata:
  name: ceph-secret
  namespace: **storage**
type: "kubernetes.io/rbd"
data:
  key: **QVFCakxMNWNlcUI4QmhBQTd2UUNDNFNSaTk0ZDgvMTNXOUdMemc9PQ==**
```

要从 Ceph 集群获取密钥，请在它的一个节点上运行这些命令。

```
ceph auth get-key client.admin | base64ceph auth add client.kube mon 'allow r' osd 'allow rwx pool=kube'
ceph auth get-key client.kube | base64
```

最后要编辑的是`./ceph/examples/class.yaml`，应该是下图这样。在监视器下设置您的 Ceph 集群 IP 地址，并引用上面创建的秘密。

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: rbd
provisioner: ceph.com/rbd
parameters:
  monitors: **116.202.35.140:6789,116.202.35.141:6789,116.202.35.142:6789**
  pool: **kube**
  adminId: admin
  adminSecretNamespace: **storage**
  adminSecretName: **ceph-admin-secret**
  userId: kube
  userSecretNamespace: **storage**
  userSecretName: **ceph-secret**
  imageFormat: "2"
  imageFeatures: layering
```

最后，我们需要将配置文件应用到集群。

```
kubectl apply -f ./ceph/deploy/rbac
kubectl apply -f ./ceph/examples/secrets.yaml
kubectl apply -f ./ceph/examples/class.yaml
```

您可以使用示例下的`claim.yaml`和`test-pod.yaml`文件来测试 RBD。

**CephFS** 现在设置 CephFS 的步骤是相似的。首先，我们将创建这个秘密，或者使用上面创建的秘密，并将其复制到`cephfs`名称空间中。

```
apiVersion: v1
kind: Secret
metadata:
  name: ceph-admin-secret
  namespace: **cephfs**
type: "kubernetes.io/rbd"
data:
  key: **QVFDMHFJWmNSay9ZSnhBQXlhSFhTOFo5a3hKODE1ZUdQWVRYYmc9PQ==**
```

对于 CephFS，admin-secret 是唯一需要的。

现在来创建角色并应用这个秘密。

```
kubectl apply -f ./cephfs/deploy/rbac
kubectl apply -f ./cephfs/example/secret.yaml
```

最后要编辑的是在`./cephfs/example/class.yaml`中找到的存储类配置，方法是将正确的 IP 地址添加到 Ceph 集群中，并引用上面创建的秘密。

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: cephfs
provisioner: ceph.com/cephfs
parameters:
    monitors: **116.202.35.140:6789,116.202.35.141:6789,116.202.35.142:6789**
    adminId: **admin**
    adminSecretName: **ceph-secret-admin**
    adminSecretNamespace: **cephfs**
    claimRoot: /pvc-volumes
```

然后我们就可以应用它了。

```
kubectl apply -f ./cephfs/example/class.yaml
```

仅此而已。

## 入口和证书管理器

开箱即用，K8s 集群不处理 HTTP(s)请求。我们需要配置一个*反向代理*和/或*负载平衡器*。这些资源被称为*入口控制器*。

入口控制器的工作是根据相关的入口规则将流量路由到所需的 Pod。

有不同的解决方案可用作入口控制器， [Nginx](https://www.nginx.com/products/nginx/kubernetes-ingress-controller) ， [HAProxy](https://github.com/haproxytech/kubernetes-ingress) ， [Istio](https://istio.io/docs/tasks/traffic-management/ingress/) ， [Traefik](https://github.com/containous/traefik) 。Traefik 附带了 Let's Encrypt incorporated。您也可以使用多个控制器。

**Nginx** 我是 Nginx 的粉丝，所以我要跟它走。按照以下步骤安装 Nginx

```
helm install stable/nginx-ingress --name nginx-ingress-controller
```

现在部署一个示例项目来检查入口控制器是否在工作

```
kubectl apply -f [https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/docs/tutorials/acme/quick-start/example/deployment.yaml](https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/docs/tutorials/acme/quick-start/example/deployment.yaml)kubectl apply -f [https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/docs/tutorials/acme/quick-start/example/service.yaml](https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/docs/tutorials/acme/quick-start/example/service.yaml)
```

现在，我们将为上述内容部署入口，记住将主机名编辑为我们自己的主机名。

```
kubectl create --edit -f [https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/docs/tutorials/acme/quick-start/example/ingress.yaml](https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/docs/tutorials/acme/quick-start/example/ingress.yaml)
```

现在转到这个领域，您应该会看到 **Kuard** 。

> 如果你在使它工作上有困难，那可能是因为入口控制器不知道它是否有可用的公共 IP 以及如何使用它们。尝试将“外部 IP”添加到包含集群所有公共 IP 阵列的入口控制器中。MetalLB 会自动处理这些问题。

**Let's Encrypt** 据我所知，Kubernetes 使用最多的 Let's Encrypt 实现是 [**cert-manager**](https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html#installing-with-helm) 。

它由 [JetStack](https://www.jetstack.io) 维护和开发。该安装在许多网站和 [Github](https://github.com/jetstack/cert-manager) 上被广泛记录。

首先要做的是创建自定义资源定义。

```
kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.9/deploy/manifests/00-crds.yaml
```

然后我们将创建一个专用的名称空间。

```
kubectl create namespace cert-manager
```

以下命令将禁用资源验证。

```
kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true
```

然后安装它，我们需要添加 Helm repo 并继续安装。

```
helm repo add jetstack [https://charts.jetstack.io](https://charts.jetstack.io)
helm repo update
helm install \
  --name cert-manager \
  --namespace cert-manager \
  --version v0.9.1 \
  jetstack/cert-manager
```

最后，我们需要创建 2 个证书颁发者，一个用于暂存，一个用于生产。唯一需要更新的是邮件。

```
kubectl create --edit -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/docs/tutorials/acme/quick-start/example/staging-issuer.yamlkubectl create --edit -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/docs/tutorials/acme/quick-start/example/production-issuer.yaml
```

这应该给我们一个工作环境。

现在，为了让我们的 **Kuard** 例子使用 Let's Encrypt，我们应该重新部署它的**入口**。下面的文件包含配置中的`tls`键，它指定了证书应该工作的域。还要注意决定使用哪个发行者的注释。

```
kubectl create --edit -f [https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/docs/tutorials/acme/quick-start/example/ingress-tls-final.yaml](https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/docs/tutorials/acme/quick-start/example/ingress-tls-final.yaml)
```

## 包管理器

幸运的是，Kubernetes 世界已经发展到有了一个包管理器。事实上， [Helm](https://helm.sh) 正是如此，设置起来非常简单直接。

在您的系统上安装`helm`

```
brew install kubernetes-helm
```

通过创建并应用该文件，授予它访问群集的权限

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

假设我们称之为 *rbac-config.yml*

```
kubectl create -f rbac-config.yaml
helm init --service-account tiller --history-max 200
```

可以走了！

# 问题

## Resolve.conf

有时，根据云提供商的不同，你的`resolve.conf`文件可能有 3 条以上的记录。这可能是一个问题，因为在这种情况下， **kube-dns** 和 **coredns** 都无法解析域名。这不是这两个服务本身的问题，但它与 Linux 的 libc 有关，正如这里所说的

## 限制

创建本地集群有不同的限制，首先是可伸缩性。扩展这些服务器要困难得多，您必须购买硬件或额外的节点等等。

随着可扩展性而来的是**成本**，因为你有物理机器，你不能按需缩减。这意味着即使您只使用了基础设施的一小部分，您也要为整个基础设施付费。

**负载均衡**是另一个极限。配置 MetalLB 或类似的东西并不总是可行的。它可以在实际的 K8s 节点上配置，但是这会增加网络和集群本身的开销。如果要在 K8s 群集之外进行管理(就像我们对存储所做的那样)，我们至少还需要 3 台服务器来保持高可用性。对于控制平面请求和应用请求，需要分别在控制平面和工作节点上进行负载平衡。解决这个问题的另一种方法是使用 DNS 级别的负载均衡器，它可以工作，但是没有那么快。

# 监控和管理

这里有很多部分，事情可能会失败。这就是为什么我们需要持续监控系统，以便在出现问题时快速采取行动。除了监控工具，我们还需要让自己更容易管理一切。

## 舵

帮助我们安装和更新完成群集所需的工具和软件。

如上所述进行安装。

## 大牧场主

[Rancher](https://rancher.com/products/rancher) 是在同一个地方管理一个或多个 Kubernetes 集群的工具。

它可以安装在任何地方，不一定要安装在集群上。如果您在群集上安装它，它会识别它，并进行自我配置，使您可以管理该群集。

你可以像下面这样把它安装成 docker 容器

```
docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
```

或者在 K8s 集群上安装它。更新主机名和电子邮件，以便正确创建证书。(这假设您正在使用证书管理器)

```
helm repo add rancher-latest [https://releases.rancher.com/server-charts/latest](https://releases.rancher.com/server-charts/latest)helm install rancher-latest/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=me@example.org
```

部署完成后你就可以走了。

## 普罗米修斯/格拉法纳

现在我们有了牧场主，我们可以安装普罗米修斯和格拉夫纳。这两种工具都可以监控群集的硬件资源，并且可以配置为在出现问题时发送警报。

这些也可以通过头盔安装。这正是牧场主所做的。它内置了 Helm，因此我们可以在通过 Rancher 管理的所有集群上安装软件包。

通过 Rancher 部署它们，将指标集成到 Rancher 的面板中。

## 发信号

从集群进行监控和警报是好的，但这还不够。如果由于某种原因，整个集群不可访问，可能是巨大的网络故障，会发生什么？因此，我们也需要外部监控。

为此，我使用了[节点查询](https://nodequery.com)和[状态蛋糕](http://statuscake.com)。在监控服务器资源方面，两者是相似的。您在主机中安装一个代理，它会定期发送系统状态。

Status Cake 还可以用于 ping 不同的服务，并将其配置为在它们没有响应时发送警报。可以配置各种错误，40x、50x 等，以及*超时*。

## 管理

其他值得注意的管理工具当然是 [Kubernetes 仪表盘](https://github.com/kubernetes/dashboard)和 [K9s](https://k9ss.io)

要安装**仪表板**，请将其配置应用到集群，然后创建凭证，如下所示

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yamlkubectl create clusterrolebinding kubernetes-dashboard \
  --clusterrole=cluster-admin \
  --serviceaccount=kube-system:kubernetes-dashboard
```

要使用它，首先要获得访问令牌

```
kubectl get secrets --namespace kube-system **#to find how it's called**kubectl describe secrets --namespace kube-system **kubernetes-
dashboard-token-9gz66**
```

因为默认情况下它不会通过入口暴露，所以您可以只使用`kubectl proxy`然后打开这个[链接](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default)。

K9s 安装非常简单，在 Mac 上如下

```
brew install derailed/k9s/k9s
```

要使用它，只需从终端运行`k9s`。

系列文章

[](https://medium.com/@anisinanaj/on-premise-ha-kubernetes-cluster-15e41f18bd12) [## 本地 HA Kubernetes 集群

### 在建立生产 Kubernetes 集群时克服基础设施限制

medium.com](https://medium.com/@anisinanaj/on-premise-ha-kubernetes-cluster-15e41f18bd12) [](https://medium.com/@anisinanaj/storage-on-kubernetes-efa0a5b4f858) [## Kubernetes 上的存储

### 了解存储在分布式系统和 Kubernetes 中的工作原理。内部存储解决方案

medium.com](https://medium.com/@anisinanaj/storage-on-kubernetes-efa0a5b4f858) [](https://medium.com/@anisinanaj/kubernetes-cluster-networking-ad77eb4f826b) [## Kubernetes 集群网络

### 构建本地 K8s 和 Ceph 集群时处理网络的方法

medium.com](https://medium.com/@anisinanaj/kubernetes-cluster-networking-ad77eb4f826b) 

我希望我涵盖了一切。感谢你有耐心读到这里。
敬请关注更多:)