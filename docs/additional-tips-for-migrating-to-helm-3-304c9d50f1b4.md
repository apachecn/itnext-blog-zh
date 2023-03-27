# 迁移到头盔 3:你需要知道什么

> 原文：<https://itnext.io/additional-tips-for-migrating-to-helm-3-304c9d50f1b4?source=collection_archive---------4----------------------->

![](img/2cd11273ff43c071de7a8fd5bbb914cb.png)

马特·阿特兹在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

现在 Helm 包管理器的最新和最好的主要版本已经稳定了，您可能想知道升级/迁移/移动需要花费多少精力。尽可能复杂的头盔，放心升级过程非常简单。**官方 Helm 博客有一个优秀的迁移过程指南，** [**可以在这里找到**](https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/) 。这篇文章非常简单明了，它给了你开始将各个版本单独移植到 Helm 3 所需要的一切。但是，如果您想一次全部迁移，该怎么办呢？在移除蒂勒之前，你如何确定没有东西还在使用它？

# 下载 Helm 3 二进制文件

我们想在测试最新版本的同时测试 Helm 2，所以我们应该准备好两个版本的二进制文件，直到我们准备好拔掉 v2 的插头。在这里下载 v3 [的最新稳定版本，并将其添加到您的 PATH 中。我发现将现有的 v2 二进制文件重命名为`helm2`并将最新版本重命名为`helm3`更容易。我把两个版本都保存在`/usr/local/bin`，这样我就可以在需要的时候在它们之间切换:](https://github.com/helm/helm/releases)

```
➜ helm2 version
Client: &version.Version{SemVer:"v2.16.0", GitCommit:"e13bc94621d4ef666270cfbe734aaabf342a49bb", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}➜ helm3 version
version.BuildInfo{Version:"v3.0.1", GitCommit:"7c22ef9ce89e0ebeb7125ba2ebf7d421f3e82ffa", GitTreeState:"clean", GoVersion:"go1.13.4"}
```

# 准备您的 CI 脚本和图表

在运行升级过程之前，您必须确保您的配置项脚本和自定义图表与 Helm 3 兼容。我写了一篇博客，[涵盖了一些需要注意的事情，其中大部分都可以轻松解决。OpenAPI 验证机制非常酷，但是它可能会让您措手不及:](/breaking-changes-in-helm-3-and-how-to-fix-them-39fea23e06ff)

```
➜ helm install prometheus .
Error: unable to build kubernetes objects from release manifest: error validating "": error validating data: ValidationError(Deployment.spec.template.spec.containers[0].volumeMounts[0]): unknown field "defaultMode" in io.k8s.api.core.v1.VolumeMount
```

一旦所有这些初期问题都解决了，你就可以开始迁移到头盔 3 了！

# 迁移舵配置

我之前提到的 Helm 博客文章中的这个步骤会更新你所有的本地配置，这样它就可以被 Helm 3 使用了。如果您从一个 CI 系统(如 Jenkins、TeamCity 或 TravisCI)中的构建代理运行 Helm，那么您可以跳过这一步。如果你从本地机器或者一个有持久文件系统的中央服务器上运行 Helm，一定要把你的配置转移过来，特别是如果你有自己的 Helm repos 或者使用定制插件的话。不管怎样，确保你通读了这篇文章，以确定它是否与你相关。

# 迁移释放物—带分蘖

现在，这里有几种方法。你可以将特定的版本移植到 Helm 3 来做一些测试，这在博客文章[这里](https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/#migrate-helm-v2-releases)中有所涉及。您还可以选择迁移发布版本，并从 Tiller 中一次性删除它们。就我个人而言，我发现在一个给定的环境中一次性迁移我们所有的版本更容易，但是*将发布数据留在 Tiller* 中，直到我完全确定 Helm v2 没有在我们的环境中使用。这样，就没有盲点，一切都使用相同版本的 Helm。这里有一个快速的方法:

```
# List Helm 2 Releases
# omit --tls flag if you're not using TLS
RELEASES=$(helm list --tls -aq)# Loop through releases and, for each one, test conversion
while IFS= read -r release; do
  helm3 2to3 convert $release --dry-run
done <<< "$RELEASES"
```

一旦你对它的外观感到满意，移除`--dry-run`标志，观察`2to3`插件的神奇之处。

**注意:**正如我提到的，有一个`--delete-v2-releases` 标志，它将迁移释放*并从 Tiller 中删除它。如果你确定你不再需要这些信息，那就自担风险吧。*

# 在你移除分蘖之前…

这是我不想仓促进行的一步，以防我们需要回到掌舵 2。在这一点上，只要你的 CI 系统和团队成员都在使用 Helm 3，就没有理由保留 Tiller。但是如果你想*绝对确定*没有任何东西还在使用旧版本，我建议让 Tiller 呆几个小时，观察`helm ls`的输出，看看`UPDATED`栏中的时间戳是否有任何变化。如果是的话，你知道有东西/有人没有使用头盔 3。

如果版本在迁移到 Helm 3 后被 Helm 2 修改，您必须删除保存版本信息的 Helm 3 Kubernetes secret，以便在不删除相关资源的情况下将其从 Helm 3 中清除:

```
➜ kubectl get secret -n dev
NAMESPACE                NAME                                                 TYPE                                  DATA   AGE                           dev          sh.helm.release.v1.postgres.v1                 helm.sh/release.v1                    1      36d
➜ kubectl delete secret -n dev sh.helm.release.v1.postgres.v1
secret "secret "sh.helm.release.v1.postgres.v1" deleted
```

现在，如果我们用 Helm 3 列出`dev`名称空间中的版本，我们将看到该版本不再存在:

```
NAME           NAMESPACE      REVISION UPDATED                                 STATUS   CHART                APP VERSION
```

现在，我们可以再次执行迁移过程，一旦我们找出谁或什么仍然在使用 Helm 2。一旦解决了这个问题，使用`helm3 2to3 convert`进行迁移就大功告成了。

一旦您完全确定已经准备好删除 Tiller 及其相关的 rbac 角色和数据，只需运行`helm 2to3 cleanup`。

# 迁移释放——无舵舵舵

只需在我上面提供的脚本中添加`--tiller-out-cluster`标志，`2to3`插件就会从您的本地 Tiller 实例中删除发布信息！

```
# List Helm 2 Releases
# omit --tls flag if you're not using TLS
RELEASES=$(helm list --tls -aq)# Loop through releases and, for each one, test conversion
while IFS= read -r release; do
  helm3 2to3 convert $release --tiller-out-cluster
done <<< "$RELEASES"
```