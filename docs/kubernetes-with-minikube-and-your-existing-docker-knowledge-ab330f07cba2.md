# Kubernetes 与 Minikube 和你现有的码头知识

> 原文：<https://itnext.io/kubernetes-with-minikube-and-your-existing-docker-knowledge-ab330f07cba2?source=collection_archive---------4----------------------->

![](img/0c90ab5266c88f507ed493b4d4cd418b.png)

让我们假设你已经开发了你的应用程序。这是一个很棒的应用程序，你对输出很满意，现在你需要部署。简单的解决方案。我们去码头吧。为了简单起见，我将展示我在科罗纳疫情期间(当然是在远程办公室工作后的空闲时间)学习 node.js 和 express 的过程中构建的一个应用程序。在此处找到源代码:

```
[https://github.com/humayun-rashid/covid-19-node-app](https://github.com/humayun-rashid/covid-19-node-app)
```

假设这是你的应用程序，你想把它部署到 docker 注册中心。事实上，如果你知道一些关于 docker 的基础知识，这是非常简单的。步骤是:

*   克隆存储库
*   创建 Dockerfile 文件
*   构建和标记图像
*   推送图像

## 克隆

```
git clone [https://github.com/humayun-rashid/covid-19-node-app](https://github.com/humayun-rashid/covid-19-node-app)
```

## 构建和标记

在 Dockerfile 中添加配置，并将其放在同一个 app 目录中。如果您正在使用我提供的 git 存储库链接，它已经在那里了。

```
# Base Image
FROM node:slim# Create app directory
WORKDIR /usr/src/app # Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install# Bundle app source
COPY . . EXPOSE 3000CMD [ "node", "server.js" ]docker build -t covid-19:latest .
```

docker 标签(在斜杠前使用您自己的存储库名称)

```
docker tag covid-19:latest raahat/covid-19:latest
```

推

```
docker push raahat/covid-19:latest
```

它会询问 docker 用户名和密码。完成了！现在运行它:

```
docker run --rm --name covid-19-app -dit -p 3000:3000 raahat/covid-19:latest
```

打开浏览器并访问本地主机，验证应用程序是否已启动并正在运行。

```
http://localhost:3000
```

## 样本项目:新冠肺炎

对于我们的项目，我将使用我以前的一个项目作为例子。您可以从这里下载:

```
[https://github.com/humayun-rashid/covid-19-node-app](https://github.com/humayun-rashid/covid-19-node-app)
```

您可以构建映像并运行，也可以直接从 docker 注册表运行映像。直接从 docker 注册表运行映像:

```
docker run -it --rm --name covid-19 -p 3000:3000 raahat/covid-19:latest
```

成功执行后，应该会有以下输出:

```
Server is running in 3000
```

如果您想要构建并运行，那么请转到目录并运行-

```
docker build -t covid-19:latest .
docker run -it --rm --name covid-19 -p 3000:3000 covid-19:latest
```

## Kubernetes 部署的步骤

*   创建名称空间
*   创建部署
*   规模部署
*   创建服务以公开部署
*   配置入口控制器
*   创建入口控制器

## 创建名称空间

可以使用 Kubectl CLI 创建名称空间。首先，通过以下命令检查可用的名称空间:

```
$ kubectl get namespace
or 
$ kubectl get ns
```

输出如下所示:

```
Output
```

让我们使用 Kubectl CLI 创建一个新的名称空间。我们将把它命名为“发展”。

```
$ kubectl create namespace development
```

它将创建一个新的名称空间。我们将在该名称空间下部署所有内容。如果所有部署都在任何特定的名称空间下，那么处理任何事情都会容易得多。

现在，我们已经使用 kubectl CLI 创建了一个新的名称空间，但这不是标准的生产过程。我们需要使用 YAML 编写配置文件。运行以下命令，该命令将为当前目录中的命名空间生成 YAML 配置。

```
$ kubectl create namespace development --dry-run -o yaml > namespace.yaml
```

现在，您可以使用以下命令创建名称空间:

```
$ kubectl apply -f namespace.yaml 
```

