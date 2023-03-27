# Kubernetes 世界的微服务应用之旅

> 原文：<https://itnext.io/journey-of-a-microservice-application-in-the-kubernetes-world-bdfe795532ef?source=collection_archive---------3----------------------->

## webhooks.app 演示

![](img/59ce769d4a75d32af5a8e197d0601487.png)

[马特·霍华德](https://unsplash.com/@thematthoward?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/journey?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍照

## TL；速度三角形定位法(dead reckoning)

[webhooks.app](https://webhooks.app) 是遵循微服务架构的开源应用。它的目的是为演示提供一个 webhook 端点。在这一系列文章中，我将介绍这个应用程序以及我使用(并将使用)的几个步骤，让它在 Kubernetes 集群中运行。

## 关于这个系列

本系列将分为以下几篇文章，详细介绍 webhooks 应用程序的用途:它的架构，如何在本地和 Kubernetes 集群上部署它，以及它所依赖的各种项目。其他文章可能会添加到写作时想到的主题中。

*   webhooks.app 的演示(当前文章)
*   [使用 Helm](/journey-of-a-microservice-application-in-the-kubernetes-world-3c2a9e701e9f) 在 Kubernetes 上运行应用程序
*   [在 Civo Kubernetes 集群上运行应用](/journey-of-a-microservice-application-in-the-kubernetes-world-e800579f0be3)
*   [使用 GitOps 和 ArgoCD 进行连续部署](/journey-of-a-microservice-application-in-the-kubernetes-world-d9493b19edff)
*   [使用 Loki 堆栈的可观察性](/journey-of-a-microservice-application-in-the-kubernetes-world-876f72ce1681)
*   [使用 Acorn 定义应用](/journey-of-a-microservice-application-in-the-kubernetes-world-e2f6475ddde1)
*   [安全注意事项:安全相关工具](/journey-of-a-microservice-application-in-the-kubernetes-world-6abd625c60fe)
*   [安全考虑:修复错误配置](/journey-of-a-microservice-application-in-the-kubernetes-world-eb0fb52e1bf0)
*   [安全考虑:政策执行](/journey-of-a-microservice-application-in-the-kubernetes-world-f760cba7600f)
*   安全考虑:漏洞扫描(即将推出)

## 关于申请

[webhooks.app](https://webhooks.app) 是一个应用程序，其目的是提供始终在线的安全 webhook 端点(基本上是一个等待从 HTTP POST 请求接收 json 有效负载的服务器)。Webhook 是应用程序相互通信的一种方式，通常以 json 有效载荷的形式交换信息。

让我们考虑下面的例子:我们有一个[港](https://goharbor.io)的容器注册中心，并希望在每次有新图像被推送到给定项目时得到通知。Harbor 允许指定一个 webhook (url 和认证令牌),并在每次推送新图像时向该 webhook 发送一个 json 有效负载。

![](img/f7e7ecf1babf2429b31982b8a5002593.png)![](img/704170f1a7a0b9af503a231aea73aa80.png)

指定 webhook，这样 Harbor 将在每次新图像被推送到 api 项目时向该 webhook 发送一个有效负载

注意事项:

*   许多其他注册中心允许使用 webhooks 与外部系统通信(您也可以在 Docker Hub 仓库中使用它)
*   Harbor 还可以为其他类型的事件(图像扫描的结果、超出配额等)向 webhook 端点发送有效负载

为了说明这一点，我们可以标记一个虚拟图像，并将其发送到港口登记处:

```
$ docker login -u admin harbor.techwhale.io
$ docker image pull nginx:1.20
$ docker image tag nginx:1.20 harbor.techwhale.io/api/pong:2.0
$ docker image push harbor.techwhale.io/api/pong:2.0
```

我们立刻在 webhooks.app 的仪表盘上看到 Harbor 发送的有效载荷内容:

![](img/0b182acd76fc61315b96b7cb59b16dae.png)

webhook 端点接收的 json 负载的内容

webhooks.app 只允许您查看传入负载内部的内容，而不会以任何方式处理负载。这对于演示非常有用，因此我们可以检查有效负载的内容，这是开发实际处理该有效负载的服务器之前的第一步。

如果你想了解更多关于 webhooks 的概念，我推荐[这篇很棒的文章](https://www.getvero.com/resources/webhooks/)。

## 整体架构

该应用托管在 GitLab 的 [web-hook 组](https://gitlab.com/web-hook)中。每个微服务都有自己的存储库:

*   web-hook :包含应用程序细节的元存储库，比如在本地运行它的方式，以及如果你可以贡献的话的说明
*   **www** :嗯……前端 web
*   **api** :处理数据并与底层数据库通信的微服务
*   **ws** : websocket 服务器，确保每次应用程序接收到新的有效负载时，前端都会实时更新

除了这些微服务之外，还使用了其他组件:

*   NATS 消息代理
*   MongoDB 数据库
*   应用程序前面的 Traefik 反向代理

下面的模式说明了应用程序的体系结构:

![](img/3a977a7932d80f216608f0c00891c92f.png)

应用程序的架构

基本上:

*   Traefik 向外部世界公开 api 和 web 前端
*   每次有效载荷被发送到 api，它被保存在数据库中，并在 NATS 的一个专门的主题上发布
*   websocket 服务器接收来自 NATS 的消息，并通过 websocket 将更新发送给浏览器

注意:体系结构中使用 NATS 来分离 api 和 ws 组件。它允许建立一个发布/订阅系统， **api** 是 NATS 的发布者，而 **ws** 是 NATS 的订阅者:api 发送的消息由 ws 接收。如果需要，以后可以很容易地添加额外的订户。

## 每个微服务的简单 CI 设置

在 **api** 、 **www** 和 **ws** 中的每个库都包含一个. gitlab-ci.yml 文件，该文件定义了在 git 推送时触发的动作。例如，文件[https://gitlab.com/web-hook/api/-/blob/main/.gitlab-ci.yml](https://gitlab.com/web-hook/api/-/blob/main/.gitlab-ci.yml)定义了 **api** 的 CI(持续集成)管道。当新代码被推送到 api 存储库的主分支时，会发生以下动作:

*   创建了一个新标签(遵循 SemVer 格式)
*   使用这个标签构建并标记一个新的容器图像
*   用这个标签更新配置存储库(在下一篇专门讨论 GitOps 方法的文章中有更多相关内容)

因此，CI 的主要结果是创建一个新的映像，这个映像托管在微服务存储库的容器注册表中。

![](img/d2f4db6d5ccb23fdd4d8c655a9df04ab.png)

已经为 api 微服务构建的容器映像的历史

## 用 Docker Compose 在本地运行它

在 [web-hook 库](https://gitlab.com/web-hook/web-hook)中，你可以找到在本地运行整个项目的所有指令。下面详细介绍了如何使用 Docker Compose 在容器中运行应用程序。

首先在新文件夹中克隆存储库

```
mkdir webhooks && cd webhooks
for repo in www api ws web-hook; do
  git clone git@gitlab.com:web-hook/$repo.git
done
```

接下来，使用 web-hook 存储库中的 docker-compose.yml 文件运行应用程序:

```
cd web-hook
docker compose up
```

几秒钟后，您可以访问应用程序，将您的 web 浏览器指向本地主机，并单击底部的 **Get my webhook** 按钮获得您自己的 webhook 端点。

![](img/28504585110ffa7451e2a00e1f543050.png)![](img/dbff553cbba5c87b6cc78bd4a0e1cdc3.png)![](img/17ed6dcd700cc4928dbdda97f9d32f5c.png)

访问应用程序并获得自己的 webhook 端点(url 和身份验证令牌)

让我们跳到仪表板上:

![](img/e8068109013656f527370a0c1d1c794e.png)

在那里，您有一个 curl sample 命令，可以用来从命令行直接发送有效负载:

```
$ curl -XPOST -H "Authorization: Bearer 7227a2c2ebdbc267438e7ece2bdd0a" [http://localhost/data](http://localhost/data) -d '{"ok": "hello"}' -H 'Content-Type: application/json'
```

仪表板将实时更新有效负载的内容:

![](img/63a20b6df611a1d4f3fab16af08ad0ed.png)

仪表板中收到的有效负载

在下一篇文章中，我们将在本地 Kubernetes 集群上部署 webhooks 应用程序。