# 如何为 Kubernetes 自定义资源定义生成客户代码(CRD)

> 原文：<https://itnext.io/how-to-generate-client-codes-for-kubernetes-custom-resource-definitions-crd-b4b9907769ba?source=collection_archive---------1----------------------->

![](img/b183f8e3684989d7a66a06411d1b2678.png)

Alejandro Escamilla 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

我们可以使用 Kubernetes 定制控制器来处理核心资源，这太棒了。但是，这些核心资源是不可编辑的。在许多场合，你会想介绍你自己的资源，你会完全控制它。CRD(自定义资源定义)[1]正是为此而设计的。

然而，对于创建的每个 CRD 资源，都有许多样板代码。为了避免这种情况，我们可以使用`k8s.io/code-generator`来自动生成 CRD 工作所需的所有信息、列表等。

> 注[2]: `[kubernetes/code-generator](https://github.com/kubernetes/code-generator)`是从`[k8s.io/code-generator](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/code-generator)`同步的。代码更改在那个位置进行，合并到`k8s.io/kubernetes`，然后同步回`[kubernetes/code-generator](https://github.com/kubernetes/code-generator)`。

在本文中，我们将关注如何从头开始使用生成器。

首先，让我们假设我们想要创建一个属于 foo.com 组的 CRD，并且在规范中有一个名为“HelloType”的带有“message”字段的类型:

```
# Definition
---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:  
  name: hellotypes.foo.com
spec:  
  group: foo.com  
  version: v1
  scope: Namespaced  
  names:    
    kind: HelloType   
    shortNames: ht
    plural: hellotypes
    singular: hellotype# HelloType
---
apiVersion: foo.com/v1
kind: HelloType 
metadata: 
  name: superman-hello
spec:
  message: hello world
```

要生成 CRD 资源代码，需要两个步骤。

1.  用适当的代码生成器标记编写类型定义代码
2.  运行代码生成器，为您的客户资源自动创建客户端代码，包括 clientset、informers、listers

我们将使用最新的 go 1.11.2 进行演练。

## **CRD 类型定义**

首先让我们创建一个名为`github.com/superman/demo`的回购。我们将创建的框架文件布局如下:

```
pkg/
├── apis
    └── foo
        └── v1
            ├── doc.go
            ├── register.go
            └── types.go
```

`foo` 是我们正在定义的新资源组。其下的所有类型都属于`foo.com`。

如果你写的是低于 1.0.0 的资源版本，`v1`下的所有代码都可以放在`foo`目录下。其他版本遵循惯例，在`foo`下单独创建`v1`、`v1beta1`、`v2`等目录。在本例中，我们将为自定义资源创建版本 1。

`foo/v1/doc.go`

这个文件提供了对全局标签的控制，这些标签将应用于相同版本中的每种类型的默认设置。标签位于注释块中。各个类型可以选择退出默认设置或使用不同的标签。这被称为“本地标签”。

标签由两部分组成，函数名和正常传入的值。例如在`+k8s:deepcopy-gen=package`中，`deepcopy-gen`是为类型生成深层副本代码而调用的函数。`package`表示在这种情况下将该功能应用于整个 v1 包。`deepcopy-gen`函数和其他类似的生成器函数可以在“k8s.io/code-generator/cmd/”中找到。通过查找这些目录下的`main.go`文件中的注释，您可以找到相关标签的解释。

如果你对这些代码实际上是如何生成的感兴趣，你可以进一步研究"k8s.io/gengo"。

文件中的另外两个标记指示为 struct TypeMeta 生成 defaulter，并将组名设置为“foo.com”。

`foo/v1/types.go`

在这个文件中，我们创建了自定义类型“HelloType”。我们还定义了自定义的`Status`和`Spec`。任何自定义类型也必须有复数结构。在这种情况下，我们有“HelloTypeList”。

对于我们在此文件中使用的标签:

`+genclient`表示为该类型生成 API 客户端。

`+k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object`表示为类型生成 deepcopy 时，使用运行时。对象接口。

`+optional`表示该字段是可选的。该字段的数据可以为空。

`foo/v1/register.go`

`register.go`文件包含将我们刚刚创建的新类型注册到模式中的函数，这样 API 服务器就可以识别它们。

**客户端代码生成**

现在我们有了代码生成所需的所有文件。go 1.11+的这个过程有几个选项。

选项 1:不带 Go 模块

默认情况下，`$GOPATH/src`下的所有代码都不会使用 Go 模块。假设你把你的代码放在`$GOPATH/src/github.com/superman/demo`里。

然后运行下面的代码来获取生成器和代码库:

```
$ go get k8s.io/code-generator$ go get k8s.io/apimachinery
```

进入`$GOPATH/src/k8s.io/code-generator`，运行以下命令:

```
$ ./generate-groups.sh all \
    "github.com/superman/demo/pkg/client" \
    "github.com/superman/demo/pkg/apis" \
    foo:v1
```

选项 2:使用 Go 模块

Go 1.11 中引入了 go 模块。它提供了一种处理依赖关系的直观方式。您可以将您的存储库放在任何地方，包括在`GOPATH`之外。

不幸的是，目前`code-generator`没有处理围棋模块。因此，让它工作起来有点棘手。

一种解决方案是使用 docker。下面是 Dockerfile 文件和 shell 脚本的一个例子。同样，假设你的围棋模块名是`github.com/superman/demo`。

使用上述选项之一完成代码生成后，您的`pkg`文件夹结构将类似于:

```
pkg/
├── apis
│   └── foo
│       ├── register.go
│       └── v1
└── client
    ├── clientset
    │   └── versioned
    ├── informers
    │   └── externalversions
    └── listers
        └── foo
```

新创建的`client`文件夹包含所有生成的代码，类型的 deepcopy 代码也将驻留在类型旁边的`v1`中。

最后，您可以像通常使用其他核心资源一样，为定制控制器使用生成的 clientset 和 informers。

**资源:**

1.  [Openshift:代码生成深度挖掘](https://blog.openshift.com/kubernetes-deep-dive-code-generation-customresources/)
2.  [创建控制器和 CRD](https://medium.com/@trstringer/create-kubernetes-controllers-for-core-and-custom-resources-62fc35ad64a3)
3.  [创建自定义控制器](https://medium.com/@cloudark/kubernetes-custom-controllers-b6c7d0668fdf)
4.  标签(非排气) :

**参考:**

1.  [Kubernetes 自定义资源](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
2.  [kubernetes/代码生成器](https://github.com/kubernetes/code-generator)

*免责声明:本文仅代表作者个人观点，绝不代表任何组织的观点。*