## 创建 Pod

一个 Pod 可以被视为 Kubernetes 中一个应用程序的单个实例，代表一个部署单元。根据 Kubernetes 官方网站:

> 一个 *Pod* 是 Kubernetes 应用程序的基本执行单元——是您创建或部署的 Kubernetes 对象模型中最小和最简单的单元。一个 Pod 代表在您的[集群](https://kubernetes.io/docs/reference/glossary/?all=true#term-cluster)上运行的进程。

要考虑的主要重要因素-

*   pod 能够运行应用程序的容器。
*   一个 pod 可以由一个容器组成，也可以由多个容器组成，这些容器相互绑定以共享资源。

使用 CLI 和 YAML 创建 Pod

```
$ kubectl run --image=raahat/covid-19 covid-19-app --port=3000 --restart=Never --namespace=development
```

让我们试着拿到豆荚:

```
kubectl get pods -n developmentNAME           READY   STATUS    RESTARTS   AGE
covid-19-app   1/1     Running   0          31mkubectl delete pods covid-19-app -n monitoringkubectl port-forward covid-19-app 3000:3000 -n developmentkubectl get pods -n development
No resources found in development namespace.kubectl run --image=raahat/covid-19 covid-19-app --port=3000 --restart=Never --namespace=development --dry-run -o yaml > pod.yamlkubectl create -f pod.yaml
```

# 使用 CLI 和 YAML 创建部署

Kubernetes [豆荚](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)是凡人。他们出生，当他们死去，他们不会复活。如果您使用[部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)来运行您的应用程序，它可以动态地创建和销毁 pod。

```
kubectl create deployment covid-19 --image=raahat/covid-19 --namespace=development --dry-run=client -o yaml > deployment.yaml
```

打开 deployment.yaml 文件，在容器部分中添加端口和容器端口。它应该是这样的:

```
containers:
- image: raahat/covid-19
  name: covid-19
  ports:
  - containerPort: 3000
```

现在通过以下方式应用它:

```
kubectl apply -f deployment.yaml
```

每个 Pod 都有自己的 IP 地址，但是，在部署中，某个时刻运行的一组 Pod 可能与稍后运行该应用程序的一组 Pod 不同。

这就导致了一个问题:如果某组 Pods(称之为“后端”)向集群中的其他 Pods(称之为“前端”)提供功能，前端如何发现并跟踪要连接到哪个 IP 地址，以便前端可以使用工作负载的后端部分？

对于应用程序的某些部分(例如，前端)，您可能希望将服务公开到一个外部 IP 地址，即您的集群之外。

Kubernetes `ServiceTypes`允许你指定你想要什么样的服务。默认为`ClusterIP`。

`Type`价值观及其行为是:

*   `ClusterIP`:公开集群内部 IP 上的服务。选择该值将使服务只能从集群内部访问。这是默认的`ServiceType`。
*   `[NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport)`:在一个静态端口暴露每个节点的 IP 上的服务(`NodePort`)。一个`ClusterIP`服务，自动为其创建`NodePort`服务路由。您将能够通过请求`<NodeIP>:<NodePort>`从集群外部联系`NodePort`服务。
*   `[LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)`:使用云提供商的负载均衡器对外公开服务。外部负载平衡器路由到的`NodePort`和`ClusterIP`服务是自动创建的。

# 公开部署

```
kubectl expose deployment covid-19 — type=NodePort --port=3000 --namespace=development --dry-run=client -o yaml > service.yaml
kubectl apply -f service.yaml 
```

验证服务已创建并且在节点端口上可用:

```
kubectl get service -n development
NAME       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
covid-19   NodePort   10.105.76.139   <none>        3000:31450/TCP   5s
```

通过节点端口访问服务:

```
minikube service covid-19 --url -n development
[http://192.168.99.224:31450](http://192.168.99.224:31450)
```

如果将`type`字段设置为`NodePort`，Kubernetes 控制平面将从`--service-node-port-range`标志指定的范围内分配一个端口(默认值:30000-32767)。