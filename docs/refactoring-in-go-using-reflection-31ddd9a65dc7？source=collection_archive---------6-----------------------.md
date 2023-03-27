# Go 中的重构:使用反射

> 原文：<https://itnext.io/refactoring-in-go-using-reflection-31ddd9a65dc7?source=collection_archive---------6----------------------->

![](img/afe6b73f8aa0f13d098adcd6a4b2adc3.png)

*张诗钟·戈伦德在 Unsplash 上拍摄的照片*

Go 的反射 API 对于许多开发人员来说是相当陌生的，但是在某些场景中它绝对可以派上用场。在本文中，我们将在一个场景中使用 Go 的反射，这个场景应该足够熟悉，可以看到使用反射的实际用途。

# 方案

想象一下——出于测试目的——我们想要定期调用 [Kubernetes 客户端 API](https://github.com/kubernetes/client-go) 中随机成员的方法，以检查集群是否正确响应我们的请求。

因此，当`c`是一个`Kubernetes.Clientset`时，我们想在每个不同的成员中随机调用一个方法`List`。

```
c.ConfigMaps(Namespace).List(v1.ListOptions{}) c.Pods(Namespace).List(v1.ListOptions{}) c.Deployments(Namespace).List(v1.ListOptions{}) 
...
```

如你所见，这是完全相同的呼叫，只是成员不同`ConfigMaps`、`Pods`、`Deployments...`。最后，我们希望有一个函数`getRandomApiFunction`随机返回这些方法中的一个，这样我们就可以调用它。

这里的问题是，这些对象中的每一个都有不同的类型签名，所以我们不能编写泛型代码来处理它们(如果 Go 有泛型，整篇文章会更容易编写……但这是另一个完全不同的问题)。

让我们把手弄脏，看看我们如何能做这件事。

# 我们找到的代码

这是在野外发现的代码:

```
const Namespace = "default"
type apiFunc func(*kubernetes.Clientset) (interface{}, error)

// Store a list of available API functions to test
var api_functions_list = []apiFunc{
 func(c *kubernetes.Clientset) (interface{}, error) { 
  return c.ConfigMaps(Namespace).List(v1.ListOptions{}) 
 },
 func(c *kubernetes.Clientset) (interface{}, error) { 
  return c.Pods(Namespace).List(v1.ListOptions{}) 
 },
 func(c *kubernetes.Clientset) (interface{}, error) { 
  return c.Deployments(Namespace).List(v1.ListOptions{}) 
 },
 func(c *kubernetes.Clientset) (interface{}, error) { 
  return c.ResourceQuotas(Namespace).List(v1.ListOptions{}) 
 },
 func(c *kubernetes.Clientset) (interface{}, error) { 
  return c.ReplicaSets(Namespace).List(v1.ListOptions{}) 
 },
 func(c *kubernetes.Clientset) (interface{}, error) { 
  return c.Secrets(Namespace).List(v1.ListOptions{}) 
 },
 func(c *kubernetes.Clientset) (interface{}, error) { 
  return c.Services(Namespace).List(v1.ListOptions{}) 
 },
 func(c *kubernetes.Clientset) (interface{}, error) { 
  return c.ServiceAccounts(Namespace).List(v1.ListOptions{}) 
 },
 func(c *kubernetes.Clientset) (interface{}, error) { 
  return c.LimitRanges(Namespace).List(v1.ListOptions{}) 
 },
 func(c *kubernetes.Clientset) (interface{}, error) { 
  return c.Ingresses(Namespace).List(v1.ListOptions{}) 
 },
}

func getRandomApiFunction() apiFunc {
 rand.Seed(time.Now().UTC().UnixNano())
 rand_position := math.Mod(float64(rand.Int()), float64(len(api_functions_list)))
 return api_functions_list[int(rand_position)]
}
```

上面的代码是可行的，也足够容易理解，但是有点繁琐。

它从定义一个新的类型`apiFunc`开始:

```
type apiFunc func(*kubernetes.Clientset) (interface{}, error)
```

因此，无论谁实现了`apiFunc`接口，都必须是`kubernetes.Clientset`结构的成员，并且必须返回一个 tuple，其值为 *any* kind(因此有了`interface{}`声明)和一个错误。`List`所有 Kubernetes 客户端 API 中不同对象的方法实现了所有这些限制。

之后，我们找到一个包含几个实现`apiFunc`的相同函数的片`api_functions_list`，每个函数都在不同的 Kubernetes 对象中调用`List`方法。

最后，`getRandomApiFunction`在切片中检索一个随机函数并返回它，以备我们使用。

# 有些东西闻起来很奇怪

这段代码中的味道是调用 Kubernetes API 函数的那部分函数。考虑到所有的功能都几乎相同，这感觉有点多余。如果你有一点像我，那就是我不喜欢。一定有更好的解决办法。

# 让我们反思一下

我们可以在 [Go 博客](https://blog.golang.org/laws-of-reflection)中找到反射的定义:

> *计算中的反射是程序检查自身结构的能力，特别是通过类型；这是元编程的一种形式。这也是一个很大的困惑来源。*

Go 博客中的那篇文章对于理解反射在语言中是如何工作的很有帮助，如果你想理解 Go 中反射的一些内部原理，去看看吧！

## 我们来重构一下！

首先，我们将重新定义`getRandomApifunction`，以便它接受一个`kubernetes.Clientset`并返回一个空接口(这与旧代码中匿名函数的签名相同):

```
func getRandomApiFunction(c *kubernetes.Clientset) (interface{}, error) {
  ...
}
```

然后我们用我们计划调用`List`方法的对象集创建一个切片:

```
var interfaceSlice = []interface{}{
  c.ConfigMaps,
  c.Deployments,
  c.ResourceQuotas,
  c.ReplicaSets,
  c.Services,
  c.ServiceAccounts,
  c.LimitRanges,
  c.Ingresses,
}
```

现在才是有趣的时候。我们希望能够在这些对象中调用`List`，考虑到它们的类型签名。我们创建一个函数`callListMethodOnInterface`:

```
func callListMethodOnInterface(kInterface interface{}) []reflect.Value {
 kInterfaceValue := reflect.ValueOf(kInterface)
 ListMethod := kInterfaceValue.MethodByName("List")
 params := []reflect.Value{reflect.ValueOf(v1.ListOptions{})}
 return ListMethod.Call(params)
}
```

`callListMethodOnInterface`接受一个可以是任何对象的`kInterface`，并返回一个类型为`reflect.Value`的数组。里面发生了什么？

*   `kInterfaceValue := reflect.ValueOf(kInterface)`
    首先我们提取`kInterface`的反映值。这给出了一个我们可以使用反射方法的值。
*   `ListMethod := kInterfaceValue.MethodByName("List")`
    因为我们知道所有这些实体都有一个`List`方法，所以我们在反射值上调用`MethodByName`来检索它。
*   `List`方法在所有情况下都采用`v1.ListOptions{}`，因此我们提取它的反射值并将其放入一个`reflect.Value`数组中。
*   `ListMethod.Call(params)`相当于这个程序第一个版本的调用(例如`c.ConfigMaps(Namespace).List(v1.ListOptions{})`)

现在，我们如何使用这个新创建的`callListMethodOnInterface`函数？与任何其他函数相同，只是我们必须将它返回的反射值展开为最终值:

```
// The randomization code is identical than before
rand.Seed(time.Now().UTC().UnixNano())
randomChoice := rand.Intn(len(interfaceSlice))
randomMethod := interfaceSlice[randomChoice].(apiFunc)

// Here's the call
retv := callListMethodOnInterface(randomMethod(Namespace))
return retv[0].Interface(), retv[1].Interface().(error)
```

`callListMethodOnInterface`返回反射值的数组(`[]reflect.Value`)。在我们的例子中，两个特殊的值:一个`interface{}`和一个潜在的`error`(与`List`方法相同)。为了“拆箱”它们以符合`getRandomApiFunction`签名，我们对每个值调用`Interface()`方法，对于`error`值，我们将其转换为 Go 错误。

我们编写的最终代码是这样的:

```
import (
 "fmt"
 "math/rand"
 "reflect"
 "time"

 "k8s.io/client-go/kubernetes"
 "k8s.io/client-go/pkg/api/v1"
)

type apiFunc func(string) interface{}

func callListMethodOnInterface(kInterface interface{}) []reflect.Value {
 kInterfaceValue := reflect.ValueOf(kInterface)
 ListMethod := kInterfaceValue.MethodByName("List")
 params := []reflect.Value{reflect.ValueOf(v1.ListOptions{})}
 return ListMethod.Call(params)
}

func getRandomApiFunction(c *kubernetes.Clientset) (interface{}, error) {
 var interfaceSlice = []interface{}{
  c.ConfigMaps,
  c.Deployments,
  c.ResourceQuotas,
  c.ReplicaSets,
  c.Services,
  c.ServiceAccounts,
  c.LimitRanges,
  c.Ingresses,
 }

 rand.Seed(time.Now().UTC().UnixNano())
 randomChoice := rand.Intn(len(interfaceSlice))
 randomMethod := interfaceSlice[randomChoice].(apiFunc)
 retv := callListMethodOnInterface(randomMethod(Namespace))

 return retv[0].Interface(), retv[1].Interface().(error)
}
```

# 最后的话

这只是对反射的可能性的一个小测试，但是我喜欢它，因为它展示了如何应用反射来解决一个简单的、常见的问题。反射并不是一个经常出现的领域，但是在某些情况下，当类型系统阻碍了您试图解决的问题时，它会非常有用。

同样重要的是要记住，反射也是搬起石头砸自己的脚，因为它很容易推翻 Go 的类型系统，潜在地使你的程序更难调试。

如果你想知道更多可以使用反射的场景，请查看 Jon Bodner 的这篇文章。它提供了很好的解释和例子，说明了反射在现实世界中的作用，以及它的主要优缺点。

*原载于 2018 年 8 月 24 日 sergimansilla.com*[](https://sergimansilla.com/blog/refactoring-in-go-using-reflection/)**。**