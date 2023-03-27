# Kubernetes 中的功能部署

> 原文：<https://itnext.io/feature-deployments-in-kubernetes-c74bdcff0d8e?source=collection_archive---------0----------------------->

![](img/b2c11a46aaaa6de26e1fb51b5c1d4600.png)

许多公司想要同样的东西。能够部署代码来测试特定的功能/缺陷，而无需将其实际部署到开发、试运行或生产中。Kubernetes 通过使用名称空间使这变得非常容易。有关名称空间的更多信息，请查看:[https://kubernetes . io/docs/concepts/overview/working-with-objects/namespaces/](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

所以问题是:我们如何让我们的项目可以被部署到一个名称空间，而不必教我们所有的开发人员 kubernetes 的来龙去脉？要实现这一点，需要做几件事:

*   命名空间
*   配置映射
*   秘密
*   正在部署的服务的清单
*   一个将一切联系在一起的脚本

我决定采取配置和约定相结合的方式来完成上述任务。

# T 何命名空间

我编写了一个简单的 namespace.json 模板，可以用名称空间覆盖它

```
{
  "kind": "Namespace",
  "apiVersion": "v1",
  "metadata": {
    "name": "${NAMESPACE}",
    "labels": {
      "name": "${NAMESPACE}"
    }
  }
}
```

# 配置图

每个服务都有自己的配置变量，需要在运行时可供服务使用。我决定采用一种传统的方法，混合了 how [。env](https://www.npmjs.com/package/dotenv) 作品。

写入以'. env '结尾的文件的键值对将被组合在一起，并添加到环境变量服务将使用的配置映射中。

将以下列方式为每个服务创建配置映射:

*   `.env/staging.env`将为暂存环境中的所有服务设置变量
*   `.env/service-a.env`将为所有`service-a`设置变量，而不管环境如何
*   `.env/feature-myfeature.env`将为`feature-myfeature`名称空间中的所有服务设置变量
*   `.env/staging/service-a.env`将为暂存环境中的所有`service-a`设置变量
*   `.env/staging/feature-myfeature/service-a.env`将只为运行`feature-myfeature`名称空间的 staging 上的`service-a`设置变量

我意识到，允许基于特性设置配置的粒度，而不是强制将配置写入环境名称，可能有些矫枉过正…但是我觉得这是必要的，以便提供一种方法来测试一个特性，而不会在合并分支时意外覆盖上游配置。

下面是将每个文件组合成最终 env 文件的 python 脚本:

我将在下面的脚本部分讨论如何创建实际的配置图。

# 秘密

每个重要的服务都有一些连接到各种其他服务所必需的秘密，比如密码、api 令牌、随机数等。由于信息的敏感性，您不希望将其作为项目的一部分签入。因此，我们根据运行时定义的 env 变量创建一个 kubernetes 秘密。由于我一直在做的项目使用了 [CircleCI](https://circleci.com/) ，我将讨论这是如何在那个环境中完成的。然而，它应该很容易应用于另一个集成工具，如 [Jenkins](https://jenkins.io/) 。剧本是这样的:

它遍历环境变量，并根据我们要部署到的环境将它们存储为密钥/值。因此，任何用前缀`STAGING_`定义的键都将被写入到登台环境的机密文件中。由于功能部署是登台环境的分支，为登台环境定义的秘密适用于功能部署。

# 服务清单

我在这里采用了传统的方法。将部署在与我们当前环境的名称相匹配的目录下定义的任何 kubernetes 清单。如上所述，特性部署是分支于登台环境的，所以来自登台的清单被应用于特性部署。

# 脚本将所有这些联系在一起

这里发生的事情基本上是这样的:

*   如果`NAMESPACE`不是默认的，创建一个新的名称空间
*   基于设置的环境变量创建一个 kubernetes 秘密
*   结合所有的。env 文件合并到一个文件中，并基于该文件创建一个 kubernetes 配置映射
*   将`MANIFEST_IN_DIR`中定义的所有清单应用到`NAMESPACE`中
*   等待每个`IMAGES`的展示完成

我使用`envsubst`将环境变量注入到我为名称空间和清单创建的模板中。

# 结论

这在很大程度上仍是一项进行中的工作。我希望将大部分内容清理干净并放到我的 github 上，这样它就可以很容易地应用到其他项目中。我省略了部署过程中的一些步骤，比如构建/推送映像、在合并开发分支后清理名称空间，以及部署到生产环境而不是部署到暂存环境。我计划在以后的文章中写下这是如何实现的。