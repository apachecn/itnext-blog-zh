# 使用 kubernetes 配置映射，第 1 部分:卷挂载

> 原文：<https://itnext.io/working-with-kubernetes-configmaps-part-1-volume-mounts-f0ace283f5aa?source=collection_archive---------0----------------------->

一些想法。

如果您编写在 kubernetes 上运行的应用程序，那么您至少有机会顺便(或更多)熟悉 golang。Kubernetes 毕竟是用 go 编写的，围绕 kubernetes/docker 涌现出了许多也是用 go 编写的工具，go 在更广泛的云开发社区中获得了很大的吸引力。

如果您编写在 kubernetes 上运行的应用程序，那么您几乎 100%会对 configmaps 有所了解(如果您不了解，您是如何做到这一点的)。

配置映射对于在 kubernetes 上运行应用程序是必不可少的，因为它们提供了管理应用程序配置的主要机制。还有其他的(例如容器规范中的环境变量)，但是使用配置映射有足够的优势，大多数开发人员最终会习惯使用它们。

我开始写这篇文章的目的是假设你对 configmap 的基础知识相当熟悉，并快速了解使用`knative.dev/pkg/configmap`包是多么容易。(确实如此，我将会谈到这一点，只是没有之前预想的那么快)。但是今天我和一个新的开发人员一起工作，她是我们从大学毕业就为我们团队雇佣的，她几周前才加入我们。很明显，我认为理所当然的概念，至少对 kubernetes 新手来说，并不像我认为的那样简单，我认为我们应该从应用程序如何与配置图交互的背景开始。

# 背景

