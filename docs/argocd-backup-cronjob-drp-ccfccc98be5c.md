# ArgoCD 备份 cronjob — DRP

> 原文：<https://itnext.io/argocd-backup-cronjob-drp-ccfccc98be5c?source=collection_archive---------3----------------------->

![](img/d7306a0663e1fb80a6a52f8290a16008.png)

凯利·西克玛在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

当麻烦来临时，你最好做好准备。有一个灾难恢复计划将在未来的某一天拯救你。比如有人会不小心删除你的 k8s 集群会怎么样？如果您的组织使用 GitOps 部署风格，这本身就是一种 DRP 友好的部署方式，(所有 k8s/helm yaml 文件都安全地存储在 git 中)。ArgoCD 会将这些文件应用到集群中。Argo 是另一个可能被搞砸/删除的东西。它将自己的状态存储在 k8s 配置图、秘密和 CRD 的 argocd-apps 中。备份它们将使您能够恢复 argo 和所有应用程序安装它与一个单一的命令。

在这篇文章中，我将描述如何设置一个 cronjob 来备份 GCP 的 argo 日志。我们将进入 k8s 服务帐户，将 GCP SA 与 KSA 联系起来，以允许备份 pod 访问 k8s 配置地图和谷歌云存储。(对不起 EKS 用户……)

[argocd-utils](https://argoproj.github.io/argo-cd/operator-manual/server-commands/argocd-util/) 是 argo 团队制作的一款便捷工具，可以从 yaml 文件导出/导入状态。它没有捆绑 argocd 二进制文件，为了在本地运行它，你需要一个 docker 镜像。

本地运行

为了让它在本地工作，你需要安装一个能够访问集群 argocd 的 kubeconfig。

这很容易…现在让我们把它放在一个 cronjob 中并每天运行！为了实现这一点，我们首先需要一个 **Dockerfile** ，它将扩展 argocd 映像并在其上安装 gcloud cli。

在 argocd 映像上安装 gcloud cli

关于上面的一个小评论，ArgoCD 镜像基于 Ubuntu，运行 apt 需要**用户 root** 。

备份脚本

请注意，备份脚本中没有任何与身份验证相关的内容，我们将使用一种巧妙的方式来允许我们的备份 pod 被授权访问 k8s api 和 GCS，称为 [GCP 工作负载身份](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity)。总之。我们将创建一个 k8s 服务帐户，并将其绑定到一个可以访问 k8s api 的角色，我们还将创建一个 GCP 服务帐户，该帐户对包含我们的备份的 GCS 存储桶具有管理员访问权限。并将 KSA 绑定到 GSA，以允许 pod 将两个服务帐户识别为一个帐户。

将 GSA 与 KSA 绑定

让我们分解以上内容:

*   创建 GCS 存储桶
*   创建 GCP 服务帐户
*   在存储桶上分配 GCP SA 对象管理角色
*   将 GCP 服务协议绑定到 KSA 服务协议(k8s 服务帐户)
*   测试任务有效

在测试这个神奇的作品之前，您首先需要创建一个 k8s SA，并用与 GCP SA 相同的名称对其进行注释。

sa+角色+tolebinding

上面重要的部分是 ClusterRole 规则，它使 argocd-util 能够访问相关的 k8s api，以及 ServiceAccount 注释，GCP 工作负载 identity 将使用该注释将 KSA 链接到我们前面创建的 GSA。

如果到目前为止一切都做得正确，那么您应该能够运行 Test 命令并从 test pod 中列出空的 GCS bucket。

最后一件事当然是将 CronJob.yaml 链接到 KSA，并告诉它每天运行。

```
*# cronjob.yaml
...
schedule*: "0 0 * * *"
...
serviceAccount: stg-argocd-backup-eu-west1
```

我们完了。

要从备份运行中恢复 ArgoCD:

```
argocd-util -n argocd import < /backup/backup.yaml
```

如果这篇文章帮助你离开，给我一个评论。

别忘了给仙人掌浇水。

干杯。