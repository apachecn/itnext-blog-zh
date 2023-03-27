# 测试 Kubernetes Go 应用程序

> 原文：<https://itnext.io/testing-kubernetes-go-applications-f1f87502b6ef?source=collection_archive---------0----------------------->

![](img/d6ca3f91c3ba295ac6467498063eb864.png)

你可以在我的新书中了解更多关于使用 Kubernetes API 和操作符的信息:[https://bit.ly/3vsuwI9](https://bit.ly/3vsuwI9)

Kubernetes 应用程序可以用 Go 语言编写，使用`client-go`库:【https://github.com/kubernetes/client-go】。这个库提供了测试用它编写的应用程序的工具。让我们来探索这些工具。

你可以在一个专门的 github 知识库上找到这篇文章的完整来源:[https://github.com/feloy/testing-k8s-go-apps](https://github.com/feloy/testing-k8s-go-apps)

# kubernetes 套餐

让我们首先检查一下`kubernetes`包，它提供了用于创建新的`Clientset`对象的`NewForConfig`方法，需要连接到 Kubernetes 集群。

`Clientset`类型实现了同一个包中定义的`Interface`接口，该类型的所有方法都属于该接口。因此，可以创建这个接口的另一个实现来替换真实的实现。

感谢`client-go`开发者，他们在`kubernetes/fake`包中完成了这项工作。这个包提供了一个新的`fake.Clientset`类型，也实现了这个接口。这个包还提供了一个`NewSimpleClientset`方法，可以用来创建一个假的客户端集，这对测试我们的应用程序非常有用。

# 使用`Interface`，卢克！

为了测试您的应用程序，您将不得不使用真实的(来自`kubernetes`包)`Clientset`用于您的应用程序，使用假的(来自`kubernetes.fake`包)用于您的测试。

出于这个原因，你必须声明你的方法使用`Clientset`来代替使用`Interface`类型。因此，您将能够使用任何实现，这取决于您是在应用程序中还是在测试中。

在我们的例子中，我们创建了一个包含类型为`kubernetes.Interface`的`clientset`字段的`k8s`结构。

# 获取服务器版本

我们现在可以开始使用客户机集来处理我们的 Kubernetes 集群了。首先，让我们编写一个获取 Kubernetes 服务器版本的方法。下面是完整的实现:

如果您可以访问 Kubernetes 集群，并且在您的`.kube`路径中有相应的`config`文件，您可以尝试运行您的应用程序:

```
$ go run main.go
v1.9.2
```

# 并测试它

我们现在必须测试我们的`getVersion`方法。为此，我们首先必须在`NewSimpleClientset`方法的帮助下，创建一个`k8s`结构的实例，其中包含一个客户端集的伪实现，而不是真实的实现。然后，我们可以在这个假实例上调用`getVersion`方法，并获得由我们的假 kubernetes 集群返回的版本，它将总是返回相同的默认值。

# 验证服务器版本

我们现在可以编写一个更高级的方法来验证集群服务器是否是预期的版本:

# 并测试它

为了测试它，我们将不得不改变由假客户机集返回的版本值。为此，我们看到假的`Discovery`实现公开了一个我们可以修改的`FakedServerVersion`字段。多亏了这个工具，我们能够测试我们的`isVersion`方法的所有情况:

# 创作，反应！

在某些情况下，我们希望当我们对它进行一些操作时，我们的假集群有所反应。例如，如果我们需要验证用户可以在集群上创建一些部署，我们将必须创建一些`SelfSubjectAccessReview` (SSAR)并检查其状态。当我们创建这样一个 SSAR 时，一个真正的 Kubernetes 集群将用相应的值更新它的状态。假的集群不行，但是我们自己可以做。

# 我可以吗？

这里有一个新方法，使用`SelfSubjectAccessReview`来确定运行应用程序的用户是否可以创建部署:

# 并测试它

和以前一样，我们将首先尝试用一个`NewSimpleClientset`来测试这个方法。在这种情况下，我们创建的 SSAR 不会更新，其状态始终为 false。该值涵盖了我们想要测试用户没有创建部署的授权的情况，但不包括其他情况。

# 测试包装中的假类型

如果我们查看`fake.Clientset`类型的内容，我们可以看到它嵌入了`testing.Fake`类型。这种类型实现了`client.Interface`接口，尤其是公开了一个`AddReactor`方法，用于对某些操作做出反应。这是它的签名:

```
type ReactionFunc func(action [Action](https://godoc.org/k8s.io/client-go/testing#Action)) (handled [bool](https://godoc.org/builtin#bool), ret [runtime](https://godoc.org/k8s.io/apimachinery/pkg/runtime).[Object](https://godoc.org/k8s.io/apimachinery/pkg/runtime#Object), err [error](https://godoc.org/builtin#error))func (c *[Fake](https://godoc.org/k8s.io/client-go/testing#Fake)) AddReactor(verb, resource [string](https://godoc.org/builtin#string), reaction [ReactionFunc](https://godoc.org/k8s.io/client-go/testing#ReactionFunc))
```

`AddReactor`方法首先获得两个参数`verb`和`resource`，它们描述了我们想要对其做出反应的操作。比如在`create`上打一个`SelfSubjectAccessReview`。该方法然后获得最新的参数`reaction`，该参数是将在该操作上执行的方法。这个`reaction`方法返回一个`handled`值，指示这个反应器是否有效地处理了这个案例，一个`ret`对象将替换原来的对象，以及一个`err`，如果我们想要在操作过程中模拟一个错误。

有了这些反应器，我们现在可以处理两种最新的情况，用户可以创建一个部署，或者当 SSAR 的创建引起错误时。

# 结论

有了这些简单的方法，您应该能够开始并测试用 Go 编写的 Kubernetes 应用程序。

我真的很感谢对这些第一方法的任何评论，以及任何到其他方法的链接——我会在发现新方法后尽快完成这篇文章。