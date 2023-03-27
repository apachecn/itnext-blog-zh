# 在 OpenShift 上向您的应用添加安全层-更新:欢迎使用 Vault 代理注入器

> 原文：<https://itnext.io/adding-security-layers-to-your-app-on-openshift-update-welcome-vault-agent-injector-46cab161c366?source=collection_archive---------0----------------------->

今天，保护你的应用程序是必须的，但是如果你没有在一开始就考虑它，那么在不修改代码的情况下引入它是很困难的。幸运的是，像 [Kubernetes](http://kubernetes.io/) 这样的容器和平台带来的新的云原生模式提供了简单的方法来解决安全问题，而无需接触代码。

这是对本系列的第 3 部分和第 4 部分的更新，在这里我们已经了解了如何在 [Kubernetes](http://www.kubernetes.io/) 上轻松部署 [Hashicorp Vault](https://www.vaultproject.io/) 并从我们的`fruits-catalog`应用程序中使用它来保护数据库凭证等敏感信息。

时间飞逝…从这个系列开始，Vault 与 Kubernetes 的集成已经有了很大的改进！我们将在这一部分看到——这是对以前部分的更新——由于有了 [Vault Agent Injector](https://www.vaultproject.io/docs/platform/k8s/injector/) ，现在将我们的应用程序连接到 Vault 进行秘密注入是多么简单和优雅。

> 这部分由 Hashicorp 的 [Nicolas Ehrman](https://medium.com/@nehrman) 实现并校对。感谢 Nico 的帮助和建议！

![](img/2782f91f5faba0bd8b1717aa627fa4aa.png)

更新-欢迎 Vault Agent Injector Kubernetes 集成

*这篇文章是一个更大的系列文章的一部分，讲述了如何给现有的应用程序增加安全层。到目前为止公布的其他部分有:*

*   [*第 1 部分—部署和 TLS 入口*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-1-deployment-and-tls-ingress-9ef752835599)
*   [*第二部分——使用钥匙锁进行认证和授权*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-2-8320018bcdd1)
*   [*第三部分——金库秘密管理*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-3-secret-management-with-vault-8efd4ec29ec4)
*   [*第四部——带金库的动态秘密*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-4-dynamic-secrets-with-vault-b5fe1fc7709b)
*   [*第 5 部分——与 Istio*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-5-mutual-tls-with-istio-a8800c2e4df4) 的相互 TLS
*   [*第 6 部分—使用保管库和证书管理器的 PKI 服务*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-6-pki-as-a-service-with-vault-and-cert-e6dbbe7028c7)
*   *更新—欢迎保险库代理注入器(此帖子)*

# 我们想在这部分实现什么？

还记得本系列的第 3 和第 4 部分吗？通过丰富我们的配置的部署配置，我们在应用程序容器旁边添加了一个 [Init 容器](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)和一个 [Sidecar 容器](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#understanding-pods)，以便最初从 Vault 获取和更新数据库凭证。

此时，我们除了编辑“手边的”部署和添加基本的容器(这些容器执行奇特的 API 调用和文件处理命令，以便在正确的位置提供应用程序配置文件)之外，没有其他选择。现在这变得容易多了，这要感谢库代理注入器，它是一个 Kubernetes 的本地对象，通过在我们的`fruits-catalog` pod 中注入库代理来为我们完成这些修改。

因此，我们将在我们的 Vault 实例旁边部署这些注入器，以便使它集成我们的应用程序启动流程，如下所述:

![](img/c6130e2fc668d82b7048a3e83d338cfe.png)

用这个注入器改造我们的架构和启动流程，这样我们就会得到这个相关点:

1.  在`DeploymentConfig`创建时，根据规范注释的存在，`vault-agent-injector`将放置一个变异钩子。所以每次我们创建一个`fruits-catalog`容器，一个[初始容器](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)和/或一个[端容器](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#understanding-pods)将被注入到我们的容器中，以检索管理到保险库中的秘密，
2.  我们还将获得一个由 Kubernetes 颁发的特定令牌——以及相关的证书——在启动时注入到我们的应用程序 Pod 中，
3.  由名为`vault-agent-init`的 Vault 代理注入器注入的 [Init 容器](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)将使用该特定令牌对 Vault 进行认证调用，
4.  Vault 将能够根据 Kubernetes APIs 检查这个调用的有效性。如果成功，Vault 将允许应用程序访问存储在 Vault 中的机密，
5.  最后，从 Vault 中检索到的秘密将通过一个共享文件系统提供给我们的主 Spring Boot 容器:一个配置文件将在`/deployment/config`上创建，这样在这个文件中找到的值将覆盖来自部署配置的值。

除了另一个名为`vault-agent`的边车容器被注入并负责租约更新之外，这个过程在动态机密的情况下保持完全相同。

当 pod 被删除时——由于重新调度或应用程序缩减——变异挂钩负责在保险库端停止所获得的凭证租用。

# 如何应用？

现在让我们看看如何部署和配置它…如果您已经阅读了本系列的第一部分，您可能已经克隆了包含所有资源和资产的 [my GitHub repository](https://github.com/lbroudoux/secured-fruits-catalog-k8s) 。如果没有，我敦促你这样做；-)

> 在开始动手操作之前，我们假设您已经部署并配置了 Vault 来使用 Kubernetes 身份验证。如何做到这一点在本系列的[第 3 部分](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-3-secret-management-with-vault-8efd4ec29ec4)中有详细介绍。

第一项任务是部署存储库代理注入器。Hashicorp 现在提供了一种便捷的方式来部署保险库和保险库代理注入器，方法是使用一个简单的舵图(此处记录为)。如果您已经阅读了前面的部分，我们已经部署了 Vault，所以我提供了一个简单方便的 YAML 文件，用于添加 Vault 代理注入器。但在此之前，我们必须创建一个 ServiceAccount 并为 OpenShift 设置正确的权限:

```
$ oc create sa vault-agent-injector -n fruits-catalog$ oc adm policy add-scc-to-user anyuid -z vault-agent-injector -n fruits-catalog$ oc apply -f k8s/vault-agent-deployment.yml -n fruits-catalog
```

这就差不多完成了！最后要做的事情是更新我们的`DeploymentConfig`,在这里我们可以删除所有之前添加的容器，只放置这个 se tof 注释:

你看所有这些`vault.hashicorp.com/*`的注解。大多数都很简单，有一个值得我们注意:

```
vault.hashicorp.com/agent-inject-template-application.properties: |          
  {{- with secret "database/creds/fruit-catalog-role" -}}           
  spring.data.mongodb.uri=mongodb://{{ .Data.username }}:{{ .Data.password }}@mongodb/sampledb
  {{- end }}
```

这确实是一个[领事模板](https://github.com/hashicorp/consul-template)的表情，有助于判断:

*   生成的文件的名称。这里`application.properties`，
*   该文件的内容。这里有一个简单的行，包含到数据库的连接链:`spring.data.mongodb.uri=mongdb://user:password@mongodb/sampledb`

使用这种模板符号比我们使用的 shell 命令要简单得多。为了方便起见，我为静态秘密(第 3 部分)和动态秘密(第 4 部分)放置了现成的资源文件。因此，这只是用以下资源替换现有资源的问题:

```
$ oc replace -f k8s/fruits-catalog-static-secret-with-agent-deployment.yml# OR for dynamic secrets...$ oc replace -f k8s/fruits-catalog-dynamic-secret-with-agent-deployment.yml
```

# 检查它是否按预期工作

除了记录在[第 3 部分](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-3-secret-management-with-vault-8efd4ec29ec4)和[第 4 部分](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-4-dynamic-secrets-with-vault-b5fe1fc7709b)中的程序外，此处不再赘述😉

# 结论

在这次更新中，我们看到了现在将管理委托给 Hashicorp Vault 以及将我们的数据库凭证注入到我们的应用程序中是多么简单和优雅。从一个有点糟糕和容易出错的情况来看，Vault Agent Injector 所取得的进步现在正在将这种集成模式转变为一种优雅且非常容易设置的模式！

使用云原生模式，如由 Kubernetes [许可控制器](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)创建的由声明性注释触发的 [Init](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) 和 [Sidecar](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#understanding-pods) 容器，再加上生命周期挂钩，您现在拥有了短暂的凭证，它们只存在于 pod 的生命周期中。使用[咨询模板](https://github.com/hashicorp/consul-template)，整个过程变得简单且高度可配置。

感谢阅读，敬请关注！