首先，我们需要回顾一些关于如何使用`kubectl`创建配置图的基础知识。我将介绍从文件创建配置图的两种方法，以及结果配置图之间的结构差异。你可以在官方的 kubernetes 文档[中找到更多关于创建配置图的例子和信息。](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

以下所有示例中使用的代码(包括应用程序和脚本)和文件都可以在我位于 https://github.com/ScarletTanager/configmap-watcher-example 的资源库中找到。您可以自由地使用和修改我放在这里的任何示例，当我探索一些在 go 中操作/读取配置映射的进一步技术时，我可能会添加更多的示例。如果您只想运行示例，那么您可以通过将环境变量`REGISTRY`的值设置为 docker Hub 注册表并从存储库根目录运行`build`脚本来为示例创建 Docker 映像。这将创建并推送三个 docker 映像，每个示例应用程序一个。

我还将在`knative.dev/pkg/configmap`中展示和讨论一小部分代码，因为该包也提供了说明性的例子和一些非常易于使用的类型/函数来为 configmaps 工作。

# 创建配置映射并通过卷装载使用它

访问存储在配置图中的信息有两种主要方法。第一种方法是将 configmap 作为卷挂载到应用程序容器中，然后您可以使用标准的文件 I/O 调用来读取它。第二个是通过 kubernetes API 检索配置图。我不打算讨论这两种方法的优缺点——它们都有自己的位置，我在自己的代码中使用了这两种方法。

大多数人需要做的第一件事是在应用程序启动时加载配置图。由于 kubernetes 使得将配置图挂载为一个卷成为可能，让我们看看这是如何实现的。在`configmap-watcher-example`存储库中有一个文件`clusters.properties`，它包含一组`key=value`对，如下所示:

```
current.target=cluster1.endpoint
cluster1.endpoint=[https://foo.com](https://foo.com)
cluster2.endpoint=[https://bar.com](https://bar.com)
cluster3.endpoint=[https://foobar.com](https://foobar.com)
cluster4.endpoint=[https://free.willy](https://free.willy)
cluster5.endpoint=[https://take.me.to.funkytown](https://take.me.to.funkytown)
```

如果您运行命令`kubectl create cm clusters-config-env --from-env-file=clusters.properties`，您将得到一个配置图，其`data`部分如下所示:

```
data:
  cluster1.endpoint: [https://foo.com](https://foo.com)
  cluster2.endpoint: [https://bar.com](https://bar.com)
  cluster3.endpoint: [https://foobar.com](https://foobar.com)
  cluster4.endpoint: [https://free.willy](https://free.willy)
  cluster5.endpoint: [https://take.me.to.funkytown](https://take.me.to.funkytown)
  current.target: cluster1.endpoint
```

现在，我们可以使用与*几乎*相同的命令`kubectl create cm clusters-config-file --from-file=clusters.properties`创建一个配置图，我们最终会得到稍微不同的结果:

```
data:
  clusters.properties: |
    current.target=cluster1.endpoint
    cluster1.endpoint=[https://foo.com](https://foo.com)
    cluster1.name=mycluster
    cluster2.endpoint=[https://bar.com](https://bar.com)
    cluster3.endpoint=[https://foobar.com](https://foobar.com)
    cluster4.endpoint=[https://free.willy](https://free.willy)
    cluster5.endpoint=[https://take.me.to.funkytown](https://take.me.to.funkytown)
```

有什么区别？

在第一种情况下，我们使用了`--from-env-file`标志——使用该标志要求文件内容采用`key=value`对的形式，每行一个(标准的环境文件格式)。在`data`部分，键与文件中出现的键相同——所以我们的映射与原始文件具有相同的键/值对。

在第二个例子中，我们使用了`--from-file`标志。有了这个标志，文件内容格式就没有真正的限制了。为什么？看上面第二张地图的`data`部分，名为`clusters-config-file`:只有**一个**键存在，而且是原文件的名字:`clusters.json`。因此，只有一个值，该值是包含文件全部内容(包括换行符)的单个字符串。

为什么这很重要？因为如何创建配置映射将影响如何从应用程序中读取其内容。我们将在以下章节中参考`clusters-config-file`和`clusters-config-env`配置图。

# 将配置图装入应用程序容器

在我们第一次尝试读取配置图时，我们将在 kubernetes 中创建一个应用程序，并对其进行配置，使其能够从文件挂载中读取配置图。我们将创建一个使用`cm-loader`映像(使用我前面提到的`build`脚本构建和上传)的 pod，将 configmap `clusters-config`挂载为一个卷，然后将该卷挂载到主应用程序容器中。完成这些工作的 pod 规范的 yaml 代码片段如下所示(我去掉了任何不直接相关的内容，实际的 pod 规范将包含更多信息):

```
spec:
  containers:
    image: index.docker.io/sandycash/cm-loader
    volumeMounts:
    - mountPath: /clusters-config
      name: clusters-config-volume
  volumes:
    - configMap:
        name: clusters-config-file
      name: clusters-config-volume
```

该代码片段指定执行以下操作:

1.  使用 DockerHub 中的指定图像创建一个容器；
2.  使配置图`clusters-config-file`成为名为`clusters-config-volume`的可挂载卷；和
3.  将该卷装入路径为`/clusters-config`的容器

我们将在第一个例子中使用的应用程序是从我的存储库中的`load/load.go`源文件构建的——下面是源代码:

```
package mainimport (
 "encoding/json"
 "fmt"
 "net/http""knative.dev/pkg/configmap"
)const (
 CM_NAME = "clusters-config"
)var (
 namespace string
)func main() {
 cfg, err := configmap.Load(CM_NAME)
 if err != nil {
  panic("Unable to load config map!")
 } else {
  http.HandleFunc("/config", func(w http.ResponseWriter, r *http.Request) {
   body, err := json.Marshal(cfg)
   if err != nil {
    w.WriteHeader(http.StatusInternalServerError)
   } else {
    w.WriteHeader(http.StatusOK)
    w.Write(body)
   }
  })
 }fmt.Printf("Listening on port 8080\n")
 http.ListenAndServe(":8080", nil)
}
```

这是一个非常简单的应用程序。它使用 knative 的`configmap`包中的`Load()`方法加载一个 configmap 我将在这里暂停一下，说这是一种简单而有用的方法，我们许多人已经(重新)实现了几十次，现在我再也不会实现它了。该方法将配置图的数据部分作为类型`map[string]string`的值返回。然后，应用程序创建一个 HTTP 处理程序，该处理程序简单地将地图整理成 JSON 并打印出来。如果我们`curl`这个应用程序上的`/config`端点并将结果传输到`jq`(只是为了格式化)，我们会得到:

```
{
  "clusters.properties": "current.target=cluster1.endpoint\ncluster1.endpoint=[https://foo.com\ncluster1.name=mycluster\ncluster2.endpoint=https://bar.com\ncluster3.endpoint=https://foobar.com\ncluster4.endpoint=https://free.willy\ncluster5.endpoint=https://take.me.to.funkytown\n](https://foo.com\ncluster1.name=mycluster\ncluster2.endpoint=https://bar.com\ncluster3.endpoint=https://foobar.com\ncluster4.endpoint=https://free.willy\ncluster5.endpoint=https://take.me.to.funkytown\n)"
}
```

就像 configmap 本身一样:一个键，它是原始文件的名称，一个值——文件的全部内容是一个字符串，包括换行符。

现在让我们运行我们的应用程序，但是安装`clusters-config-env`地图，这需要将 pod 规范的`volumes`部分改为:

```
spec:
  [snip]
  volumes:
    - configMap:
        name: clusters-config-env
      name: clusters-config-volume
```

我们不需要改变`volumeMounts`部分，因为我们正在挂载不同的地图，但是*在相同的挂载点*。当我们现在向`/config`端点发出 GET 调用时，我们得到:

```
{
  "cluster1.endpoint": "[https://foo.com](https://foo.com)",
  "cluster1.name": "mycluster",
  "cluster2.endpoint": "[https://bar.com](https://bar.com)",
  "cluster3.endpoint": "[https://foobar.com](https://foobar.com)",
  "cluster4.endpoint": "[https://free.willy](https://free.willy)",
  "cluster5.endpoint": "[https://take.me.to.funkytown](https://take.me.to.funkytown)",
  "current.target": "cluster1.endpoint"
}
```

就像在原始的`clusters-config-env` configmap 的`data`部分一样，我们为原始文件中的每一对都有一个键/值对。

好吧，我明白了，但我为什么要在乎呢？

简短的回答是，因为每个选项更适合不同的情况，而且，正如我之前所说的，您选择哪个选项将影响您如何使用配置图。

在我们讨论这些方法的优缺点之前，让我们先简单地了解一下为什么会这样——为什么一种方法甚至使用文件名作为键，而另一种方法实际上解析文件？

# configmap 卷装载的工作方式

要回答前面的问题，我们需要考虑“将配置图作为卷挂载”意味着什么如果您还记得，我们将我们的挂载点指定为`/clusters-config`——这是您的应用程序在通过文件系统访问这个配置图时将引用的名称。一个关键的方面是*挂载点总是目录*。因此应用程序实际上将配置图视为包含一个或多个文件的目录(假设您的配置图的`data`部分不为空)。这些文件来自哪里？我们从未创建过任何“文件”，不是吗？

答案是这样的:配置图中的每个键都作为配置图挂载点下的一个单独的文件公开。即使该值只是一个相对较短的字符串，就像我们的`clusters-config-env`映射中的情况一样，每个键都将作为一个单独的文件出现，并且该文件的内容将是该键在原始配置映射中的值。

在我们的`clusters-config-file`映射中，单个键(原始文件的名称)显示为一个文件…与原始文件同名。内容是…原始文件的内容。因此，在这种情况下，对这些内容的解析完全留给了应用程序。但是`clusters-config-env`有多个键/值对，这意味着多个(相比之下非常小)文件。

为了说明这一点，让我们看看`knative.dev/pkg/configmap.Load()`的源代码:

```
func Load(p string) (map[string]string, error) {
 data := make(map[string]string)
 err := filepath.Walk(p, func(p string, info os.FileInfo, err error) error {
  if err != nil {
   return err
  }
  for info.Mode()&os.ModeSymlink != 0 {
   dirname := filepath.Dir(p)
   p, err = os.Readlink(p)
   if err != nil {
    return err
   }
   if !filepath.IsAbs(p) {
    p = path.Join(dirname, p)
   }
   info, err = os.Lstat(p)
   if err != nil {
    return err
   }
  }
  if info.IsDir() {
   return nil
  }
  b, err := ioutil.ReadFile(p)
  if err != nil {
   return err
  }
  data[info.Name()] = string(b)
  return nil
 })
 return data, err
}
```

`Load()`采用单个参数——应用程序已知的配置图的“名称”,这意味着卷的挂载点。它将 configmap 的内容作为一个`map[string]string`返回。为了构建该地图，它执行以下操作:

1.  它创建一个映射来保存返回值。
2.  它调用`filepath.Walk()` —这个函数从最顶端的节点开始，遍历一个目录树。它有两个参数:一个是起始路径(我们的 configmap 挂载点)，另一个是回调函数，该函数为树中的每个节点调用一次。
3.  回调有三个参数:当前节点的路径、文件名和一个错误(例如，没有足够的权限`stat()`该文件)。回调执行以下操作:
    1 .如果我们遇到错误，返回(跳过这个节点)。
    2。如果节点是一个符号链接，获取目标，规范化路径，然后尝试`stat()`目标。
    3。如果节点是一个目录，跳过它(我们只关心常规文件)。
    4。读取文件的内容，然后将它们作为一个值添加到我们的地图中，以文件名作为关键字。

# 我应该使用哪种方法，何时使用？

当创建要作为卷装载的配置图时，在某些情况下只有一个选项就可以了。一个明显的例子是，如果您的文件不是有效的`env`类型格式(意味着它不是一组`key=value`对，每行一个)。例如，如果您在 JSON 或 YAML 中有一个配置文件，则不能将这些文件中的单个值直接作为配置映射中的值公开。因此，除非您想在创建映射之前预处理您的配置数据，否则您必须从配置映射中将每个文件作为单个条目单独加载，然后您的应用程序需要能够解组和使用内容本身。

除了偏好之外，将配置存储为 JSON 或 YAML 的一个优点是能够将配置直接解组为某种结构化类型，这种类型可能有多个层次。所以让我们考虑一下文件`config.json:`

```
{
  "current": {
    "target": "cluster1"
  }
}
```

和`clusters.json`:

```
{
  "cluster1": {
    "endpoint": "[https://foo.com](https://foo.com)"
  },
  "cluster2": {
    "endpoint": "[https://bar.com](https://bar.com)"
  },
  "cluster3": {
    "endpoint": "[https://foobar.com](https://foobar.com)"
  },
  "cluster4": {
    "endpoint": "[https://free.willy](https://free.willy)"
  },
  "cluster5": {
    "endpoint": "[https://take.me.to.funkytown](https://take.me.to.funkytown)"
  }
}
```

这些都是将`clusters.properties`(不是唯一可能的)改写成一对 JSON 文件的琐碎工作。也许我们希望将这些解组成类似(同样，不是唯一的可能性):

```
type MyConfig struct {
  Current HostSpecification `json:"current"`
}type HostSpecification struct {
  Endpoint string `json:"endpoint,omitempty"`
  Target   string `json:"target,omitempty"`
}type Clusters map[string]HostSpecification
```

我可以将这两个配置文件存储在一个配置图中，如下所示:

```
apiVersion: v1
kind: ConfigMap
data:
  clusters.json: |
    {
      "cluster1": {
        "endpoint": "[https://foo.com](https://foo.com)"
      },
      "cluster2": {
        "endpoint": "[https://bar.com](https://bar.com)"
      },
      "cluster3": {
        "endpoint": "[https://foobar.com](https://foobar.com)"
      },
      "cluster4": {
        "endpoint": "[https://free.willy](https://free.willy)"
      },
      "cluster5": {
        "endpoint": "[https://take.me.to.funkytown](https://take.me.to.funkytown)"
      }
    }
  config.json: |
    {
      "current": {
        "endpoint": "cluster1"
      }
    }
metadata:
  name: clusters-config-multiple-files
```

当我们挂载地图并对应用程序的`/config`端点发出 GET 命令时，我们得到的结果是可以预见的:

```
{
  "clusters.json": "{\n  \"cluster1\": {\n    \"endpoint\": \"[https://foo.com\](https://foo.com\)"\n  },\n  \"cluster2\": {\n    \"endpoint\": \"[https://bar.com\](https://bar.com\)"\n  },\n  \"cluster3\": {\n    \"endpoint\": \"[https://foobar.com\](https://foobar.com\)"\n  },\n  \"cluster4\": {\n    \"endpoint\": \"[https://free.willy\](https://free.willy\)"\n  },\n  \"cluster5\": {\n    \"endpoint\": \"[https://take.me.to.funkytown\](https://take.me.to.funkytown\)"\n  }\n}\n",
  "config.json": "{\n  \"current\": {\n    \"endpoint\": \"cluster1\"\n  }\n}\n"
}
```

我们可以加载如下内容:

```
cfgData, _ := configmap.Load("clusters-config-multiple-files")var myConfig MyConfig
json.Unmarshal(cfgData["config.json"], &myConfig)clusters := make(Clusters)
json.Unmarshal(cfgData["clusters.json"], &clusters)
```

所以总而言之:

1.  如果您有一个相对简单的配置，或者您希望通过一个键直接访问每个值，而不需要维护潜在的多个数据类型来存储您的配置，请使用 env 文件格式，其中 configmap 是从配置键到值的直接映射。
2.  如果您有一个不止一层的配置，您可以在环境风格的配置中使用带点的属性名(这就是我们在`clusters.properties`中所做的),并使用`--from-env-file=<PATH>`语法加载您的 configmap。然后，您可以像在(1)中那样继续通过按键直接访问这些值。
3.  如果您的配置不止一层，和/或您更喜欢在 JSON/YAML 中维护它，例如，因为这有助于直接在您自己的数据类型中解组(通常但不总是`structs`)，您的 configmap 将文件内容存储为 blob，您将使用`--from-file=<PATH>`语法创建它。
4.  如果您有多个配置文件(任何格式)，并且您想将它们都存储在一个 configmap 中，那么最简单的方法就是使用一个类似于上面的示例`clusters-config-multiple-files`的规范来创建您的 configmap，然后您可以自动检索每个文件的内容作为给定键的值，并相应地处理它们。

# 在第 2 部分中…

我将把注意力转向通过 Kubernetes API 加载配置映射——直接使用 Kubernetes 客户端和 knative 的`configmap`包提供的其他工具。我将特别关注一些简单的模式，通过这些模式，您可以使您的应用程序能够动态地处理对配置图的更改，并且如果有人意外删除了配置图，那么*和*不会受到影响。