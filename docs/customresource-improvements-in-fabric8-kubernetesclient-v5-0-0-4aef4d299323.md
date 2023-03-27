# Fabric8 中的自定义资源改进 KubernetesClient v5.0.0

> 原文：<https://itnext.io/customresource-improvements-in-fabric8-kubernetesclient-v5-0-0-4aef4d299323?source=collection_archive---------1----------------------->

![](img/6f8fc6c970d4c4727c35fbb5b811bc63.png)

Fabric8 Kubernetes Java 客户端

Fabric8 Kubernetes 客户端最近发布了 5.0.0 版本，重点改进了对 CustomResource 和 Kubernetes 运营商相关使用的支持。它已经集成到了 Java Operator SDK 中。

在这篇博客中，我们将详细了解所有新的改进。我们还将了解 Fabric8 KubernetesClient 5.x 与 4.x 版本在使用上的差异。对于这个博客，让我们考虑一个简单的名为`Book`的定制资源:

```
**apiVersion:** testing.fabric8.io/v1alpha1 
**kind:** Book 
**metadata:** 
  **name:** effective-java 
**spec:** 
  **title:** "Effective Java" 
  **author:** "Joshua Bloch" 
  **isbn:** "0134685997"
```

它基于这个 customresourcediation`books.testing.fabric8.io`:

```
**apiVersion:** apiextensions.k8s.io/v1 
**kind:** CustomResourceDefinition 
**metadata:** 
  **name:** books.testing.fabric8.io 
**spec:** 
  **group:** testing.fabric8.io 
  **versions:** 
    - **name:** v1alpha1 
      **served:** true 
      **storage:** true 
      **schema:** 
        **openAPIV3Schema:** 
          **type:** object 
          **properties:** 
            **spec:** 
              **type:** object 
              **properties:** 
                **title:** 
                  **type:** string 
                **author:** 
                  **type:** string 
                **isbn:** 
                  **type:** string 
            **status:** 
              **type:** object 
              **properties:** 
                **issued:** 
                  **type:** boolean 
                **issuedto:** 
                  **type:** string       
      **subresources:** 
        **status:** {}   
  **scope:** Namespaced 
  **names:** 
    **plural:** books 
    **singular:** book 
    **kind:** Book 
    **shortNames:** 
    - book
```

您可以通过使用`kubectl`或使用 Fabric8 Kubernetes 客户端在您的 Kubernetes 集群中创建这个`CustomResourceDefinition`(注意:您需要 ClusterAdmin 权限才能创建一个`CustomResourceDefinition`):

使用 Fabric8 Kubernetes 客户端创建 CRD 图书

好的，一旦`CustomResourceDefinition`被应用，我们就可以向前迈进，检验 fabric 8 Kubernetes Client v 5 . 0 . 0 API 的新改进。

# 为 Book CustomResource 定义 POJOs:

## 4.x 行为:

在 KubernetesClient 的 4.x 版本中，您需要提供 4 种不同的 POJOs。例如，我们正在为`Book` CustomResource 创建 CustomResource 客户端，我们需要创建这些 POJOs:

*   [Book.java](https://github.com/rohanKanojia/librarybookoperatorinjava/blob/quarkus-1-10-5-final/src/main/java/io/fabric8/testing/model/v1alpha1/Book.java)->POJO for`Book`自定义资源
*   [BookList.java](https://github.com/rohanKanojia/librarybookoperatorinjava/blob/quarkus-1-10-5-final/src/main/java/io/fabric8/testing/model/v1alpha1/BookList.java)->POJO 为`Book`的列表类型
*   [DoneableBook.java](https://github.com/rohanKanojia/librarybookoperatorinjava/blob/quarkus-1-10-5-final/src/main/java/io/fabric8/testing/model/v1alpha1/DoneableBook.java)->sund Rio 中可完成接口所需的 POJO

在`Book` CustomResource POJO 中，我们需要为`spec`和`status`提供这两个额外的 POJO:

*   [BookSpec.java](https://github.com/rohanKanojia/librarybookoperatorinjava/blob/quarkus-1-10-5-final/src/main/java/io/fabric8/testing/model/v1alpha1/BookSpec.java)->POJO 为`Book`的`spec`
*   【BookStatus.java】的 - > POJO 为`Book`的`status`

让我们继续，看看我们是如何在 client 的 5.x 版本中定义 POJOs 的。

## 5.x 行为:

在 Fabric8 Kubernetes Client 5.x 中，我们已经去掉了`Doneable`接口，现在您不需要为您的 CustomResource 提供列表类型。它被保持为可选的，如果你不提供它，KubernetesClient 将假定 list 是类型`KubernetesResourceList<Book>`(对于类型`Book`的 CustomResource)。这意味着您只需要提供这些 POJOs:

*   Book.java-> POJO for`Book`自定义资源
*   BookSpec.java-> POJO 为`Book`的`spec`
*   BookStatus.java-> POJO 为`Book`的`status`

您也不需要为`CustomResourceDefinition`相关信息提供`CustomResourceDefinitionContext`对象。现在，您可以在 CustomResource POJO 本身中以注释的形式提供这些信息。到目前为止，支持这些注释:

*   `@Group`:允许指定自定义资源 ApiGroup
*   `@Version`:允许指定自定义资源 ApiVersion
*   `@Kind`:(可选)允许指定种类来引用注释类的实例。如果没有提供，则根据带注释的类名计算默认值。
*   `@Plural`:(可选)允许指定与自定义资源相关的复数形式。如果没有提供，它将默认为一个计算值。
*   `@Singular`:(可选)允许指定与自定义资源相关的单数形式。如果没有提供，它将默认为基于 POJO 类名的计算值。

由于每个 CustomResource 都需要`.spec`和`.status`字段，我们已经将 KubernetesClient 中的 [CustomResource](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-client/src/main/java/io/fabric8/kubernetes/client/CustomResource.java) 类修改为一个通用类，它将分别需要 spec 和 status 类型。如果这听起来有点混乱，请不要担心，当您按照 Fabric8 Kubernetes Client 5.0.0 查看清理后的 POJOs 版本时，会清楚得多。

让我们看看我们的 POJOs 如何寻找`Book` CustomResource:

**Book.java:**

根据 Fabric8 Kubernetes 客户端 5.0.0 在 POJO 一书中提供的注释

你可以看到我们如何使用`@Version`和`@Group`注释来指定我们的 CRD `apiGroup`和`apiVersion`。`Book`正在实现`Namespaced`接口，因为它是一个命名空间的作用域资源。

**BookSpec.java:**

BookSpec POJO

**BookStatus.java:**

博若书店

# 正在创建图书客户端:

## 4.x 行为:

一旦您创建了所有这些 POJO，您就可以创建这个`Book` CustomResource 客户端，如下所示。

注意，为了创建`Book` CustomResource 客户端，我们必须提供`CustomResourceDefinitionContext`(包含关于`Book` CustomResource 的`CustomResourceDefinition`的详细信息的对象)、`Book.class`、`BookList.class`和`DoneableBook.class`，这看起来不太可读。

在 Fabric8 Kubernetes 客户端的 4.x 版本中创建图书自定义资源客户端

## 5.x 行为:

在 Fabric8 Kubernetes Client v5.0.0 中，我们在`client.customResources()`方法中去掉了这些不必要的参数。您可以看到，之前需要 4 个参数的 DSL 方法现在只需要一个参数，它是特定 CustomResource 的类。是否提供`BookList`类型完全由你决定。所有与`CustomResourceDefinition`相关的信息要么是从提供的注释中挑选出来的，要么是基于自以为是的缺省值构建的。

在 Fabric8 5.x 中创建图书客户端的新方法

好了，除了如何定义 POJOs，以及如何创建对应于 CustomResource 的类型化客户端(本例中为`Book`)；DSL 的其他用法与 Fabric8 Kubernetes Client 4.x 版本相同。

让我们来看看我们可以用`Book`的 CustomResource 客户端完成的其他常见操作。

## 将自定义资源从 YAML 加载到 Java 对象:

将自定义资源 YAML 清单加载到 Book 对象中

## 列出指定命名空间中的所有 CustomResource:

列出指定命名空间中的所有 CustomResource 对象

## 正在获取某个命名空间中具有特定名称的 CustomResource:

在名称空间“default”中获取一个 Book CustomResource，并将其命名为“effective-java”

## 创建自定义资源:

创建图书自定义资源

## 更新自定义资源:

更新现有 CustomResource 中的批注

## 更新 CustomResource 中的状态子资源:

更新图书自定义资源的状态子资源

## 监视指定命名空间中的所有自定义资源:

监视指定命名空间中的 CustomResource

## 删除自定义资源:

删除某个命名空间中具有指定名称的自定义资源

今天的博客到此结束。我希望我能够向您概述 Fabric8 Kubernetes Client v5.0.0 中的所有自定义资源改进。如果您使用的是 4.x 版本的 Kubernetes Client，您还可以查看 5.0.0 的迁移指南:

[](https://github.com/fabric8io/kubernetes-client/blob/master/doc/MIGRATION-v5.md) [## fabric 8 io/kubernetes-客户端

### 为了减少客户端生成的类的数量和包的整体大小，Doneable 接口…

github.com](https://github.com/fabric8io/kubernetes-client/blob/master/doc/MIGRATION-v5.md) 

您可以在这个 github 资源库找到本博客中链接的所有源代码:

[](https://github.com/rohanKanojia/kubernetes-client-demo) [## rohanKanojia/kubernetes-客户端-演示

### 该项目包含 Fabric8 Kubernetes 客户端不同用法的各种示例。我通常在我的…

github.com](https://github.com/rohanKanojia/kubernetes-client-demo) 

# 结论:

再次感谢你花时间阅读这篇博客！我希望它有帮助。如果您喜欢我们的项目，请随时通过以下渠道加入我们，成为我们不断发展的社区中的一员:

*   创建 [Github 问题](https://github.com/fabric8io/kubernetes-client/issues)当事情不按预期工作时
*   给我们发[拉请求](https://github.com/fabric8io/kubernetes-client/pulls)来修复错误/添加增强功能
*   如有疑问，请通过我们的 [Gitter 频道](https://gitter.im/fabric8io/kubernetes-client?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)与我们聊天
*   在推特上关注我们