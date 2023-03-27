# 使用 Fabric8 Tekton 客户端访问 Java 中的 Tekton 管道

> 原文：<https://itnext.io/access-tekton-pipelines-in-java-using-fabric8-tekton-client-bd727bd5806a?source=collection_archive---------7----------------------->

![](img/6f8fc6c970d4c4727c35fbb5b811bc63.png)

[Fabric8 Tekton 客户端](https://github.com/fabric8io/kubernetes-client/tree/master/extensions/tekton)

Fabric8 Kubernetes 客户端还为 [Istio](https://github.com/snowdrop/istio-java-api) 、 [Knative](https://knative.dev/) 、 [Tekton](https://tekton.dev/) 和[服务目录](https://kubernetes.io/docs/concepts/extend-kubernetes/service-catalog/)提供了非常有用的扩展。从那以后，它已经进化了很多。今天，我将简要介绍 Fabric8 Tekton extension。

[](https://github.com/fabric8io/kubernetes-client) [## fabric 8 io/kubernetes-客户端

### 这个客户端通过一个流畅的 DSL 提供对完整的 Kubernetes 和 OpenShift REST APIs 的访问。

github.com](https://github.com/fabric8io/kubernetes-client) 

# 获取 Fabric8 Tekton 客户端:

你可以像往常一样在 [Maven Central](https://search.maven.org/search?q=tekton-client) 上找到 Tekton 客户端。您可以通过将 Tekton client 作为依赖项添加到您的`pom.xml`中来开始使用它:

```
<dependencies>
  <dependency>
    <groupId>io.fabric8</groupId>
    <artifactId>tekton-client</artifactId>
    <version>${tekton-client.version}</version>
  </dependency>
</dependencies>
```

Tekton 客户端的大部分用法都很容易理解，并且与其母 Fabric8 Kubernetes 客户端相似。这是使用`v1alpha1`或`v1beta1` apiVersions 的一个非常基本的顶级概述:

结构中的 DSL 入口点 8 Tekton 客户端

# 列出给定命名空间中的所有管道:

一旦添加为依赖项，您就可以开始使用 Fabric8 Tekton 客户端。让我们从列出所有的`Pipeline`对象开始。以下是使用 Fabric8 Tekton 客户端的操作方法:

列出默认命名空间中的所有管道

# 使用 Fabric8 的构建器创建 Tekton 资源:

因为 Tekton 客户端基于 Tekton 模型，该模型是通过将 Tekton 源代码中的 go 结构解析成 JSON 模式，然后在其上使用 [sundrio](https://github.com/sundrio/sundrio/) 而形成的。我们得到了非常有用的构建器对象，使用它们我们可以快速构建 Tekton 资源，而不需要从任何 Yaml 清单中加载任何东西。下面是一个创建 hello world `Task`和`TaskRun`的例子:

使用构建器创建 hello world 任务

# 将 YAML·泰克顿清单载入 Java 对象:

Fabric8 Tekton 客户端，就像 Fabric8 Kubernetes 客户端一样，在将 YAML 清单序列化到 Java 对象或从 Java 对象反序列化时，提供了无缝的开发人员体验。假设您有这样一个简单的`Pipeline`(摘自 [Tekton Github 库示例](https://github.com/tektoncd/triggers/blob/master/examples/example-pipeline.yaml)):

简单管道 YAML 清单

如果我想将这个`Pipeline`加载到一个 Java 对象中，然后将它应用到 Kubernetes 上，我会这样做:

将 YAML 清单加载到 Java 对象

# 创建、更新和删除 Tekton 资源:

如果你已经熟悉 Fabric8 Kubernetes 客户端 API，你就不会面临任何标准操作的问题。如果你正在访问一些资源(例如，`Task`)。您可以使用以下方式访问它:

CRUD 操作概述

让我们来看一个使用 Fabric8 Tekton 客户端的 CRUD 操作的工作示例。我们将加载、编辑并最终删除一个`Route`对象。以下是这一系列操作的代码:

tekton.dev/v1beta1#Task 的 CRUD 操作

# 创建触发器:

Fabric8 Tekton 客户端最近支持 [Tekton 触发器](https://github.com/tektoncd/triggers)。现在，您可以使用 Fabric8 Tekton Client 轻松定义管道触发器。让我们考虑一个例子，假设您使用 Fabric8 Tekton Client 创建了这个简单的 maven deployer 管道，它使用 [Eclipse JKube](https://github.com/eclipse/jkube) 部署任何基于 maven 的应用程序:

现在，如果我想使用 Fabric8 Tekton 客户端创建一个`TriggerTemplate`、`TriggerBinding`和`EventListener`。我会这样做:

这将创建所有提到的资源。EventListener 还会创建一个`Pod`，您可以使用`kubectl port-forward $(kubectl get pod -o=name -l eventlistener=jkube-listener) 8080`将它向前移植。这将暴露您接受连接的触发器。

这就是我对 Fabric8 Tekton 客户端的简短概述。我希望你喜欢它。请试用并提供反馈。我在这个 Github 资源库中提供了与这个博客相关的所有代码:

[](https://github.com/rohanKanojia/tekton-java-client-demo) [## rohanKanojia/tek ton-Java-客户端-演示

### 这个项目通过创建一个简单的管道来演示 Fabric8 Tekton Java 客户端的使用，该管道使用 Eclipse JKube 来…

github.com](https://github.com/rohanKanojia/tekton-java-client-demo) 

# 加入我们:

你喜欢 Fabric8 Kubernetes 的客户，并希望参与项目的开发。随意参与方法:

*   当事情没有按预期运行时，创建 [Github 问题](https://github.com/fabric8io/kubernetes-client/issues)
*   给我们发[拉请求](https://github.com/fabric8io/kubernetes-client/pulls)来修复错误/添加增强功能
*   在我们的 [Gitter 频道](https://gitter.im/fabric8io/kubernetes-client?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)与我们聊天
*   在推特上关注我们