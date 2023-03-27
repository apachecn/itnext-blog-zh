# 通过审核 Kubernetes 清单来学习

> 原文：<https://itnext.io/learning-auditing-kubernetes-manifests-2a82db24f8a7?source=collection_archive---------8----------------------->

![](img/9aace58ea0fae4121a19edcccfc429db.png)

去年，我在大英博物馆举行的全国 DevOps 会议上发言。我之前已经参观过这个博物馆了，但是说起来那是一次奇妙的经历。此外，我们有几个小时完全属于自己的博物馆。如果你曾经参观过那个地方，你知道我的意思。

反正我也参加了一个关于 [Checkov](https://www.checkov.io/) 的讲座:

> *Checkov 扫描云基础设施配置，在部署之前发现错误配置。*
> 
> *Checkov 使用通用命令行界面来管理和分析跨平台的基础架构代码(IaC)扫描结果，如 Terraform、CloudFormation、Kubernetes、Helm、ARM 模板和无服务器框架。*

演讲结束后，我想试一试。虽然我不是一个超级云用户，但我在我的几个演示中使用了 Kubernetes。我安装了 CLI，在这个[文件夹](https://github.com/hazelcast-demos/zerodowntime/tree/master/infrastructure/kube)上启动，只是为了好玩。有趣的是，我学到了很多*。*

*让我们先快速总结一下:*

```
*checkov -d infrastructure/kube/ --quiet --compact | grep CKV | sort --unique*
```

*该命令输出我违反的所有规则。*

*我知道其中一些，尽管我不打算在我的上下文中修复它们。例如，请求和限制在生产中是有意义的，但是在我的演示中我不关心它们。*

*其他的对我来说是全新的。我很高兴在这里分享我的 TIL(或没有)。*

# *更喜欢摘要而不是标签*

*   **规则* : [CKV_K8S_43 —确保使用摘要选择图像](https://docs.bridgecrew.io/docs/bc_k8s_39)*
*   **严重性*:中等*

*我以前被这只咬过。即使我会保持原样，它也值得一个解释。并且解释为**图像标签不是不可变的**。*

*标签只是指向特定图像的指针。发布者可以将现有标签指向另一个图像。在这种情况下，构建不是等幂的，因为基础映像可能从一个构建变化到另一个构建。*

*对于这个演示，我在本地机器上构建图像，这样保留标签就有意义了。我可以更新依赖项的版本，重新构建，并获得最新构建的映像。*

# *明确禁止权限提升*

*   **规则* : [CKV_K8S_20 —确保集装箱不会以 AllowPrivilegeEscalation 运行](https://docs.bridgecrew.io/docs/bc_k8s_19)*
*   **严重性*:中等*

*这一次的出现有点出乎意料。要理解这个问题，我们需要阅读相关文档:*

> ****allowprivilegescalation****—决定是否允许用户将容器的安全上下文设置为* `*allowPrivilegeEscalation=true*` *。这默认为 allowed，以便不破坏 setuid 二进制文件。将它设置为* `*false*` *可以确保容器的子进程不会获得比其父进程更多的特权。**
> 
> **—* [*Pod 安全策略—权限提升*](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#privilege-escalation)*

*总结一下:**默认情况下，容器的子进程可以获得比其父进程**更多的特权。*

*这是一个相当严重的问题。请注意，Pod 安全策略本身在 1.21 版中已被否决，并将在 1.25 版中被删除。在该版本之前，应该始终将该属性显式设置为`false`。*

# *设置安全上下文*

*   **规则* : [CKV_K8S_30 —确保安全上下文应用于 pod 和容器](https://docs.bridgecrew.io/docs/bc_k8s_28)*
*   **严重性*:低*

*虽然严重程度被标记为低，但我认为这条规则是必不可少的。每个箱或集装箱在货单中可以有一个`securityContext`部分。该部分有许多不同的字段:*

*   *`runAsNonRoot`、`runAsUser`和`runAsGroup`:参见下面的*不作为根*运行*
*   *`seccompProfile`:参见下面的*设置秒补偿轮廓**
*   *`fsGroup`和`fsGroupChangePolicy`*
*   *`seLinuxOptions`*
*   *`supplementalGroups`*
*   *`sysctls`*
*   *`windowsOptions`*

*其中一些在下面的章节中有更详细的描述，但是完整的描述不仅仅是一篇介绍性的博客文章。如果你想潜得更深，请查阅[的相关文档](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#securitycontext-v1-core)。*

*或者，Snykt 写了一篇[很棒的文章](https://snyk.io/blog/10-kubernetes-security-context-settings-you-should-understand/)来解释最重要的问题。*

# *不要以 root 身份运行*

*   **规则* : [CKV_K8S_23 —最小化根容器的接纳](https://docs.bridgecrew.io/docs/bc_k8s_22)*
*   **严重性*:中等*

*我认为这一个是相当众所周知的，但它仍然值得重复。容器是一个利用内核特性的 Linux 进程:*

*   **控制组*限制、说明和隔离资源使用，*例如* CPU、内存、磁盘 I/O、网络等。*
*   **名称空间*划分内核资源，使得一组进程看到一组资源，而另一组进程看到不同的资源*

*虽然两者都提供了进程隔离，但并不是万无一失的。如果以根用户身份运行的容器受到威胁，攻击者可以使用额外的权限进行进一步攻击。要解决此问题，请使用以下代码片段:*

```
*apiVersion: v1
kind: Pod
metadata:
  name: <name>
spec:
    securityContext:
    runAsNonRoot: true
    runAsUser: <user>*
```

# *设置高 UID 用户*

*   **规则* : [CKV_K8S_40 —确保容器以高 UID 运行，以避免主机冲突](https://docs.bridgecrew.io/docs/bc_k8s_37)*
*   **严重性*:低*

*在之前的规则检查中，我们没有以`root`的身份运行，也就是 UID 1。然而，即使使用其他 uid，如果容器受到危害，我们也有在主机系统上冒充另一个用户的风险。为了大幅降低概率，已配置的用户应该只从 UID 10，000 开始。*

```
*apiVersion: v1
kind: Pod
metadata:
  name: <name>
spec:
    securityContext:
    runAsUser: <+10,000>*
```

# *设置 seccomp 配置文件*

*   **规则* : [CKV_K8S_31 —确保 seccomp 配置文件设置为 Docker/Default 或 Runtime/Default](https://docs.bridgecrew.io/docs/bc_k8s_30)*
*   **严重性*:低*

> **安全计算模式(* `*seccomp*` *)是 Linux 内核的一个特性。您可以使用它来限制容器内可用的操作。* `*seccomp()*` *系统调用在调用进程的 seccomp 状态下运行。您可以使用此功能来限制您的应用程序的访问。**
> 
> **默认的* `*seccomp*` *配置文件为使用 seccomp 运行容器提供了一个合理的默认值，并禁用了 300+个系统调用中的大约 44 个。它在提供广泛的应用程序兼容性的同时具有适度的保护性。**
> 
> **—* [*停靠站的安全配置文件*](https://docs.docker.com/engine/security/seccomp/)*

*解决该问题取决于您的 Kubernetes 集群的版本:*

*   *对于 Kubernetes 1.18，对救援的注释:*

```
*apiVersion: v1
kind: Pod
metadata:
  name: <name>
  annotations:
    seccomp.security.alpha.kubernetes.io/pod: "runtime/default"*
```

*   *对于 Kubernetes 1.19+，属性`securityContext`具有一个`secompProfile`:*

```
*apiVersion: v1
kind: Pod
metadata:
  name: <name>
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
    ...*
```

# *结论*

*如果您想在`seccomp`执行策略，请根据您的 Kubernetes 版本，查看与 [Pod 安全许可](https://kubernetes.io/docs/concepts/security/pod-security-admission/)或 [Pod 安全策略](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)相关的文档。*

*总的来说，运行分析/审计工具是深入了解一个主题的好方法。我已经用 Checkov 在 Kubernetes 上做过了，但是这类工具的数量已经足够大，如果你喜欢的话，可以获得即时的知识提升。当然，这不会让你马上成为某个领域的专家，但这是一个很好的开始。*

***更进一步:***

*   *[Checkov:适用于所有人的策略代码](https://www.checkov.io/)*
*   *[Checkov: Kubernetes 政策指数](https://docs.bridgecrew.io/docs/kubernetes-policy-index)*
*   *[用 seccomp 强化 Docker 和 Kubernetes】](https://martinheinz.dev/blog/41)*
*   *[10 Kubernetes 你应该了解的安全上下文设置](https://snyk.io/blog/10-kubernetes-security-context-settings-you-should-understand/)*
*   *[Pod 安全准入](https://kubernetes.io/docs/concepts/security/pod-security-admission/)*

**最初发表于* [*一个 Java 怪胎*](https://blog.frankel.ch/learning-auditing-kubernetes-manifests/)*2022 年 7 月 4 日**