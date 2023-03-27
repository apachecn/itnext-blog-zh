# 如何通过 FlexVolume 在 Kubernetes 中创建自定义持久卷插件(第 2 部分)

> 原文：<https://itnext.io/how-to-create-a-custom-persistent-volume-plugin-in-kubernetes-via-flexvolume-part-2-c6390f9f94e0?source=collection_archive---------6----------------------->

> 这是一个两部分的系列。 [**第一部分**](/how-to-create-a-custom-persistent-volume-plugin-in-kubernetes-via-flexvolume-part-1-f6d9d966e123) :司机。**第二部**:补给者

![](img/9b7c667900b047553ba5d8c4a9dc161a.png)

照片由 [Alexandre Debiève](https://unsplash.com/@alexkixa?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

在 [**第一部分**](/how-to-create-a-custom-persistent-volume-plugin-in-kubernetes-via-flexvolume-part-1-f6d9d966e123) 中，我已经谈到了如何基于 FlexVolume 的规范编写一个定制的永久卷(PV)驱动程序，它使人们能够创建与定制的 PV 生命周期相集成的静态 PV。

在这一部分中，我将带您了解如何创建一个动态供应器，它将使用在第一部分中创建的驱动程序并动态供应您的 PVs。

管理静态 PV 是一个手动过程，需要电子表格类型的工作来跟踪 PV 库存。这种方法有很多开销，但是它确实有一些有用的用途。例如，它允许您创建一个简单的备份和灾难恢复(DR)计划，因为所有的 PV 都是已知的，并根据使用情形进行了逻辑标记。它还可以通过仅向群集用户提供有限的库存来充当基础配额。

尽管静态 PV 有其优点，但在许多其他情况下，使用动态供应器更为常见，因为现代应用程序(尤其是微服务环境)大多是无状态的。有状态数据存储在缓存或数据库中，或者当 pod 重新启动时，它可以很容易地恢复到 PVs 上。

我们将利用[外部存储](https://github.com/kubernetes-incubator/external-storage)库(在 [Go](https://golang.org/) 中编写)来创建一个动态供应器。它为构建树外持久性卷供应器提供了一个简单的界面。它基本上是一个使用 client-go 库的定制 Kubernetes 控制器(对于感兴趣的人，我将在另一篇文章中详细介绍如何创建自己的定制 Kubernetes 控制器)。

有几个步骤涉及到:

*   实现两个外部存储器的接口
*   整理家属
*   封装二进制文件并运行您的部署清单

## 提供者

你需要做几件关键的事情[1]:

1.  导入“github . com/kubernetes-孵化器/外部-存储/库/控制器”
2.  为您的置备程序实现两个接口:

```
Provision(VolumeOptions) (*v1.PersistentVolume, error)Delete(*v1.PersistentVolume) error
```

基本上，您的供应器将观察 PVC 的生命周期，并在相应事件发生时进行供应和删除。

`Provision`方法处理所有需要的业务逻辑，并返回相应的 PV(类似于 PV 清单的规范要求)。例如，业务逻辑将包括提供物理持久性卷，从而可以避免手动管理。

`delete`方法将根据 PV 的保留策略指定的内容删除物理持久性卷。

如果您计划运行共享同一存储类的多个不同的置备程序实例，您将需要为每个置备程序提供一个身份，通过注释将该身份添加到返回的 PV 中，并在删除 PV 时对其进行检查。需要注意的是，如果您删除了其中一个置备程序，则该置备程序创建的 PVs 将是无父级的，并且由于身份不匹配，不会由其余置备程序处理。

下面是示例代码(**不是*一个完整的实现)来帮助你入门。一个是主要的，另一个是提供者。***

## *谈谈 Go 的包装管理*

*尽管 **dep** 是官方的实验，但是没有一个通用的包管理工具可以像其他语言一样管理 Go。由于历史原因，许多应用程序都使用其他包管理工具[2]。比如 Glide 和 Godep。现在，Vgo 有了一个新的提议，但仍不清楚何时会正式安装一个通用的包管理工具[更新:似乎 Vgo 被批准了，[但社区对其方法有分歧](https://codeengineered.com/blog/2018/golang-godep-to-vgo/#comment-3912763285)][更新 2:考虑使用 [go 模块](https://blog.golang.org/using-go-modules)。尽管 Dep 努力提供一个通用的接口，但是这些工具之间并不兼容，因此仍然可以导入带有不同软件包管理工具的旧回购协议。然而，它仍然有问题，并不总是与使用不同包管理工具的旧回购。*

## *家属*

*外部存储使用[滑动](https://github.com/Masterminds/glide)作为包管理工具。因此，您的新回购也需要使用 glide。*

*要引导 glide，只需运行:*

```
*$ glide init*
```

*该命令将生成一个`glide.yaml`文件，除了`version`之外，如下图所示。您可能希望指定[外部存储发布版本](https://github.com/kubernetes-incubator/external-storage/releases)，这样您就可以确保您的 provisioner 可以在正确版本的 Kubernetes 上运行。*

```
*package: github.com/your-github-username/awesome-repo
import:
- package: github.com/golang/glog
- package: github.com/kubernetes-incubator/external-storage
  subpackages:
  - lib/controller
  - lib/util
  **version: v1.10.beta**
- package: k8s.io/api
  subpackages:
  - core/v1
- package: k8s.io/apimachinery
  subpackages:
  - pkg/api/resource
  - pkg/apis/meta/v1
  - pkg/util/wait
- package: k8s.io/client-go
  subpackages:
  - kubernetes
  - rest*
```

*在`glide.yaml`文件准备好之后，您需要运行下面的命令将依赖项安装到供应商文件夹中，这样代码就可以被编译，而不是依赖于`$GOPATH`中的库:*

```
*$ glide install -v*
```

***注意:你必须使用** `**-v**` **标志，这样嵌套的供应商文件夹将被 glide 移除，否则你的编译将会失败。***

## *汇编*

*您可以使用普通的`go build`命令编译您的代码。然而，在创建 docker 映像时，您需要使用已经安装的基本容器。*

*另一种方法是编译静态链接的二进制文件，例如为 linux 编译:*

```
*CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo ldflags '-w' -o awesome-provisioner*
```

*这样做的好处是您可以使用任何基于 linux 的映像，包括`scratch`。*

## *完成*

*在容器化了您的 provisioner 之后，剩下要做的一件事就是编写一个`deployment`清单，然后简单地将它部署到您的 Kubernetes 集群。请注意，如果您要将机密传递给`FlexVolume`，您将需要由您的置备程序创建或引用相关的机密。*

## *参考*

*[1] [编写一个树外动态供应器](https://github.com/kubernetes-incubator/external-storage/blob/master/docs/demo/hostpath-provisioner/README.md)*

*[1] [Go 包管理工具](https://github.com/golang/go/wiki/PackageManagementTools)*