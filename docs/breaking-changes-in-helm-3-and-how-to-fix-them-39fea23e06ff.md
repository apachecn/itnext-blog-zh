# 头盔 3 的突破性变化(以及如何修复它们)

> 原文：<https://itnext.io/breaking-changes-in-helm-3-and-how-to-fix-them-39fea23e06ff?source=collection_archive---------2----------------------->

![](img/58e96ef4917b3a6525eeec3e00107c82.png)

弗兰克·艾弗特在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄的照片

*2020 年 14 月 2 日编辑:添加了关于-* n *标志如何变化的说明。还添加了关于命名空间创建的说明。*

*编辑 22/05/2020:增加了* [*巴蒂*](https://medium.com/u/f45899a535aa?source=post_page-----39fea23e06ff--------------------------------) *和* [*格诺特·霍本赖希*](https://medium.com/u/17b5bbacec6e?source=post_page-----39fea23e06ff--------------------------------) 的一些建议

最近我很幸运地测试了从头盔 2 到头盔 3 的迁移。头盔 3 是一个主要版本，解决了头盔 2 的许多问题。在很大程度上，这是一个平稳的过渡。helm-2to3 插件使这个过程变得简单透明，一旦你移动了你所有的财产，Helm 3 很容易使用。也就是说，要知道**你的 CI/CD 脚本可能会崩溃。Helm 接受的参数、Helm 查询的发布方式以及图表发送到 Kubernetes API 之前的新 OpenAPI 验证流程都有重大变化。在我自己遇到这些问题之前，我肯定没有意识到这些变化，并且在迁移文档中也没有发现特别指出它们的地方。所以，我想我应该在这里为那些挠头想知道为什么部署失败的人记录所有这些！**

# 对标志的更改(新行为、弃用等)

这将主要(如果不是全部)归因于新版本头盔中旗帜的变化。**这里列出了你在头盔 2 中最可能用到的旗帜的变化:**

1.  **使用** `**helm install**`时 `**-n**` **标志不再存在。使用 Helm 2，你可以使用`-n`来指定发布的名字，而不是使用一个自动生成的名字。**

```
helm install . -n my-awesomechart --namespace dev
```

这是有问题的，因为当使用 kubectl 与您的集群交互时，`-n`用来指您运行命令的名称空间。有了头盔 3，`-n`就完全没了。相反，*发布的名称是一个位置参数*:

```
➜ helm3 install -hThis command installs a chart archive.
[...]
Usage:
  helm install [NAME] [CHART] [flags]
```

此外，如果您希望 Helm 为您生成发布名称，您现在必须提供`--generate-name`参数并省略发布名称的位置参数。

2.净化现在是头盔 3 的默认行为。因此，任何传递了`--purge`标志的脚本现在都会抛出一个错误:

```
➜ helm delete --purge
Error: unknown flag: --purge# If you want to keep the release history like Helm 2 used to when omitting the --purge flag, do this:
➜ helm delete --keep-history
```

3.**既然舵柄没了，** `**--tls**` **旗也随之而去。**使用该标志将抛出与上面相同的`unknown flag`错误。

4.**`**--recreate-pods**`**旗也不存在了。这在头盔 2 中被标记为[不推荐使用](https://github.com/helm/helm-www/pull/286)，文档已经更新，告诉你如何在头盔 3 中正确使用。你可以在文档[这里](https://v3.helm.sh/docs/howto/charts_tips_and_tricks/#automatically-roll-deployments)看到一个例子。****

**5.**`**--timeout**`**旗帜现在需要一个持续时间单位。之前您可以指定 300，这将转换为 300 秒。现在您必须指定`m`为分钟或`s`为秒，例如`300s`、`5m`或`5m0s`。******

****6.**`**--strict**`**`**helm repo update**`**的标志不见了。**在头盔 3 中似乎没有类似的选项。********

****7.**`**--save**`**`**helm dependency update**`**的标志不见了。再次声明，头盔 3 中似乎没有类似的选项。**********

****8.`**helm status**` **不再显示资源赫尔姆创建的状态。**GitHub 有一个未解决的问题，要在 Helm 3 中恢复这个功能。你可以在这里阅读这个。****

****感谢 [Bharti](https://medium.com/u/f45899a535aa?source=post_page-----39fea23e06ff--------------------------------) 的第 6、7、8 项！****

# ****Helm 3 按名称空间列出了版本****

****头盔 3 现在的行为类似于 kubectl，我认为这是一个了不起的增加。使用 Helm 2，只需输入`helm ls`就可以查询所有名称空间中的*所有版本。使用 Helm 3，运行`helm ls` 只会显示默认名称空间中的版本，就像`kubectl get pods`一样。要列出任何非默认名称空间中的版本，您必须*提供`-n`参数和名称空间，就像使用 kubectl 时一样。这里有一个更好的例子来说明我的意思:******

```
**# Helm 2: shows all releases across all namespaces
➜ helm ls
NAME        REVISION      UPDATED      ...  NAMESPACE
prometheus  1             Sat Jan 04   ...  default
devapp      2             Sat Jan 04   ...  dev
prodapp     2             Sat Jan 04   ...  production# Helm 3: only shows releases in default namespace
➜ helm ls
NAME        REVISION      UPDATED      ...  NAMESPACE
prometheus  1             Sat Jan 04   ...  default# Helm 3: using -n shows release in specified namespace
➜ helm ls -n production
NAME        REVISION      UPDATED      ...  NAMESPACE
prodapp     2             Sat Jan 04   ...  production# Helm 3: shows all releases across all namespaces
➜ helm ls --all-namespaces
NAME        REVISION      UPDATED      ...  NAMESPACE
prometheus  1             Sat Jan 04   ...  default
devapp      2             Sat Jan 04   ...  dev
prodapp     2             Sat Jan 04   ...  production**
```

******注意:Helm 3 看不到 Helm 2 管理的版本，除非您迁移它们。上面的例子只是为了说明清单是如何变化的。你必须使用上面的 2 对 3 插件把你的版本移植到头盔 3 上，头盔 3 才能看到它们。******

****这是一个很小的变化，但是如果您的 Bash 脚本先前已经使用了`helm ls`,然后在输出中使用了`grep`,那么这个变化就不小了。现在，这些命令将会失败，因为它们只检查默认的名称空间。如果你想要和在头盔 3 中一样的行为，确保你通过了`--all-namespaces`标志！****

# ****Helm 不会为您创建名称空间，除非您要求它这样做****

****这实际上并不是很大的变化，但是您仍然应该意识到这一点。当安装一个图表时，Helm 2 会为你创建一个目标名称空间，如果它还不存在的话。Helm 3 只将版本安装到预先存在的名称空间中，并会抛出一个错误，指出目标名称空间不存在。从 Helm v3.2.0 开始，您可以明确地告诉 Helm 应该使用`--create-namespace`标志来创建名称空间(感谢[Gernot hben Reich！](https://medium.com/u/17b5bbacec6e?source=post_page-----39fea23e06ff--------------------------------))。****

# ****Kubernetes OpenAPI 验证****

****Helm 3 根据 Kubernetes OpenAPI 模式验证所有渲染模板。[这是在 3.0.0 版中启用的。](https://github.com/helm/helm/releases/tag/v3.0.0)在此之前，Helm 会渲染你的模板，并将它们发送到 Kubernetes API，并由 API 来确定哪些是有效的。任何不正确缩进的内容都会引发错误，传递给错误端点的参数通常会被丢弃，而不会记录到 stdout/stderr 中。****

****使用 Helm 3，你的渲染模板在被发送到 Kubernetes API 之前会被验证*。*这意味着任何无效的字段都将触发一个非零退出代码，从而导致发布失败。如果你的 YAML 文件与 Kubernetes API 文档完全一致，那就太好了。如果没有，您将不得不删除任何无效字段。****

****这里有一个例子。取[稳定的普罗米修斯图表](https://github.com/helm/charts/tree/master/stable/prometheus)并将无效参数插入普罗米修斯容器的`volumeMounts`部分。在本例中，我将添加参数`defaultMode`，根据 Kubernetes API 规范:，该参数[无效](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.11/#volumemount-v1-core)****

```
**[...]
volumeMounts:- name: config-volume mountPath: /etc/config defaultMode: 256 readOnly: true[...]**
```

****我现在试着用头盔 2 安装这张图表:****

```
**helm install . -n prometheus --name prometheus
➜ helm ls
NAME        REVISION      UPDATED      ...  NAMESPACE
prometheus  1             Sat Jan 04   ...  default**
```

****嗯，那很容易，而且库伯内特似乎没有抱怨。让我们用头盔 3 试试:****

```
**➜ helm install prometheus .
Error: unable to build kubernetes objects from release manifest: error validating "": error validating data: ValidationError(Deployment.spec.template.spec.containers[0].volumeMounts[0]): unknown field "defaultMode" in io.k8s.api.core.v1.VolumeMount**
```

****如果我们删除无效的参数，我们可以看到头盔 3 没有问题，现在安装图表:****

```
**➜ helm install prometheus .
NAME: prometheus
LAST DEPLOYED: Mon Jan  6 15:11:14 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
prometheus-server.default.svc.cluster.local
...**
```

****你的图表可能没有这些问题，但是如果它们是在亿万年前写的，并且一直愉快地使用头盔 2，你可能会看到一些像这样的验证失败。无需担心；只需检查 Kubernetes API 文档，确保参数有效或缩进正确。****

# ****结论****

****这些只是我在转向头盔 3 时发现的一些问题。我应该提一下，这并不是要以任何方式打击头盔 3；这是对头盔 2 的巨大改进，我强烈推荐移植到头盔 2 上。如果你看到了任何问题，请随时给我留言，我会更新这篇文章！****