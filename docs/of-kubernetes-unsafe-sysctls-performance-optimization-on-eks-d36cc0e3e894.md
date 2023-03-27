# EKS Kubernetes 不安全系统及性能优化

> 原文：<https://itnext.io/of-kubernetes-unsafe-sysctls-performance-optimization-on-eks-d36cc0e3e894?source=collection_archive---------0----------------------->

![](img/f15fc8b763c4128a8f5a01e684256587.png)

# 什么是不安全的系统？

在诸如 Kubernetes 之类的容器化工作负载编排环境中运行时，内核参数(参见运行`sudo sysctl -a`的完整列表)可以分为“安全”和“不安全”两类。

一个“安全”的 sysctl 仅仅意味着所述内核参数是“命名空间”的。即，一个内核名字空间(容器)内的值不一定反映另一个内核名字空间(容器)内的值，因此不会干扰底层容器化机器的操作方式。安全 sysctl 的一个例子是本地 ip 端口范围(`net.ipv4.ip_local_port_range`)

相比之下,“不安全”的 sysctl 没有命名空间，如果从 pod 或容器中修改，可能会导致问题。在这种情况下，Kubernetes 集群上所有不安全的 sysctls 都被默认禁用。

# 为什么启用 usafe sysctls？

很棒的问题！答案很简单，并不是所有不安全的系统都是真正&完全“不安全”的。事实上，随着新版内核和对命名空间的新支持，越来越多的 sysctls 从“不安全”过渡到“安全”类别。一些仍处于不安全类别的内核参数可以进行调整，以便在仔细考虑整个集群的整体视图时，实现整个应用程序的性能优势。

根据个人经验，一个这样的例子是一个名为`net.ipv4.tcp_tw_reuse`的内核参数。这个参数允许内核重用处于`TIME-WAIT`状态的 TCP 套接字。默认情况下，重用这样的套接字是禁用的。对于服务于大量流量的 web 服务器来说，在任何给定的时间点发现成千上万个套接字处于这种状态并不罕见:

```
$ **ss -alnpt | grep TIME-WAIT**
...
...
TIME-WAIT   0        0               127.0.0.1:45448           127.0.0.1:7142                                                                                   
TIME-WAIT   0        0               127.0.0.1:42242           127.0.0.1:7142                                                                                   
TIME-WAIT   0        0               127.0.0.1:42554           127.0.0.1:7142                                                                                   
TIME-WAIT   0        0               127.0.0.1:44324           127.0.0.1:7142                                                                                   
TIME-WAIT   0        0               127.0.0.1:41880           127.0.0.1:7142                                                                                   
TIME-WAIT   0        0               127.0.0.1:43522           127.0.0.1:7142                                                                                   
...
...
```

不重用这些套接字会导致网络服务器连接缓慢和性能下降。当您在内核的`dmesg`输出中发现以下消息时，您可能会遇到性能问题:

```
[250516.270111] TCP: request_sock_TCP: Possible SYN flooding on port 7142\. Sending cookies.  Check SNMP counters.
```

# 我们如何在 EKS 启用不安全的系统？

虽然 kubernetes 的官方文档中有很多关于启用 usafe sysctls 的信息，但我在 EKS 上启用它们时还是遇到了一些细微差别(因此有了这篇博文！).

第一步是在 EKS 集群的特定节点组(或所有节点组)中允许一组(子)不安全系统。这必须在创建节点组时完成，因为这是在`kubelet.`中的设置，如果您使用的是[管理的 EKS 节点组](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html)，则没有直接的方法来完成此操作。因此，我必须在 EKS 创建一个新的*非托管*节点组。使用 [eksctl](https://eksctl.io) 管理集群非常简单，我所要做的就是在 yaml 的新[节点组部分下包含以下](https://eksctl.io/usage/customizing-the-kubelet/)[配置](https://eksctl.io/usage/schema/):

```
**kubeletExtraConfig**:
  **allowedUnsafeSysctls**:
    - "net.ipv4.tcp_tw_reuse"
```

解决了这个问题，下一步是允许相关 [pod 安全策略](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)的规范中的不安全 sysctl:

```
**spec**:
  **allowedUnsafeSysctls**:
  - net.ipv4.*
```

(注意这里通配符的使用，它有点随意，但只是为了说明这是可能的。实际上，我们应该将这个列表限制在最少的必需系统中

一旦以上两项完成，您就可以在 pod 中自由地微调这些内核参数。只需设置您的`deployment.spec.template.spec.**securityContext**`(或者如果您直接使用 pod，则`pod.spec.**securityContext**`为:

```
**sysctls**:
  - **name**: net.ipv4.tcp_tw_reuse
    **value**: "1"
```

如果你使用 [helm](https://helm.sh) 来部署你的应用程序，你引导应用程序的默认 helm 图表应该在`values.yaml`中有一个叫做`podSecurityContext`的部分，你可以在那里插入相同的代码片段。

最后但并非最不重要的是，一旦你的应用程序部署了上述调整，你可以使用`sudo sysctl -a`检查这些参数是否打开

Et，瞧！您应该有一个重用 tcp 套接字的高性能 web 服务器！