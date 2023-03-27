# 用 Flux CD 升级到 Helm 3

> 原文：<https://itnext.io/upgrading-to-helm-3-with-flux-cd-6b7014223a51?source=collection_archive---------3----------------------->

![](img/df459cefbbc98e5dfa2c1c28253ab3db.png)

即使上述内容在本周公开，Mettle 的平台团队总是希望走在曲线的前面。因此，这篇博文的剩余部分将谈论我们将集群升级到 v3 的旅程。

# 准备

以下部分详细介绍了开始升级前所需的准备工作。

## HelmRelease:全部默认为 v2

作为准备，我们确保我们所有的头盔版本都指定了他们使用的头盔版本，这是通过添加以下内容来完成的:

```
spec:
  chart:
    name: backend
    repository: https://example.storage.googleapis.com
  releaseName: account-balance
  helmVersion: v2
```

## 舵操作员:允许 v2 和 v3 图表

我们需要在 helm operator 实例上设置以下选项，以便它可以支持 Helm v2 和 v3:

```
- --enabled-helm-versions=v2,v3
```

## 赫尔姆:在本地安装 v2 和 v3

为此，我们需要头盔 2 和头盔 3，所以最好两个都安装，你可以通过执行以下命令来完成:

```
**brew** install helm@2
**brew** install helm
**cd** /usr/local/bin
**ln -s** /usr/local/opt/helm@2/bin/tiller tiller
**ln -s** /usr/local/opt/helm@2/bin/helm helm2
**ln -s** helm helm3
```

**从现在开始你需要用** `**helm3**` **来控制第三代头盔，用** `**helm2**` **来控制第二代头盔。**

# 升级工作流

这篇博文的剩余部分描述了升级的工作流程。

## 停止舵操作员

我们需要停止集群中的 helm 操作符，以允许集群中现有的 Helm 版本进行升级。我们通过在部署中将副本数量扩展到零来实现这一点。

## 在集群中升级 helm 版本

为了升级集群中已经存在的 helm 版本，我们需要使用[https://github.com/helm/helm-2to3](https://github.com/helm/helm-2to3)。

首先，我们需要使用以下命令列出集群中所有现有的 helm 版本:

## 移除分蘖

一旦我们将集群中现有的版本升级到 v3，Tiller 就不再需要运行了。我们通过在部署中将副本数量扩展到零来实现这一点。

## 将 HelmReleases 升级到 v3

然后我们更新了我们所有的头盔版本，指定`v3`为头盔版本，这是通过添加以下内容完成的:

```
spec:
  chart:
    name: backend
    repository: [https://example.storage.googleapis.com](https://example.storage.googleapis.com)
  releaseName: account-balance
  helmVersion: v3
```

将这些更改合并到您的集群中，并在启动 Helm 操作器之前等待集群中的所有`HelmRelease`清单都使用 v3。

你可以使用`kubectl get hr -A -o yaml | grep ‘helmVersion: v2’`来确保所有的 v2 引用已经被完全移除。

## 启动舵操作器

最后一步是将 helm operator 部署扩大到 1 个副本，并且只有**允许发布版本 3，这是通过添加以下内容完成的:**

```
- --enabled-helm-versions=v3
```

**只允许** `**v3**` **这里是关键，因为集群不再有舵柄，所以** `**v2**` **不再是有效选项！**

## 合并舵操作员变更

建议首先合并舵操作员 PR，并确保 pod 成功运行。

## 合并 HelmRelease 更改

现在将 HelmRelease 的更改合并到`helmVersion:v3`中，现在是时候验证迁移工作了

# 健全性检查

一旦重新打开头盔操作器，它就开始升级头盔释放装置。当描述一个`HelmRelease`时，您将看到如下输出:

```
Running upgrade for Helm release 'platinum-docs' in 'web'.
```

你要等到所有的`HelmRelease`资源都说:

```
Release was successful for Helm release ... 
```

您可以通过运行以下命令来验证这一点:

```
watch "kubectl get hr -A | grep -v 'Release was successful for Helm release'"
```

您希望上面的命令最终是一个空列表，一旦您看到这个，迁移就成功了。

# 摘要

总的来说，这是一个相当轻松的过程，从开始到结束需要大约一个小时来执行升级(每个环境)。我想再次感谢斯蒂芬·普罗丹([https://twitter.com/stefanprodan](https://twitter.com/stefanprodan))为我提出的升级战略提供咨询。