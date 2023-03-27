# 修改头盔 3 发布清单

> 原文：<https://itnext.io/modify-helm-3-release-manifest-c3870b90213?source=collection_archive---------1----------------------->

![](img/4e8546a5e03b1553b1e454a8e8f0d867.png)

乔·什切潘斯卡在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

# 为什么我们要修改已部署的头盔发布清单？

Helm 总是试图通过“种类”(例如`Deployment`)和“API 版本”(例如`apps/v1`)来匹配清单中的 Kubernetes 对象。但是，您的 Kubernetes 集群支持的“API 版本”可能会发生变化，例如在将集群升级到更高版本或为第三方应用程序安装不同版本 [CRD(自定义资源定义)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)(例如[证书管理器](https://cert-manager.io/docs/installation/kubernetes/))之后。

当集群不再支持用于部署“种类”的 Kubernetes 对象的“API 版本”时，Helm 将无法匹配发布清单中的对象并报告错误:

```
unable to recognize "": no matches for kind "xxxxx" in version "xxxxx"
```

您可以要求 Helm 重试升级(在某些情况下，您可能需要[这种技术](https://medium.com/@t83714/how-to-fix-helm-upgrade-error-has-no-deployed-releases-mystery-3dd67b2eb126)来强制 Helm 重试)，但 Helm 永远无法从该错误中恢复，除非您手动修改最新的发布清单并将“API 版本”更新为您的集群支持的版本。显然，删除 Helm 版本并重新安装 Helm 图表是可行的——但这不太可能是大多数生产站点的选项。

# 出口放行清单

为了修改 Helm 发布清单，您需要首先将清单导出为纯文本文件。内置的`helm get manifest`命令也可以检索清单内容，但它不是我们将清单保存回编码数据所需的格式。

从 Helm 3 开始，发布数据在您的应用程序名称空间中存储为 Kubernetes " [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) "。如果通过以下方式列出应用程序命名空间中的所有机密:

```
kubectl -n [your namespace] get secrets
```

您会发现发布数据机密是按发布版本列出的:

```
NAME                           TYPE                Data  AGE
sh.helm.release.v1.xxxx.v3    helm.sh/release.v1   1     1d
sh.helm.release.v1.xxxx.v2    helm.sh/release.v1   1     1d
sh.helm.release.v1.xxxx.v1    helm.sh/release.v1   1     1d
```

这里的“xxxx”是你的头盔部署的发布名称。而`v1`、`v2`、&、`v3`都是你掌舵部署的发布版本。您通常会想要修改最新的发布版本(最高版本号)。要导出`v3`发布清单，您可以运行以下命令:

```
kubectl get secrets -n [my-namespace] sh.helm.release.v1.xxxx.v3 -o json | jq .data.release -r | base64 --decode | base64 --decode | gunzip - > manifest.json
```

请注意，要运行这个命令，您需要安装【https://github.com/stedolan/jq】()并确保`base64` & `gunzip`可用。

命令成功运行后，您可以在 JSON 文件“manifest.json”中找到清单。现在，您可以使用任何文本编辑器编辑这个 JSON 文件。

# 保存修改后的清单数据

更正清单内容后，可以通过以下命令将“manifest.json”保存回发布历史:

```
# encode manifest.json to variable DATA
DATA=`cat manifest.json | gzip -c | base64 | base64`# patch secret with new data
kubectl patch secret -n [my-namespace] sh.helm.release.v1.xxxx.v3 --type='json' -p="[{\"op\":\"replace\",\"path\":\"/data/release\",\"value\":\"$DATA\"}]"
```

> 更新:默认情况下，macOS 上的 base64 不会输出链接中断，但在其他平台上不会。如果您需要在其他平台(例如 Linux)上运行它，您需要从输出中删除换行符。您可以改为运行以下代码:DATA = ` cat manifest . JSON | gzip-c | base64 | base64 | tr-d ' \ n \ r ' `
> 感谢 Kyle Barton 在他的评论中提供了这个解决方案。

您可以使用内置的 Helm 命令来验证清单的正确性:

```
helm -n [my-namespace] get manifest xxxx --revision 3
```

这里，` xxxx '是您的发布名称，而` 3 '是发布历史修订/版本号。

# 修改头盔 2 发布清单？

修改圣盔 2 发布清单也是可能的，但是要麻烦得多。您可以根据自己的目的修改以下脚本:

[https://gist . github . com/t 83714/9317 f 8933223d 3c 4 BD 626128 ee 306208](https://gist.github.com/t83714/9317f8933223d3c4bd626128ee306208)