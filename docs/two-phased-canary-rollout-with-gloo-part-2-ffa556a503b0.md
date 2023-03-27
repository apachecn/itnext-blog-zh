# Gloo 的两阶段金丝雀展示，第 2 部分

> 原文：<https://itnext.io/two-phased-canary-rollout-with-gloo-part-2-ffa556a503b0?source=collection_archive---------3----------------------->

在[的最后一部分](/two-phased-canary-rollout-with-gloo-part-1-ec5b267cdc9e?source=friends_link&sk=773b1875d156b88171b0266031bbb0d6)中，我们看到了如何使用 Gloo 建立一个两阶段的方法来测试和推出新版本的服务。

*   在第一阶段，您重定向一小部分流量，以便验证新版本的功能。
*   一旦满意，您就进入第二阶段，在此期间，您使用加权目的地将负载逐渐转移到服务的新版本，直到您完成任务并且旧版本可以退役。

现在，我们将研究如何改进该工作流设计，以便我们可以跨许多团队拥有的许多服务进行扩展，同时考虑如何在组织中的不同角色之间划分职责，并确保平台妥善处理配置错误。

![](img/16bef20995b2a0602f3779be59b1992a.png)

# 跨多个团队扩展

正如我们在上一篇文章中看到的，Gloo 使用[虚拟服务](https://docs.solo.io/gloo/latest/introduction/architecture/concepts/#virtual-services)来管理特定域的路由。通过修改虚拟服务对象上的路由，我们能够在 Gloo 中执行我们的两阶段升级工作流。

之前，我们为名为 echo 的单个服务执行工作流。现在，我们将引入另一个由不同团队拥有的服务 foxtrot，并考虑如何将这个工作流扩展到多个团队。

特别是，我们将寻找一种能够满足以下关键目标的方法:

*   避免特定团队或个人的瓶颈
*   限制一个团队破坏另一个团队的服务健康的风险
*   非常适合组织的角色方法、基于角色的访问控制和 Kubernetes

## 选项 1:共享虚拟服务

简单的扩展方法是使用单一的虚拟服务，用相同的资源管理 echo 和 foxtrot 服务的所有路由。

马上，我们可以看到一个问题。因为我们所有的路由都是由一个对象控制的，所以我们要么需要向两个团队授予对该对象的写权限，要么需要通过一个中央管理团队的所有权限。后一种方法很可能是两害相权取其轻，而且随着分散的开发团队数量的增加，您会给中央管理团队带来越来越大的负担。

## 选项 2:跨域分离所有权

我们可以考虑的第一个选择是用不同的域对每个服务建模，以便在不同的对象上管理路由。例如，如果我们的主域是`example.com`，我们可以为每个子域提供一个虚拟服务:`echo.example.com`和`foxtrot.example.com`。

```
apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: echo
  namespace: echo
spec:
  virtualHost:
    domains:
      - 'echo.example.com'
    routes:
      - matchers:
          - prefix: /echo
...
---
apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: foxtrot
  namespace: foxtrot
spec:
  virtualHost:
    domains:
      - 'foxtrot.example.com'
...
```

现在，我们可以更干净地将路线的所有权分散到两个服务中，让每个团队拥有自己的虚拟服务。Gloo 可以很容易地观察两个名称空间，因此您可以将虚拟服务与每个团队拥有的其他资源存储在一起。

然而，在一些组织中(比如联系我的那个用户)，域和路由的所有权是分开的——一个开发运营团队负责 DNS 和证书管理，一个开发团队负责特定的路由。这意味着这两个虚拟服务的所有权仍然是共享的，只是没有在开发团队之间共享。

对于这样的组织，更好的方法是让一个管理团队拥有根虚拟服务，并且该团队能够委托一个或多个路由的所有权。

## 选项 3:通过委派在路由表之间分离所有权

为了解决这些问题，我们将利用 Gloo 中一个名为 delegation 的特性。通过委托，我们可以在一个叫做 [RouteTable](https://docs.solo.io/gloo/latest/guides/traffic_management/destination_types/delegation/) 的不同对象上定义我们的路线。在虚拟服务中，我们可以定义一个委托动作，并引用路由表。这使得我们能够将领域的所有权与特定路线的所有权分离开来，并且将不同开发团队中不同路线的所有权分离开来。

这听起来是最令人满意的方法，所以现在让我们看看这在 Gloo 中是如何工作的。

# 设置您的环境

我假设你对[第 1 部分](https://www.solo.io/blog/two-phased-canary-rollout-with-gloo/)中详述的两阶段方法有一个很好的理解。我还假设您有一个 Kubernetes 集群，并且已经按照我在第 1 部分中的解释安装了 Gloo，并且准备开始部署服务。

为了更容易地遵循本指南，我建议运行以下步骤，将所有引用的资源放入您的工作目录中。所有的 kubectl 命令都采用这个工作目录:

```
git clone [https:*//github.com/solo-io/gloo-ref-arch*](https://github.com/solo-io/gloo-ref-arch)cd gloo-ref-arch
git checkout blog-21-apr-20
cd two-phased-canary/part2
```

# 部署应用程序

首先，我们将把 echo 部署到 echo 名称空间:

```
kubectl apply -f echo.yaml
```

我们将把 foxtrot 部署到 foxtrot 名称空间:

```
kubectl apply -f foxtrot.yaml
```

我们应该等艾可和狐步舞准备好再继续前进。

## 在 Gloo 中将服务建模为上游

让我们将 echo 建模为 Gloo 中的[上游](https://docs.solo.io/gloo/latest/introduction/architecture/concepts/#upstreams)目的地:

```
apiVersion: gloo.solo.io/v1
kind: Upstream
metadata:
  name: echo
  namespace: gloo-system
spec:
  kube:
    selector:
      app: echo
    serviceName: echo
    serviceNamespace: echo
    servicePort: 8080
    subsetSpec:
      selectors:
        - keys:
            - version
```

并将其部署到集群:

```
kubectl apply -f upstream-echo.yaml
```

让我们对狐步舞做同样的事情，将其建模为上游:

```
apiVersion: gloo.solo.io/v1
kind: Upstream
metadata:
  name: foxtrot
  namespace: gloo-system
spec:
  kube:
    selector:
      app: foxtrot
    serviceName: foxtrot
    serviceNamespace: foxtrot
    servicePort: 8080
    subsetSpec:
      selectors:
        - keys:
            - version
```

并将其部署到集群:

```
kubectl apply -f upstream-foxtrot.yaml
```

## 设置路由表

现在，让我们创建一个路由表，其中包含到回送上游的路由:

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: echo-routes
  namespace: echo
spec:
  routes:
    - matchers:
        - prefix: /echo
      routeAction:
        single:
          upstream:
            name: echo
            namespace: gloo-system
          subset:
            values:
              version: v2
```

并将其部署到集群:

```
kubectl apply -f rt-echo-1.yaml
```

让我们创建狐步舞的路由表:

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: foxtrot-routes
  namespace: foxtrot
spec:
  routes:
    - matchers:
        - prefix: /foxtrot
      routeAction:
        single:
          upstream:
            name: foxtrot
            namespace: gloo-system
          subset:
            values:
              version: v1
```

并将其部署到集群:

```
kubectl apply -f rt-foxtrot-1.yaml
```

值得注意的是，我们将这些路由表资源分别存储在 echo 和 foxtrot 名称空间中。这是为了模拟 echo 和 foxtrot 团队直接拥有这些资源。我们可以假设当我们对那些资源进行变更时，我们正在模拟那些团队。

## 用虚拟服务把它连接起来

最后，我们可以通过创建一个虚拟服务，将两个服务连接到一个特定的域(在本例中是`*`)。在我们的场景中，我们假设一个中央运营团队拥有该域，因此我们将把这个配置保存在`gloo-system`名称空间中。

```
apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: app
  namespace: gloo-system
spec:
  virtualHost:
    domains:
      - '*'
    routes:
      - matchers:
          - prefix: /echo
        delegateAction:
          selector:
            namespaces:
              - echo
      - matchers:
          - prefix: /foxtrot
        delegateAction:
          selector:
            namespaces:
              - foxtrot
```

并将其部署到集群:

```
kubectl apply -f vs.yaml
```

## 测试路线

此时，我们应该能够通过 Gloo 向这两个服务发送请求:

```
➜ curl $(glooctl proxy url)/echo
version:echo-v1➜ curl $(glooctl proxy url)/foxtrot
version:foxtrot-v1
```

# 运行两阶段 canary 工作流

现在，echo 或 foxtrot 团队可以通过编辑相应的路由表，在他们的服务上运行两阶段工作流。我们将模仿狐步舞团队，开始推出 v2。

## 狐步舞队开始第一阶段

对于第一阶段，我们需要部署 foxtrot 服务的 v2，然后更新路由表，向 v2 发送包含报头`stage: canary`的请求。

我们可以部署 foxtrot-v2 并等待它被部署:

```
kubectl apply -f foxtrot-v2.yaml
```

等到命名空间“foxtrot”中的 pod 准备就绪。使用`kubectl get pods -n foxtrot`检查状态。

然后我们需要更新路线，这样我们就可以开始测试 foxtrot v2:

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: foxtrot-routes
  namespace: foxtrot
spec:
  routes:
    - matchers:
        - headers:
            - name: stage
              value: canary
          prefix: /foxtrot
      routeAction:
        single:
          upstream:
            name: foxtrot
            namespace: gloo-system
          subset:
            values:
              version: v2
    - matchers:
        - prefix: /foxtrot
      routeAction:
        single:
          upstream:
            name: foxtrot
            namespace: gloo-system
          subset:
            values:
              version: v1
```

并将其部署到集群:

```
kubectl apply -f rt-foxtrot-2.yaml
```

## 测试路线

现在我们可以测试以确保我们可以向 foxtrot v2 发送请求。我们的其他路线应该继续像以前一样:

```
➜ curl $(glooctl proxy url)/echo
version:echo-v1➜ curl $(glooctl proxy url)/foxtrot
version:foxtrot-v1➜ curl $(glooctl proxy url)/foxtrot -H "stage: canary"
version:foxtrot-v2
```

最后一个请求现在被发送到 v2，因为提供了金丝雀头。

## 狐步舞队开始第二阶段

现在我们可以切换到阶段 2，用加权目的地更新狐步舞路由表:

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: foxtrot-routes
  namespace: foxtrot
spec:
  routes:
    - matchers:
        - headers:
            - name: stage
              value: canary
          prefix: /foxtrot
      routeAction:
        single:
          upstream:
            name: foxtrot
            namespace: gloo-system
          subset:
            values:
              version: v2
    - matchers:
        - prefix: /foxtrot
      routeAction:
        multi:
          destinations:
            - destination:
                upstream:
                  name: foxtrot
                  namespace: gloo-system
                subset:
                  values:
                    version: v1
              weight: 100
            - destination:
                upstream:
                  name: foxtrot
                  namespace: gloo-system
                subset:
                  values:
                    version: v2
              weight: 0
```

并将其部署到集群:

```
kubectl apply -f rt-foxtrot-3.yaml
```

我们预计这些航线将继续像以前一样运行。

## Echo 团队开始 v2 推广

当 foxtrot 团队正在推出 foxtrot 的 v2 时，假设 echo 团队准备开始测试 echo 服务的新版本。

我们可以部署 echo-v2 并等待它被部署:

```
kubectl apply -f echo-v2.yaml
```

等待命名空间“echo”中的窗格准备就绪。使用`kubectl get pods -n echo`检查状态。

然后，我们需要更新路由，以便开始测试 echo v2:

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: echo-routes
  namespace: echo
spec:
  routes:
    - matchers:
        - headers:
            - name: stage
              value: canary
          prefix: /echo
      routeAction:
        single:
          upstream:
            name: echo
            namespace: gloo-system
          subset:
            values:
              version: v2
    - matchers:
        - prefix: /echo
      routeAction:
        single:
          upstream:
            name: echo
            namespace: gloo-system
          subset:
            values:
              version: v1
```

让我们将其部署到集群中:

```
kubectl apply -f rt-echo-2.yaml
```

## 测试路线

现在，我们应该能够测试 echo 或 foxtrot 服务的 v2，因为每个团队都在进行金丝雀部署。

```
➜ curl $(glooctl proxy url)/echo
version:echo-v1➜ curl $(glooctl proxy url)/echo -H "stage: canary"
version:echo-v2➜ curl $(glooctl proxy url)/foxtrot
version:foxtrot-v1➜ curl $(glooctl proxy url)/foxtrot -H "stage: canary"
version:foxtrot-v2
```

# 处理无效配置

正如我们现在看到的，使用路由表允许我们将 canary 部署扩展到多个开发团队，而不需要让团队拥有他们自己的域配置。团队可以并行管理他们自己的 canary 部署，而不需要协调。至少，这是真的，直到有人开始创作无效配置。我们将看到 Gloo 响应无效配置的默认行为对于我们的用例来说是不可取的；但是，我们可以解决这个问题，并通过简单的设置更改来改变行为。

## 存在无效配置时的默认 Gloo 行为

通常，当 Gloo 遇到无效配置时，它会尝试继续向 Envoy 提供最后一次已知的正确配置。这样，错误就不会导致工作路由被删除，并且一旦配置被修复，Envoy 将再次开始接收更新。这并不是万无一失的——只有当 Gloo 和 Envoy 在内存中拥有最后一个已知配置时，它才会起作用，并且不会在 pod 重启后继续存在。但是在出现问题时，它保留了适当的降级语义，并且通常是特定用例的理想行为。

让我们看看在我们的用例中是什么样子的。让我们通过引用一个不存在的上游目的地来模拟 echo 团队编写无效的配置。

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: echo-routes
  namespace: echo
spec:
  routes:
    - matchers:
        - headers:
            - name: stage
              value: canary
          prefix: /echo
      routeAction:
        single:
          upstream:
            name: echo-typo
            namespace: gloo-system
          subset:
            values:
              version: v2
    - matchers:
        - prefix: /echo
      routeAction:
        single:
          upstream:
            name: echo
            namespace: gloo-system
          subset:
            values:
              version: v1
```

让我们将其部署到集群中:

```
kubectl apply -f rt-echo-3.yaml
```

与此同时，foxtrot 团队正在努力完成阶段 2，并调整权重，开始将 100%的流量发送到 v2 目的地:

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: foxtrot-routes
  namespace: foxtrot
spec:
  routes:
    - matchers:
        - headers:
            - name: stage
              value: canary
          prefix: /foxtrot
      routeAction:
        single:
          upstream:
            name: foxtrot
            namespace: gloo-system
          subset:
            values:
              version: v2
    - matchers:
        - prefix: /foxtrot
      routeAction:
        multi:
          destinations:
            - destination:
                upstream:
                  name: foxtrot
                  namespace: gloo-system
                subset:
                  values:
                    version: v1
              weight: 0
            - destination:
                upstream:
                  name: foxtrot
                  namespace: gloo-system
                subset:
                  values:
                    version: v2
              weight: 100
```

让我们将其部署到集群中:

```
kubectl apply -f rt-foxtrot-4.yaml
```

## 测试路线

现在，我们可以通过运行`glooctl check`来判断是否有错误:

```
➜ glooctl check
Checking deployments... OK
Checking pods... OK
Checking upstreams... OK
Checking upstream groups... OK
Checking auth configs... OK
Checking secrets... OK
Checking virtual services... Found virtual service with warnings: gloo-system app
Reason: warning:
  Route Warning: InvalidDestinationWarning. Reason: *v1.Upstream {echo-typo gloo-system} not found
Route Warning: InvalidDestinationWarning. Reason: *v1.Upstream {echo-typo gloo-system} not found
Problems detected!
```

如果我们测试路由，我们将看到以下行为:

```
➜ curl $(glooctl proxy url)/echo
version:echo-v1➜ curl $(glooctl proxy url)/echo -H "stage: canary"
version:echo-v2➜ curl $(glooctl proxy url)/foxtrot
version:foxtrot-v1➜ curl $(glooctl proxy url)/foxtrot -H "stage: canary"
version:foxtrot-v2
```

正如我们所看到的，在整个代理配置中有一个错误，所以 Gloo 继续为 Envoy 提供最后一个已知的良好配置。这意味着狐步舞团队将权重设置为 v2 的更改并不适用。狐步舞队被封锁，等待回声队修复他们的配置。

当考虑一个跨大型开发组织使用的 API 网关时，如果一个团队的错误阻碍了另一个团队的进展，我们会认为这是一个危险信号。幸运的是，Gloo 有一个名为[路由替换](https://docs.solo.io/gloo/latest/guides/traffic_management/configuration_validation/invalid_route_replacement/)的特性，可以改变代理行为并解决这个问题。

## 改变无效配置的行为

让我们创建一个名为`settings-patch.yaml`的文件，其中包含以下 Gloo 设置的补丁:

```
spec:
  gloo:
    invalidConfigPolicy:
      invalidRouteResponseBody: Gloo Gateway has invalid configuration. Administrators
        should run `glooctl check` to find and fix config errors.
      invalidRouteResponseCode: 404
      replaceInvalidRoutes: true
```

这将改变 Gloo 的行为，以便在配置不良的情况下，它用 404 和错误消息替换受错误影响的路由，作为直接响应。在我们的例子中，新的 echo 路由表无效，因此路由被替换；但是，Gloo 将使用 foxtrot 路由表中的路由更新 Envoy，因为该对象是有效的。它也将继续服务于旧的`echo`路线。

我们可以使用以下命令应用补丁:

```
kubectl patch -n gloo-system settings default --type merge --patch "$(cat settings-patch.yaml)"
```

## 测试路线

现在，如果我们运行与上面相同的测试，我们将看到以下结果:

```
➜ curl $(glooctl proxy url)/echo
version:echo-v1➜ curl $(glooctl proxy url)/echo -H "stage: canary"
Gloo Gateway has invalid configuration. Administrators should run `glooctl check` to find and fix config errors.➜ curl $(glooctl proxy url)/foxtrot
version:foxtrot-v2➜ curl $(glooctl proxy url)/foxtrot -H "stage: canary"
version:foxtrot-v2
```

## 完成狐步舞展示

现在我们已经设置了 Gloo 来替换无效路线并保留有效路线，foxtrot 团队可以完成他们的首次展示了。

由于所有流量都已转移到 v2，我们现在可以清理路由:

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: foxtrot-routes
  namespace: foxtrot
spec:
  routes:
    - matchers:
        - prefix: /foxtrot
      routeAction:
        single:
          upstream:
            name: foxtrot
            namespace: gloo-system
          subset:
            values:
              version: v2
```

让我们使用以下命令来部署它:

```
kubectl apply -f rt-foxtrot-5.yaml
```

我们可以删除`v1` foxtrot 部署，它不再为任何流量提供服务。

```
kubectl delete -f foxtrot-v1.yaml
```

最后，让我们检查一下，以确保狐步舞仍然健康:

```
➜ curl $(glooctl proxy url)/foxtrot
version:foxtrot-v2
```

## 修复回显配置

我们还可以恢复无效的 echo 配置，以解除对 echo 团队的封锁。我们将恢复到旧的路由表:

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: echo-routes
  namespace: echo
spec:
  routes:
    - matchers:
        - headers:
            - name: stage
              value: canary
          prefix: /echo
      routeAction:
        single:
          upstream:
            name: echo
            namespace: gloo-system
          subset:
            values:
              version: v2
    - matchers:
        - prefix: /echo
      routeAction:
        single:
          upstream:
            name: echo
            namespace: gloo-system
          subset:
            values:
              version: v1
```

让我们将其部署到集群中:

```
kubectl apply -f rt-echo-2.yaml
```

现在，我们将看到通过运行`glooctl check`来清除错误:

```
➜ glooctl check
Checking deployments... OK
Checking pods... OK
Checking upstreams... OK
Checking upstream groups... OK
Checking auth configs... OK
Checking secrets... OK
Checking virtual services... OK
Checking gateways... OK
Checking proxies... OK
No problems detected.
```

我们将看到 echo 的路由再次按预期工作:

```
➜ curl $(glooctl proxy url)/echo
version:echo-v1➜ curl $(glooctl proxy url)/echo -H "stage: canary"
version:echo-v2
```

# 将这种方法推广到真正的自助团队

到目前为止，我们的解决方案仍然要求拥有该域的运营团队在每次向我们的应用程序添加新服务时，向虚拟服务添加新路由。我们可以通过使用标签选择器将路由表与虚拟服务相关联来消除这种依赖性。

让我们用`apiGroup: example`标记每个路由表:

```
apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: echo-routes
  namespace: echo
  labels:
    apiGroup: example
spec:
  routes:
    - matchers:
        - headers:
            - name: stage
              value: canary
          prefix: /echo
      routeAction:
        single:
          upstream:
            name: echo
            namespace: gloo-system
          subset:
            values:
              version: v2
    - matchers:
        - prefix: /echo
      routeAction:
        single:
          upstream:
            name: echo
            namespace: gloo-system
          subset:
            values:
              version: v1apiVersion: gateway.solo.io/v1
kind: RouteTable
metadata:
  name: foxtrot-routes
  namespace: foxtrot
  labels:
    apiGroup: example
spec:
  routes:
    - matchers:
        - prefix: /foxtrot
      routeAction:
        single:
          upstream:
            name: foxtrot
            namespace: gloo-system
          subset:
            values:
              version: v2
```

我们可以使用以下命令将这些应用于我们的集群:

```
kubectl apply -f rt-echo-4.yaml
kubectl apply -f rt-foxtrot-6.yaml
```

现在，我们可以更新我们的虚拟服务，在它的选择器中有一个使用这个标签`apiGroup: example`的单一路由，这样服务在创建时就立即被绑定到域。

```
apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: app
  namespace: gloo-system
spec:
  virtualHost:
    domains:
      - '*'
    routes:
      - matchers:
          - prefix: /
        delegateAction:
          selector:
            labels:
              apiGroup: example
            namespaces:
              - "*"
```

我们可以使用以下命令将其应用于集群:

```
kubectl apply -f vs-2.yaml
```

现在，我们的路由完全像以前一样工作，但是当新名称空间中的新服务上线时，我们不再需要更新虚拟服务。

这引入了一个潜在的风险，即不同的团队可能试图定义相同的路线，尽管这可能是一个可接受的风险，以支持完全自助服务的团队，并且我们可能能够找到其他方法来实施可伸缩的约定。

我们已经看到我们的用户通过这种方法帮助自助服务开发团队的一种方式是将 Gloo 路线以及部署、服务、上游和其他配置打包到一个 helm 图表中。团队可以用适合他们服务的值来定制舵图，从而为约定提供更强的防护。

通过将更多验证集成到 CI/CD 管道中，或者使用 webhook(如开放策略代理服务器)阻止无效配置写入集群，可以增加对无效路由的更严格保护。

# 摘要

在这篇文章中，我们着眼于扩展我们的两阶段金丝雀展示工作流程。我们通过将不同的路由表委派给不同的团队，在多个开发团队之间进行了扩展。通过委托，我们实现了拥有域的开发运营团队和拥有不同路线的开发团队的更清晰的责任分离。最后，很容易在 Gloo 中定制行为，以确保两个团队可以并行操作，而不需要协调，并且没有一个团队通过编写无效配置来阻止另一个团队的风险。

特别感谢 Ievgenii Shepeliuk 对[第 1 部分](https://www.solo.io/blog/two-phased-canary-rollout-with-gloo/)提供反馈，并分享路由表在他的组织中是如何使用的。要深入了解 Gloo 在新时代解决方案中的应用，请查看我们的[案例研究](https://www.solo.io/blog/end-user-case-study-hrzn-igaming-platform-by-new-age-solutions/)。

# 加入 Gloo 社区

除了企业客户群之外，Gloo 还有一个庞大且不断增长的开源用户社区。要了解更多关于 Gloo 的信息:

*   查看[回购](https://github.com/solo-io/gloo)，在这里你可以看到代码和文件问题
*   查看[文档](https://docs.solo.io/gloo/latest)，其中有大量的指南和示例
*   加入 slack 频道并开始与 Solo 工程团队和用户社区聊天

如果你想与我联系(反馈总是被感激！)，你可以在 Slack 上找到我或者发邮件给我 [rick.ducott@solo.io](mailto:rick.ducott@solo.io) 。