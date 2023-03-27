# Kubernetes 集群内部的安全措施

> 原文：<https://itnext.io/security-measures-inside-kubernetes-cluster-e43b3a328473?source=collection_archive---------0----------------------->

![](img/e3f56344935d8c877aa13ba70d853c17.png)

杰森·登特在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

由于 Kubernetes 已经迅速成为在云中部署代码的最受欢迎的选择之一，我决定写一篇文章来给你一些关于安全措施的见解。此外，在 Kubernetes 的生产集群中，您应该考虑那些隐藏的东西。

我假设这篇文章的读者熟悉 Kubernetes 结构的基本知识。本文主要是为 Kubernetes 开发人员或 DevOps 工程师编写的，他们希望在产品上部署他们的服务，并且正在寻找一些安全技巧来确保他们的服务在部署后仍然安全。

应该考虑一些关于集群本身的安全措施，尤其是作为一名集群管理员，当您想从头开始创建一个集群时。甚至在托管的 Kubernetes 服务上，如 GKE、EKS、AKS 等。你需要小心谨慎。虽然在大多数情况下，托管 Kubernetes 服务可以帮助您保护您的堆栈，如集群网络和对集群 API 服务器的访问。

> 如果你认为技术可以解决你的安全问题，那么你不了解问题，也不了解技术。
> 
> 布鲁斯·施奈尔，应用密码学

**集装箱安全**

经常犯的一个大错误是认为容器是完全隔离的。

对 Roo 说不:用 root 运行的一些例外情况可能是:

*   您的容器需要编辑主机系统，例如修改内核的配置。
*   容器需要绑定到节点上的特权端口，例如端口 80 上的 nginx 服务，尽管这可以通过 Kubernetes 中服务抽象中的端口映射来避免。
*   在运行时将软件安装到容器中。在某些情况下，传统的包管理系统可能要求 root 在某个位置运行或存储文件，尽管这种方法是不好的做法，因为在运行时安装的任何代码都没有进行漏洞或其他策略要求的扫描。

**扫描 docker 映像:**为了扫描 Docker 映像中的漏洞，您需要利用容器映像扫描器来检查映像中包含的包。图像扫描仪让您知道是否有任何已知的漏洞，包括在这些软件包或没有。
像 docker Trusted Registry、Google Container Registry 甚至 Red Hat Container Catalog 这样的知名 Docker 注册表已经有了图像扫描结果。因此，如果你想使用这些资源中的任何 docker 图像，你可以在那里检查扫描结果。

**容器实用程序:**确保你的 docker 文件中有尽可能少的实用程序，这将使运行中的容器对试图破坏它的攻击者来说用处不大。例如，假设我们在一个容器的文件中有一些可读的凭证，如果容器不包括像*或 ***more*** 或甚至 ***shell*** (像 ***sh*** 或 ***bash*** )这样，读取凭证就会困难得多。有时删除这些实用程序会使故障诊断更加困难，但这可能是一种权衡，看您是否需要这些程序。*

***图像版本:**通过使用语义版本标签来引用 pod 中的图像，或者即使可以通过其唯一的摘要而不是标签来引用它，这总是有助于您知道容器图像的哪个特定版本正在部署。如果我们忽略图像标记或摘要，图像版本将被标记为**最新**，强烈建议在生产中避免使用**最新**版本。因为当您想要回滚到前一个版本时，很难准确地跟踪正在运行的代码以及要使用的版本。*

***吊舱安全***

*Kubernetes 提供了两种 pod 级安全策略机制，第一种是允许我们限制一个 pod 内的进程可以做什么(安全上下文)，第二种是 pod 如何相互通信(网络策略)。*

***安全上下文:**我们可以在安全上下文中定义 pod 或容器级别的权限和访问控制设置。例如，假设您希望通过 ***runAsUser*** 设置让 pod 中的所有容器在 ***1001*** user 下运行，或者通过将***allowprivilagessescalation***设置为 false 来阻止 ***setuid*** 二进制文件更改用户 id。下面的示例说明了如何在 pod 中定义安全上下文:*

```
*apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1001
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: **false
**      runAsUser: 1001*
```

***网络策略:**默认情况下，允许所有类型的传入和传出流量，但您可以通过网络策略控制允许 pod 如何相互通信。*

*网络策略为您的集群增加了一个安全层，假设外部攻击者能够通过任何安全漏洞到达集群网络，网络策略可以阻止该攻击者向 pod 内部运行的应用程序代码发送流量。*

*此外，如果某个容器不知何故遭到破坏，攻击者通常会尝试探索网络以转移到其他容器，这通过限制地址、端口和容器可能要困难得多。看一下这个示例，了解如何在 pod 之间定义网络策略:*

```
*apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978*
```

***机密管理***

*建议十二因素应用程序将配置信息作为环境变量传递。这允许我们将配置从代码中分离出来，这在我们需要在不同的环境中运行代码时非常有用，比如 dev、test、acc 和 prod。*

*Kubernetes 为此提供了秘密资源，但有不同的使用方式。默认情况下，机密值与其他配置信息一起存储在 etcd 数据库中，尽管 base64 编码使内容不可读，但当您可以使用 base64 解码对其进行解码时，它不会对其进行加密。不知何故，我可以说 base64 对攻击者来说是纯文本。
看这个例子，用户名和密码是加密的，你可以很容易地解密 *:**

```
*apiVersion: v1
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: { ... }
  name: mysecret
type: Opaque*
```

*正如您可能猜到的那样，在 etcd 中存储加密值不够安全。一些第三方系统专门用于存储机密和敏感值，如 HashiCorp Vault。
这些第三方通常提供与多个后端机密存储的集成，并且还可以控制对特定机密的访问，即只有特定的容器可以访问特定的机密。*

> *如果你想保守秘密，你应该对自己隐瞒*
> 
> *乔治·奥威尔，1984*

*存储秘密变量的另一种常见方式是将它们存储在文件中，Kubernetes 支持通过卷挂载将秘密传递到 pods 中。容器化的代码需要从这些文件中读取值。如果装入的卷是一个临时文件系统，这意味着文件没有写入磁盘，而是保存在内存中，并且文件中的值不能通过 docker inspect 或 kubectl describe 访问。通过这个例子，您可以了解如何将秘密存储到文件中:*

```
*apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: **true**
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username*
```

***总结***

*Kubernetes 的安全性很复杂。不可能在一篇文章中涵盖所有主题。在这篇文章中，我打算介绍开发人员常用的安全措施，请记住在集群上部署时要遵循和阅读良好的实践。从我的建议开始吧。*

*我强烈推荐阅读 Liz Rice 和 Michael Hausenblas 撰写的 Kubernetes Security，其中涵盖了所有这些主题，并且提供了更详细的信息。*