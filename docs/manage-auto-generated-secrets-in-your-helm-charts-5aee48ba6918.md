# 管理你的舵图中自动生成的秘密

> 原文：<https://itnext.io/manage-auto-generated-secrets-in-your-helm-charts-5aee48ba6918?source=collection_archive---------1----------------------->

![](img/e35b4b8a5a877f6eff4b8239aa060cfa.png)

埃里克·麦克林在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

# 为什么？

[Helm](https://helm.sh/) 允许您以模块化的方式打包您的 Kubernetes 应用程序，并根据用户的配置[“值文件”](https://helm.sh/docs/chart_template_guide/values_files/)应用不同的部署逻辑。在第一次部署时，我们通常需要生成随机密码(例如 JWT /会话密码或随机密码)。我们可以选择让我们的舵图用户在舵图部署生命周期之外手动创建/管理这些秘密。但是如果舵图可以处理秘密自动生成的话就方便多了。然而，作为舵图部署的一部分，正确地自动生成和管理这些秘密并不容易。本文将讨论一些简单解决方案的潜在问题，并基于[头盔查找功能](https://helm.sh/docs/chart_template_guide/function_list/#lookup)提出一个更加灵活的解决方案。

# 用随机函数创造秘密

Helm 提供了一系列[随机字符串生成函数](https://helm.sh/docs/chart_template_guide/function_list/#randalphanum-randalpha-randnumeric-and-randascii)，允许我们生成加密安全的随机字符串。一个简单的解决方案是使用 randAlphaNum 函数包含一个创建 k8s secret 对象的模板。

这个简单的解决方案看起来不错。但是当您以后尝试升级现有部署时会遇到麻烦，因为模板不知道集群中当前的机密数据，并且总是用新的随机值更新机密。

# 有条件地创建秘密

更好的解决方案可能是检索现有的秘密数据，并且仅在秘密数据不存在时用新值创建秘密。

这里，我们不再总是将秘密数据设置为随机生成的值。相反，我们将尝试使用[查找函数](https://helm.sh/docs/chart_template_guide/function_list/#lookup)检索当前秘密值，并且仅在找不到现有秘密数据时将秘密数据设置为随机生成的值。

# 设置掌舵资源策略

我们现在有了一个解决方案，不仅可以用于初始部署，还可以用于后续升级。但是由于这个秘密是由 Helm 管理的，所以当你删除 Helm 部署时，它总是会被删除。这可能并不总是自动生成的秘密所期望的行为。

对于 helm 管理的资源，通过[` Helm . sh/resource-policy ` annotation](https://helm.sh/docs/howto/charts_tips_and_tricks/#tell-helm-not-to-uninstall-a-resource)将资源策略设置为“保留”可以防止用户删除/卸载 Helm 部署时资源被删除。下面是一个使用资源策略注释在删除后保留我们的秘密资源的示例:

# 允许用户提供的机密

尽管我们希望在大多数情况下自动生成密码，但我们可能仍然希望允许用户提供他们手动创建的密码，以使解决方案更加灵活。

为了实现这一点，我们可以要求用户通过一个配置字段提供手动创建的秘密名称，例如`.Values.manualSecretName`，并且仅在`.Values.manualSecretName`为空时呈现我们自动生成的名称。

# 关于查找函数的更多信息

Helm 的[查找功能](https://helm.sh/docs/chart_template_guide/function_list/#lookup)是一个非常强大的工具，允许我们根据集群状态应用不同的部署逻辑。因为查找函数需要集群实时状态数据，所以请记住，当您执行以下操作时，它将始终返回空响应:

*   用`helm template`运行你的图表
*   使用`--dry-run`开关部署或升级您的图表

# 结论

让你的头盔图在合适的地方自动为你的用户生成秘密会更方便。使用 Helm 的查找功能，我们可以检查集群的当前状态，以便以更加可靠和灵活的方式创建和管理 Helm 图表中的秘密。