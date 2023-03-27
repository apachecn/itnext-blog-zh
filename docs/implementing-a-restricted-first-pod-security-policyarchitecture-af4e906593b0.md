# 实施安全第一的 Pod 安全策略架构

> 原文：<https://itnext.io/implementing-a-restricted-first-pod-security-policyarchitecture-af4e906593b0?source=collection_archive---------7----------------------->

![](img/ab105b563cf8911e2ab2d706b73d72c0.png)

miosz klin owski 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

# Pod 安全策略

保护 Kubernetes 集群的方法有很多，无论是通过实施网络策略来控制入口/出口流量、基于角色的访问控制，还是多租户。管理在您的集群上运行的内容的最有效方法之一是通过创建 Pod 安全策略。

pod 安全策略定义了一组条件，Pod 必须在这些条件下运行才能在群集上运行。这些条件包括主机级访问、容器可以运行的 uid 范围，甚至是 pod 可以使用的卷。

在本文中，我将展示一个蓝图，通过实现 Pod 安全策略，为您的集群应用安全第一的思想。按照安全或受限优先的思维方式，默认情况下，您将锁定您的集群以运行安全的工作负载，并通过审查，为那些需要特权访问的工作负载设置例外。

# 默认为受限

由于我们是从安全第一的心态来对待我们的集群的，所以我们需要定义一个默认的 Pod 安全策略，该策略扩展到所有经过身份验证的用户和服务帐户。下面提供了一个示例限制 PSP，您可以使用它来开始，然后根据您的需要进行调整:

执行以下命令创建新的 Pod 安全策略:

```
kubectl create -f restricted-psp.yaml
```

接下来，我们要确保此 Pod 安全策略适用于所有服务帐户，因为我们是从安全第一的理念出发的。使用下面的 RBAC 资源来实现这一点:

```
kubectl create -f restricted-psp-rbac.yaml
```

> 上述资源将首先创建一个 **ClusterRole** 资源，该资源被授权使用 **restricted-psp** Pod 安全策略。然后我们创建一个 **ClusterRoleBinding** ，它将我们的 **ClusterRole** 绑定到集群中的所有服务帐户。这实际上意味着，当您在集群上部署任何 pod 时，它必须受限于我们的 **restricted-psp** Pod 安全策略所规定的条件。

# 特权访问是例外

既然我们已经应用了一个默认的限制性 Pod 安全策略，我们现在准备创建我们的特权 PSP，以允许需要提升访问级别的工作负载运行。这些工作负载的一些示例包括 kube-proxy 等控制平面单元，以及 nginx 等入口单元。

使用下面的模板创建下面的特权 PSP，你可以根据你的需要调整它:

```
kubectl create -f privileged-psp.yaml
```

> 这个 PSP 有效地允许特权访问和升级，允许容器作为任何用户运行，允许容器使用任何种类的卷，允许主机访问(文件系统和网络)，并允许任何 linux 功能。在确定您的工作负载需要多少特权时，您必须有很强的判断力。您应该首先了解这些工作负载运行所需的最小权限集，然后相应地改进 PSP。

接下来，我们需要定义一个被授权使用我们的特权 PSP 的 ClusterRole:

```
kubectl create -f privileged-cluster-role.yaml
```

现在，我们需要能够确定哪些工作负载需要特权访问，然后将这些工作负载绑定到我们的特权 PSP。这应该根据您的集群的具体情况来进行，以免向特权访问开放整个集群。

为此，我会提出两个问题:

1.  您的集群上是否有一个命名空间(例如 kube-system ),其中所有工作负载都以类似方式运行，并且需要特权访问？
2.  命名空间中是否只有特定的工作负载需要特权访问？

对于场景 1，使用以下 RBAC 资源模板允许特定命名空间中的所有服务帐户使用特权 pod 安全策略:

```
kubectl create -f namespaced-privileged-access.yaml
```

对于场景 2，使用以下 RBAC 资源模板允许命名空间中的特定服务帐户使用特权 pod 安全策略:

```
kubectl create -f workload-specific-binding.yaml
```

# 启用 Pod 安全策略准入控制器

现在我们已经创建了 pod 安全策略和 RBAC 资源，我们需要在 Kubernetes API 服务器上启用 Pod 安全策略准入控制器。确保已经创建了所有这些资源，否则，一旦启用了准入控制器，将无法运行任何工作负载(它要求至少存在 1 个 Pod 安全策略)。

这是通过更新 Kubernetes API 服务器上的 **enable-admission-plugins** 标志来完成的，以便在列表中包含 **PodSecurityPolicy** 。如果您使用的是托管 Kubernetes 提供商(EKS、AKS 等)，您的提供商有责任确保启用该功能，否则，您可以使用上述步骤。

要查看当前可用的准入插件，请运行:

```
kube-apiserver -h | grep enable-admission-plugins
```

# 确认

要查看您的 pod 正在使用的 PodSecurityPolicy，请执行以下命令:

```
kubectl describe <pod-name> -n <your-namespace>
```

# 结论

总之，pod 安全策略允许您定义一组条件，工作负载必须限制在这些条件下才能运行。您可以访问您可以为策略定义的大量条件，从而为实施安全架构提供了极大的灵活性:

*   卷类型
*   主机文件系统和网络访问
*   特许访问和升级
*   Linux 功能
*   容器运行用户

还有更多…

有了上面的蓝图，您将能够以安全第一的心态操作您的 Kubernetes 集群，同时仍然可以在需要的地方进行特权访问。

# 资源

[Pod 安全策略:Kubernetes 文档](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)