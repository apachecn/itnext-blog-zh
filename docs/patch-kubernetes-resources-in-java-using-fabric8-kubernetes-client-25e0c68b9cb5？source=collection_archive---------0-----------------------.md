# 使用 Fabric8 Kubernetes 客户端在 Java 中修补 Kubernetes 资源

> 原文：<https://itnext.io/patch-kubernetes-resources-in-java-using-fabric8-kubernetes-client-25e0c68b9cb5?source=collection_archive---------0----------------------->

![](img/6f8fc6c970d4c4727c35fbb5b811bc63.png)

[Fabric8 Kubernetes 客户端](https://github.com/fabric8io/kubernetes-client)

在今天的博客中，我们将探讨如何使用 Fabric8 Kubernetes Client 在 java 中以编程方式就地更新 Kubernetes API 对象。在我们的最新版本中，我们对这个补丁 API 做了很多改进，我认为写一篇关于它的博客是个不错的主意。为了使用 Fabric8 Kubernetes 客户端，您需要将 Fabric8 Kubernetes 客户端作为项目中的依赖项。如果这是一个 maven 项目，您需要添加以下内容:

```
**<dependency>
  <groupId>**io.fabric8**</groupId>
  <artifactId**>kubernetes-client**</artifactId>
  <version>**6.1.1**</version>
</dependency>**
```

一旦添加了依赖项，您就可以开始使用客户端了。为了使用补丁 API。您需要在 kubernetes 集群中已经存在一些资源。我使用 minikube 作为本地 kubernetes 集群进行测试。让我们创建一个简单的部署，稍后我们将使用补丁 API 对其进行修改。以下是创建部署的代码:

为我们的补丁测试创建示例部署

如果您运行这个示例，您将能够看到正在创建的部署。查看与您的部署相关联的窗格:

```
$ kubectl get pods
```

输出显示该部署有两个 pod。`1/1`表示每个 Pod 有一个容器:

```
NAME                        READY     STATUS    RESTARTS   AGE
patch-demo-28633765-670qr   1/1       Running   0          23s
patch-demo-28633765-j5qs3   1/1       Running   0          23s
```

此时，每个 Pod 都有一个运行 nginx 映像的容器。现在假设您希望每个 Pod 有两个容器:一个运行 nginx，一个运行 redis。

按照我们旧的补丁 API，您需要提供一个完整的对象，其中包含修改过的补丁对象，以便能够打补丁。下面是使用旧 API 添加另一个容器的示例:

Fabric8 Kubernetes 客户端旧补丁 API

运行此代码并查看修补后的部署:

```
**$** kubectl get deploy patch-demo  -ojsonpath='{.spec.template.spec.containers}' | jq . 
**[** 
 **{** 
 **"image":** "nginx"**,** 
 **"imagePullPolicy":** "Always"**,** 
 **"name":** "patch-demo-ctr"**,** 
 **"resources": {},** 
 **"terminationMessagePath":** "/dev/termination-log"**,** 
 **"terminationMessagePolicy":** "File" 
 **},** 
 **{** 
 **"image":** "redis"**,** 
 **"imagePullPolicy":** "Always"**,** 
 **"name":** "patch-demo-ctr-2"**,** 
 **"resources": {},** 
 **"terminationMessagePath":** "/dev/termination-log"**,** 
 **"terminationMessagePolicy":** "File" 
 **}** 
**]**
```

如你所见，我们的`Deployment`现在有两个容器，而不是一个，这意味着我们的补丁工作了。Kubernetes 支持三种补丁策略，早期的 Fabric8 Kubernetes 客户端没有为我们提供任何选项，只做了 [JSON 补丁(RFC 6902)](https://datatracker.ietf.org/doc/html/rfc6902) 。

让我们来看一下最新版本中所有新的补丁选项。

# 使用策略合并补丁进行补丁操作:

Fabric8 Kuberenetes 客户端还提供了一个`patch(String patchAsStr)`方法，该方法接受原始 JSON/YAML 字符串，您可以提供该字符串进行修补。默认情况下，它使用战略合并补丁(这只是一个定制版本的 [JSON 合并补丁(RFC 7396)](https://datatracker.ietf.org/doc/html/rfc7396) )。下面是如何用这个新方法完成上述操作:

为战略合并补丁提供 JSON 字符串

您可以通过在这个 patch 方法中提供一个额外的`PatchContext`来覆盖它，它相当于 [client-go 的 PatchOptions](https://github.com/kubernetes/apimachinery/blob/master/pkg/apis/meta/v1/types.go#L554) :

使用 PatchContext 提供补丁类型

运行上述代码后，您应该能够看到与之前全身补丁示例相同的结果。

# 使用 JSON 补丁进行补丁操作:

假设我们想将这个`Deployment`的第一个容器映像更新为`nginx:mainline`，我们想使用 JSON 补丁来完成这个任务。我们可以简单地通过在`PatchContext`中指定`PatchType.JSON`作为补丁类型来实现，如下所示:

在补丁操作中使用 JSON 补丁

当您运行这段代码时，您应该能够看到我们的`Deployment`图像正在更新:

```
**$** kubectl get deploy patch-demo  -ojsonpath='{.spec.template.spec.containers}' | jq . 
**[** 
 **{** 
 **"image":** "nginx:mainline"**,** 
 **"imagePullPolicy":** "Always"**,** 
 **"name":** "patch-demo-ctr-2"**,** 
 **"resources": {},** 
 **"terminationMessagePath":** "/dev/termination-log"**,** 
 **"terminationMessagePolicy":** "File" 
 **},** 
 **{** 
 **"image":** "nginx"**,** 
 **"imagePullPolicy":** "Always"**,** 
 **"name":** "patch-demo-ctr"**,** 
 **"resources": {},** 
 **"terminationMessagePath":** "/dev/termination-log"**,** 
 **"terminationMessagePolicy":** "File" 
 **}** 
**]**
```

# 使用 JSON 合并补丁进行补丁操作:

使用 Json 合并补丁也非常类似于使用上述两种补丁技术。我们只需要在`PatchContext`中指定`PatchType.JSON_MERGE`为补丁类型。让我们尝试使用 Json Merge patch 删除一个注释。我们的部署现在有以下注释:

```
**$** kubectl get deploy patch-demo  -ojsonpath='{.metadata.annotations}' | jq . 
**{** 
 **"app":** "nginx"**,** 
 **"deployment.kubernetes.io/revision":** "3"**,** 
 **"foo":** "bar" 
**}**
```

让我们试着用键`foo`删除注释，下面是我们的做法:

使用 JSON 合并补丁更新部署

当您运行这段代码时，您应该能够看到正在更新的注释:

```
$ kubectl get deploy patch-demo  -ojsonpath='{.metadata.annotations}' | jq . 
**{** 
 **"app":** "nginx"**,** 
 **"deployment.kubernetes.io/revision":** "6" 
**}**
```

# 结论:

在今天的博客中，您看到了如何利用 Fabric8 Kubernetes 客户端补丁 API 来更改任何 Kubernetes 对象的实时配置。您可以在这个资源库的[PatchExamples.java](https://github.com/rohanKanojia/kubernetes-client-demo/blob/master/src/main/java/io/fabric8/PatchExamples.java)中找到所有这些代码:

[](https://github.com/rohanKanojia/kubernetes-client-demo) [## rohanKanojia/kubernetes-客户端-演示

### 该项目包含 Fabric8 Kubernetes 客户端不同用法的各种示例。我通常在我的…

github.com](https://github.com/rohanKanojia/kubernetes-client-demo)