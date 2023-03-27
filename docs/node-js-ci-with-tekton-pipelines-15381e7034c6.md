# 带有 Tekton 管道的 Node.js CI

> 原文：<https://itnext.io/node-js-ci-with-tekton-pipelines-15381e7034c6?source=collection_archive---------2----------------------->

![](img/81a37f58f8ba8c085180af58849a2d7a.png)

Andrea Jaime 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

今天，Kubernetes 是一个了不起的管弦乐队，而 Tekton 是最新的本地 CI/CD 集群解决方案之一，而且，依我看，这种组合是惊人的。

在我的职业生涯中，多年来我一直在逐步接受 DevOps 实践，使用不同的工具和应用不同的方法。

然后，我突然检查了 Tekton Pipelines，嘣，它解决了我过去遇到的所有问题(我可以说出几十个)，从管道中的重复到任务的依赖性。

这几行代码并不关注 Tekton 是什么，而是关注如何使用它来实现与 Node.js 代码库的持续集成(当然，它可以扩展到您使用的任何堆栈)。

我们的案例是一个简单的 Nuxt 应用程序，它需要为我们的 CI 执行以下步骤:

*   依赖项安装
*   潜在漏洞的审计
*   林挺
*   单元测试
*   建筑物

首先，让我们设置指向 Git 存储库的 Tekton 资源，在本例中是[https://github.com/cdbkr/nuxt](https://github.com/cdbkr/nuxt)。

资源. yaml

在上面的文件中，我们声明了将在任务/管道中使用的资源(git repo)。

接下来，我们在 Tekton 任务中使用这个资源:

Task.yaml

让我解释一下 Tekton 任务规范的结构。

*   在`Inputs`中，我们用声明的 Git repo 指定连接；
*   在`Steps`中，我们指定了一组需要执行的连续步骤；

现在，让我们来看一下与常见 Node.js 应用程序紧密相关的步骤:

*   `npm install`安装依赖项；
*   `npm audit`用已安装的依赖项检查漏洞；
*   `npm run lint`运行皮棉检查；
*   `npm run test`运行单元测试；
*   `npm run build`构建我们的应用程序包；

当这些步骤中的一个将失败时，执行将被终止(例如，如果 NPM 在已安装的依赖项中发现了一些漏洞，我们不想继续进行)。

之后，我们只需要执行任务，为此，我们有几个选项:

*   它允许我们执行单个任务；
*   `PipelineRun`:它允许我们执行一组定义好的任务，并在`Pipeline`资源定义中指定；

因为这个例子非常简单，所以我们将使用 TaskRun，它触发一个包含容器/步骤的 Pod 来执行。

一旦执行并正确验证，我们就可以检查用

```
+ kubectl get pods
NAME                        READY   STATUS            RESTARTS   AGE
nodejs-taskrun-pod-67caed   6/6     Running           0          11s
```

完成后:

```
+ kubectl get pods -w
NAME                        READY   STATUS    RESTARTS   AGE
nodejs-taskrun-pod-67caed   6/6     Running   0          11s
nodejs-taskrun-pod-67caed   5/6     Running   0          52s
nodejs-taskrun-pod-67caed   4/6     Running   0          53s
nodejs-taskrun-pod-67caed   3/6     Running   0          55s
nodejs-taskrun-pod-67caed   2/6     Running   0          58s
nodejs-taskrun-pod-67caed   1/6     Running   0          64s
nodejs-taskrun-pod-67caed   0/6     Completed   0          81s----+ kubectl get taskruns
NAME             SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
nodejs-taskrun   True        Succeeded   2m4s        43s
```

不错！因此，每个步骤都作为一个单独的容器执行，TaskRun 资源用执行的状态标记，在本例中为 Success:)。

这是一个非常简单的例子，说明了如何用一个公共 Node.js 项目运行 CI。通过这种方式，你可以想象有多少事情可以实现:

*   跨运行重用声明的任务和管道
*   隔离步骤/容器之间的依赖关系(是的，可以为每个步骤使用不同的图像)。例如，在特定环境下运行 E2E 测试(在 Docker 映像中进行沙箱化)，在依赖于不同技术堆栈的给定库上进行安全测试，执行需要特殊映像的脚本等…

我希望这篇文章能让您了解如何使用 Tekton Pipelines 完成这项常见任务，在下一篇文章中，我将展示如何在这项工作的基础上实现连续部署。如果您感兴趣并想了解更多，请查看以下两个资源:

[](https://github.com/tektoncd/catalog) [## tektoncd/目录

### 这个存储库包含一个任务资源(将来还有管道和资源)的目录，它被设计成…

github.com](https://github.com/tektoncd/catalog) [](https://github.com/jenkins-x) [## 詹金斯 X

### Kubernetes - Jenkins X 上现代云应用的 CI/CD 解决方案

github.com](https://github.com/jenkins-x) 

干杯！:)