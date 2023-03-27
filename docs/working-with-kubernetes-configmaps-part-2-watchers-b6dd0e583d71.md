# 使用 kubernetes 配置图，第 2 部分:观察器

> 原文：<https://itnext.io/working-with-kubernetes-configmaps-part-2-watchers-b6dd0e583d71?source=collection_archive---------0----------------------->

![](img/54d56ecde4ff5875d61e8ccf536ae753.png)

# 介绍

在我的上一篇文章《使用 kubernetes 配置图，第 1 部分:卷挂载》中，我讨论了从容器中的卷挂载加载 kubernetes 配置图的机制，以及用于创建配置图的方法如何转化为数据的呈现方式。

在这篇文章中，我将讨论一个处理配置图的具体策略，即使用一个**观察器**。Kubernetes API 资源都支持一个[观察提要](https://kubernetes.io/docs/reference/using-api/api-concepts/#efficient-detection-of-changes)，它允许 API 客户端监控他们感兴趣的资源，被告知变化，然后相应地采取行动(以他们希望的任何方式)。因此，监视这种资源的客户机代码就是这里的**观察器**，在下面的讨论中，我将展示一些使用观察器来处理配置图的具体例子，这些例子比通过卷安装来使用配置图要动态得多。

在我们继续之前，让我澄清一下，您完全可以为挂载的 configmap 实现一个观察器。 [kubelet 保持挂载更新](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#mounted-configmaps-are-updated-automatically)(有一点点延迟，在默认配置中最多两分钟)，您当然可以像监视任何其他文件一样监视挂载文件系统中的文件的变化。

但是使用 kubernetes(甚至在混合使用 knative 时更是如此)鼓励的核心模式之一是事件驱动编程。然而，监视文件的更改更像是一个监视问题——监视代码必须知道文件的状态，然后基本上轮询文件的任何更改。例如，它变大了还是变小了，或者计算出的校验和发生了变化？在事件驱动的模型中，客户端更多地扮演被动的角色——“我想知道任何事件，只要让我知道它们何时发生。”这使我们能够编写更简单、更轻便的代码，并专注于我们真正关心的问题，而不是如何检测例如给定类型的资源的变化。

幸运的是，kubernetes(如上所述)本身提供了对事件驱动模型的支持，使用这种模型来消费 configmaps 让我们编写的代码不仅比监视文件更改的代码更简单，而且更加健壮和容错。在本文和后续的文章中，我将展示三个例子来说明如何做到这一点——第一个例子，这里介绍的，将使用 golang 的标准 [kubernetes 客户端库，第二个和第三个(将在本系列的下一篇文章中讨论)将使用我在第 1 部分中使用的](https://github.com/kubernetes/client-go) [knative configmap 包](https://pkg.go.dev/knative.dev/pkg/configmap)的不同部分。

# 在哪里可以找到样本

我将在这里展示的所有示例(代码和配置)都可以在我位于 https://github.com/ScarletTanager/configmap-watcher-example[的 GitHub repo 中找到。如果您认为合适，您可以自由复制/使用该代码的任何部分。我们将要编写的应用程序的完整源代码位于](https://github.com/ScarletTanager/configmap-watcher-example)[https://github . com/scarlet manager/config map-watcher-example/blob/main/watch/watch . go](https://github.com/ScarletTanager/configmap-watcher-example/blob/main/watch/watch.go)。

对于其中的每个示例，我将使用以下命令从上述存储库的根目录创建配置映射:

`kubectl create cm clusters-config --from-env-file=configs/clusters.properties`

这将生成一个起始配置图(我们在示例中对其进行了修改),其数据部分如下所示:

```
data:
  cluster1.endpoint: [https://foo.com](https://foo.com)
  cluster2.endpoint: [https://bar.com](https://bar.com)
  cluster3.endpoint: [https://foobar.com](https://foobar.com)
  cluster4.endpoint: [https://free.willy](https://free.willy)
  cluster5.endpoint: [https://take.me.to.funkytown](https://take.me.to.funkytown)
  current.target: cluster1.endpoint
```

# 不仅仅是配置图

如果您曾经使用过 kubernetes API，无论是直接通过编写自己的客户端库还是使用 [client-go](https://github.com/kubernetes/client-go) 库，您可能已经知道观察器不是特定于远程配置图的。事实上，您可以修改本文中的大部分代码，使其适用于几乎所有受支持的 kubernetes 资源类型。我在这里是在特定于 configmap 的上下文中讨论该模式的，但是我接下来要说的几乎所有内容都同样适用于 pod、服务、服务帐户等。

# 使用 client-go 构建观察器

让我们从一个基本目的开始:构建一个简单的 REST 端点，它像这样返回一个 JSON 有效负载:

```
{"current_endpoint": "https://somehost.domain"}
```

其中`somehost.domain`是通过`data[data["current.target"]]`访问时我们的 configmap 生成的 URL。在我之前的文章中，我们通过使用`configmap.Load()`从一个卷装载一次 configmap 来完成这个任务，但是这一次我们将使用一种不同的方法。我们从一些基本的管道开始:

```
package mainimport (
  "fmt"
  "io/ioutil"
  "net/http"

  "k8s.io/client-go/kubernetes"
  "k8s.io/client-go/rest"
)const (
  NS_FILE          = "/var/run/secrets/kubernetes.io/serviceaccount/namespace"
  CM_NAME          = "clusters-config"
  DEFAULT_ENDPOINT = "[https://wazanga.partytime](https://wazanga.partytime)"
)var (
  namespace string
)func init() {
  nsBytes, err := ioutil.ReadFile(NS_FILE)
  if err != nil {
    panic(fmt.Sprintf("Unable to read namespace file at %s", NS_FILE))
  } namespace = string(nsBytes)
}
```

我们已经声明了我们的导入，我们还设置了一些常量，其中最重要的是`DEFAULT_ENDPOINT`，我们将其定义为`https://wazanga.partytime`。我们还声明了一个变量`namespace`，然后我们用 kubernetes 名称空间的值来填充这个变量，我们的代码在这个名称空间中执行。这都是很标准的东西。

接下来，我们开始填充我们的`main()`函数:

```
func main() {
  var (
    currentEndpoint string
  ) // Let's make sure we don't forget to set a default
  currentEndpoint = DEFAULT_ENDPOINT // Get things set up for watching - we need a valid k8s client
  clientCfg, err := rest.InClusterConfig()
  if err != nil {
    panic("Unable to get our client configuration")
  } clientset, err := kubernetes.NewForConfig(clientCfg)
  if err != nil {
    panic("Unable to create our clientset")
  }
```

所以在这里，我们将变量`currentEndpoint`初始化为`DEFAULT_ENDPOINT`的值(我在前面已经指出了)，然后创建一个配置为从集群内部运行的 kubernetes 客户端。我们正在使用股票 kubernetes 客户端库， [client-go。对于大多数 kubernetes 客户端用例来说，这是您可能会用到的库，没有理由不在这里使用它。要配置客户端从集群外部进行身份验证，请参见这些说明](https://github.com/kubernetes/client-go)。

现在，我们将通过注册 http 处理程序并启动监听器来完成程序的其余部分:

```
 http.HandleFunc("/config", func(w http.ResponseWriter, r *http.Request) {
    body := []byte(fmt.Sprintf(`{"current_endpoint": "%s"}`, currentEndpoint))
    w.WriteHeader(http.StatusOK)
    w.Write(body)
  }) fmt.Printf("Listening on port 8080\n")
  http.ListenAndServe(":8080", nil)
}
```

现在，如果我们运行这个，我们将在输出中得到`DEFAULT_ENDPOINT`的值，也就是`https://wazanga.partytime`。原因应该很明显(先不说这实际上还不会编译，因为我们没有对`clientset`变量做任何事情)。但这实际上不是我们想要的—我们想要从配置图中获取端点值。

# 添加观察器(事件处理)循环

因此，让我们添加观察器代码。我们需要更多的进口商品，最终我们得到了这些:

```
import (
  "context"
  "fmt"
  "io/ioutil"
  "net/http"
  "sync" corev1 "k8s.io/api/core/v1"
  metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
  "k8s.io/apimachinery/pkg/watch"
  "k8s.io/client-go/kubernetes"
  "k8s.io/client-go/rest"
)
```

然后，在`main()`的主体中，我们为`sync.Mutex`添加了一个新变量:

```
func main() {
  var (
    currentEndpoint string
    mutex           *sync.Mutex
  )
```

接下来，我们在 goroutine 中添加一行来启动我们的监视器，并在读取变量时使用我们的`sync.Mutex`来锁定`currentEnvironment`变量:

```
mutex = &sync.Mutex{} 
go watchForChanges(clientset, namespace, &currentEndpoint, mutex) http.HandleFunc("/config", func(w http.ResponseWriter, r *http.Request) {
   mutex.Lock() // ADDED THIS
   body := []byte(fmt.Sprintf(`{"current_endpoint": "%s"}`, currentEndpoint))
   mutex.Unlock() // AND ADDED THIS
```

对于我们的 watcher 代码，我们传入 kubernetes 客户端、我们的名称空间、指向我们的`currentEnvironment`变量的指针和我们的互斥体，因为我们将在不同的 goroutine 中写入一个共享变量。

我们的 watcher 函数是一个简单的循环，它创建一个`watch.Interface`(这是变量`watcher`的值)的实例，然后阻塞，直到`updateCurrentEndpoint()`返回，然后无限重复相同的过程。

```
func watchForChanges(clientset *kubernetes.Clientset, namespace string, endpoint *string, mutex *sync.Mutex) {
  for {
    watcher, err := clientset.CoreV1().ConfigMaps(namespace).Watch(
      context.TODO(),
      metav1.SingleObject(metav1.ObjectMeta{
        Name: CM_NAME, Namespace: namespace}))
    if err != nil {
      panic("Unable to create watcher")
    }
    updateCurrentEndpoint(watcher.ResultChan(), endpoint, mutex)
  }
}
```

这种模式(可以用其他方式实现)是一个非常重要的方面——kubernetes 观察器打开一个 HTTP 连接，向 kubernetes API 服务器发出 GET 调用。显然，API 服务器不会永远保持连接打开，它会在某个时候关闭连接(该值可由集群管理员配置，但对于我们这些编写客户端代码的人来说，只需假设它会在以分钟为单位的某个时间间隔关闭)。所以你的代码*必须*能够检测到连接已经关闭并以某种方式重新打开。

查看`updateCurrentEndpoint()`的实现将展示我们如何做到这一点，以及我们如何处理收到的事件。

```
func updateCurrentEndpoint(eventChannel <-chan watch.Event, endpoint *string, mutex *sync.Mutex) {
  for {
    event, open := <-eventChannel
    if open {
      switch event.Type {
      case watch.Added:
        fallthrough
      case watch.Modified:
        mutex.Lock()
        // Update our endpoint
        if updatedMap, ok := event.Object.(*corev1.ConfigMap); ok {
          if endpointKey, ok := updatedMap.Data["current.target"]; ok {
            if targetEndpoint, ok := updatedMap.Data[endpointKey]; ok {
              *endpoint = targetEndpoint
            }
          }
        }
        mutex.Unlock()
      case watch.Deleted:
        mutex.Lock()
        // Fall back to the default value
        *endpoint = DEFAULT_ENDPOINT
        mutex.Unlock()
      default:
        // Do nothing
      }
    } else {
      // If eventChannel is closed, it means the server has closed the connection
      return
    }
  }
}
```

首先要注意的是参数列表——第一个参数`eventChannel`是类型`<-chan watch.Event` —意味着一个只读通道，我们将从该通道接收`watch.Event`实例。我们通过`watcher.ResultChan()`获得这个通道 kubernetes 客户端通过 HTTP 连接在后台接收到 API 服务器的事件提要，然后将各个事件写入这个通道。关键的是，*每当服务器关闭 HTTP 连接时，客户端都会检测到这一点并关闭通道*。这意味着我们可以很容易地检测到服务器是否已经关闭了我们的连接(这将意味着我们需要创建一个新的观察器，并从中获得一个新的事件通道)，这就是我们实际上所做的事情。返回的第一个值是事件(如果存在)，第二个值是布尔值，只要通道打开，该值就为`true`，如果通道关闭，则为`false`。

因此，如果通道关闭，我们只需从`updateCurrentEndpoint()`返回，这将使我们在`watchForChanges()`中循环另一个循环迭代，从而创建一个新的观察器(带有一个新的 API 连接)并获得一个新的通道。

事件处理的核心在`switch`块中。每个事件都属于类型`[watch.Event](https://pkg.go.dev/k8s.io/apimachinery/pkg/watch#Event)`——它有两个字段。一个字段`Type`反映了生成事件的操作—允许的值有:

```
const (
	Added    [EventType](https://pkg.go.dev/k8s.io/apimachinery/pkg/watch#EventType) = "ADDED"
	Modified [EventType](https://pkg.go.dev/k8s.io/apimachinery/pkg/watch#EventType) = "MODIFIED"
	Deleted  [EventType](https://pkg.go.dev/k8s.io/apimachinery/pkg/watch#EventType) = "DELETED"
	Bookmark [EventType](https://pkg.go.dev/k8s.io/apimachinery/pkg/watch#EventType) = "BOOKMARK"
	Error    [EventType](https://pkg.go.dev/k8s.io/apimachinery/pkg/watch#EventType) = "ERROR"
)
```

我们关心其中的三个— `Added`、`Modified`和`Deleted`。第一次创建资源时，会生成类型为`Added`的事件。Kubernetes 保留资源事件历史，其中包括这个初始事件，这样当我们的程序第一次启动时，我们会自动得到配置映射的存在和当前状态的通知——这样我们就不必在事件处理程序循环之外读取当前内容。

让我们用一个真实的例子来测试一下。我们已经在文章的开头创建了 configmap，所以让我们开始我们的程序并结束它的端点:

```
$ curl https://basic-watcher.us-south.codeengine.appdomain.cloud/config
{"current_endpoint": "https://foo.com"}
```

因此，即使配置图是在我们的程序开始之前创建的，kubernetes 也保留了事件，当我们的观察器开始从事件提要中读取时，我们收到了事件，就好像配置图是新创建的一样。

请记住，事件不是永远可用的——需要在一个窗口内检索它们，因此在实际应用中，您可能希望在启动时强制读取配置图。

我们通过一个`fallthrough`同样地处理`Added`和`Modified`——在每种情况下，我们从 configmap 的`Data`部分解析出我们需要的值。当然，我们用`mutex.Lock()...mutex.Unlock()`包围这个部分是为了防止任何潜在的数据竞争。让我们通过使用`kubectl edit cm clusters-config`编辑配置图，将`current.target`的值更改为`cluster4.endpoint`，来展示我们动态检测配置图更改的能力，这*将*导致我们的应用程序返回`[https://free.willy](https://free.willy)`作为当前端点值:

```
$ curl https://basic-watcher.us-south.codeengine.appdomain.cloud/config
{"current_endpoint": "**https://foo.com**"}
$ kc edit cm clusters-config
configmap/clusters-config edited
$ curl https://basic-watcher.us-south.codeengine.appdomain.cloud/config
{"current_endpoint": "**https://free.willy**"}
```

果然，当我编辑配置图，然后卷曲我的应用程序的端点时，更改会立即反映出来，不需要重启应用程序。这提供了许多实用性和便利性，允许应用处理例如动态的配置改变，而不需要进一步的操作员干预、现有请求处理的中断、存储器中状态的丢失等。

我们处理的第三种类型的事件是`Deleted`——在这种情况下，我们通过将端点的值恢复为原始(在这种情况下，硬编码)默认值来处理配置图的删除:

```
$ curl https://basic-watcher.us-south.codeengine.appdomain.cloud/config
{"current_endpoint": "**https://free.willy**"}
$ kc delete cm clusters-config
configmap "clusters-config" deleted
$ curl https://basic-watcher.33d0tay1up0.us-south.codeengine.appdomain.cloud/config
{"current_endpoint": "**https://wazanga.partytime**"}
```

在这里您可以看到这种行为——其效果是通过使用一组固定的默认值，允许应用程序处理资源的删除(或者根本不存在的资源),从而在我的应用程序中构建弹性。另一种可能更现实的处理`Deleted`事件的方法是缓存当前活动的配置——例如，如果有人意外删除了配置图，然后从某个旧的(修改前)状态恢复它，您可能不希望丢失这些修改，因此您可以确定由于删除而缓存的配置不会被后续的`Added`事件覆盖。您只需要花时间找出对您的应用程序和操作环境有意义的流程。

# 那么我应该只写观察器而不是从卷装载中读取吗？

简而言之，没有。我开始考虑为特定的情况编写一个观察器，在这种情况下，我知道我的应用程序需要动态地处理 configmap 更改(或者更好地说，这看起来是解决我的问题的最简单/最好的方法)。如果你确实有这样的情况，那么观察器可以是一个强大的工具。

这种模式的另一个关键优势是*不局限于配置图*。您可以将它用于任何 kubernetes API 资源——pods、secrets 等等。在许多情况下，这些都不是您可以装载的资源，因为卷装载几乎如此容易(甚至根本不可能)。因此，理解观察器(以及您可能出错的地方，比如忘记处理关闭的连接)可以在您的 kubernetes 应用程序中打开许多可能性。

但是对于许多只需要从配置图中读取的应用程序来说，这是多余的——我敢打赌，大多数应用程序可以在启动时从卷挂载中读取配置图，将配置存储在内存中，然后不再处理卷挂载。这是一种简单得多的编码模式，并且它还具有明确(在配置映射和机密的情况下)您的应用程序所依赖的资源的优势，因为那些依赖关系在部署 YAML 中被清楚地表达。

但是如果您发现需要动态处理 kubernetes 资源的变化，我希望这篇文章证明是有用的，至少作为一个起点。在第三部分中，我将展示如何通过使用由`knative.dev/pkg/configmap`及其子包提供的 watcher 实现来*极大地*简化这段代码，在那篇或后续文章中，我希望得到一些 Python 中的例子。