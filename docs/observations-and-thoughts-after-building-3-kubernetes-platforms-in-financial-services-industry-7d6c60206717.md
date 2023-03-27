# 金融服务业三大 kubernetes 平台建设后的观察与思考——第三部分——GitOps

> 原文：<https://itnext.io/observations-and-thoughts-after-building-3-kubernetes-platforms-in-financial-services-industry-7d6c60206717?source=collection_archive---------0----------------------->

![](img/930adb549b0d30b8bba4f83534568322.png)

ArgoCD 赢得 GitOps 空间

在这个博客系列的第三部分，我分享了我对使用 GitOps 管理 kubernetes 上运行的平台组件和工作负载的观察。

以前的零件:

*   [第 1 部分](/observations-and-thoughts-after-building-3-kubernetes-platforms-in-financial-services-industry-6705511c8e9b) —概述，K8s 平台即服务产品，网络
*   [第二部分](/observations-and-thoughts-after-building-3-kubernetes-platforms-in-financial-services-industry-158eba494528) —工作负载身份、秘密管理/外部化

# GitOps 工具

在这一领域，市场并不缺乏选择。GitOps 是关于只根据存储在 Git 中的状态(配置)来配置集群和工作负载，并让操作员确保所需的状态与当前状态相匹配。想法是您可以删除一个集群并从 Git 中完全重新部署它。

GitOps 工具是一个我非常固执己见的领域，因为我已经看到了好的、坏的和丑陋的方面。在我工作过的 3 个环境中，我与以下人员互动过(按优先顺序)

*   **ArgoCD** — ArgoCD 是 GitOps 刀具的瑞士刀。它以集中的模式运作。它可用于平台和应用程序工作负载。它具有 UI、身份集成、控制(资源白名单/黑名单、RBAC)、部署排序(waves)、部署监控&健康检查、强大的声明式支持(helm、kustomize、plain YAML 等)，以及一组与应用管理一致的定制资源(*应用*、 *AppProject* 、*应用集*)
*   **Terraform** —在单租户平台中，我们没有复杂的需求，所有部署都是掌舵图，所以我在 Terraform 中构建了一个小型掌舵图“部署者”，并集成了从 Azure Key Vault 中提取秘密的功能。与部署完整的 ArgoCD 堆栈、秘密外部化和其他组件相比，这是一个简单的解决方案。
*   **FluxCD —** 我在我的第一个平台构建和平台组件(Splunk daemonset、Aquasec、pod identity、nginx 等)中使用了 Flux。当时，它没有秘密集成，这是一个很大的挑战。Flux 是一个分散模型，其中每个实例同步一个应用程序堆栈。
*   **Anthos 配置同步** —针对 GKE(或 Anthos 客户)的解决方案。配置同步是一个基本的工具，支持 kustomize 和基本的头盔支持(渲染)。它的目标是应该在所有集群上的“平台”组件。它以“集群舰队为部署单位”的思维运作。它提供选择器来允许配置被部署/定制到集群的子集。总的来说，如果您将配置同步的范围限制在必须部署到所有集群的最基本的基础上，那么配置同步可能是好的。它的爆炸半径对我来说太大了。此外，将应用程序组件分解成层次结构是一项工程开销，我真的很纠结！此外，购买 Anthos 的客户认为他们必须使用它来平衡他们的投资，并且经常以次优的 GitOps 管理而告终。
*   **定制数据库/后端** —定制应用程序围绕部署和生命周期管理 SQL 和 API 中的应用程序清单。

**关键观察:**

*   将工具与小爆炸半径的展开装置对齐！部署范围太大，你会做各种各样的体操；导致修补、覆盖和排除的网络。如果你发现自己在创建一个如何定制配置的决策矩阵…那你就做错了。Kubernetes 足够复杂；不要让它变得更复杂。
*   选择一个 GitOps 工具。使用我的企业架构师思维，为一项功能选择一个工具并使用它。一旦您开始在同一个环境中使用多个冗余工具，您就在重复投资并增加复杂性
*   不要走常规路线——大多数问题已经解决了。很少有组织需要重新发明轮子。分析市场，找到最合适的，并使用它。
*   在我看来，ArgoCD 是这一领域的赢家

在未来的实验中，我想使用 ArgoCD 来管理集群上的一切，并利用 waves 在需要时以特定的顺序实例化组件(即:在任何具有外部化机密的服务之前部署外部机密)。我还想尝试应用程序集。