# 迷你视频编码器第 1 部分，引擎

> 原文：<https://itnext.io/mini-video-encoder-implementation-part-1-1fce5fb081c1?source=collection_archive---------7----------------------->

![](img/351ac3482337013494f211cae22a2524.png)

在本文中，我使用了 **Docker、** **Node.js、 **Fastify、MongoDB、**和**mongose**来实现迷你视频编码器的工作流引擎和工作流数据库。工作流引擎是迷你视频编码器(MVE)的心脏；它负责管理所有与视频编码相关的任务。MVE 是我今年开始的新的副业项目，让我探索新的令人兴奋的技术。如果你对我为什么开始这个项目及其细节感兴趣，我邀请你阅读这篇文章:**

[](https://medium.com/@pkalkman/new-year-new-side-project-e69991252bf4) [## 新的一年，新的副业？

### 我喜欢做副业，你呢？他们让我尝试新技术，研究新框架…

medium.com](https://medium.com/@pkalkman/new-year-new-side-project-e69991252bf4) 

# 迷你视频编码器平台

本文重点研究了小型视频编码器平台的工作流引擎和工作流数据库。

![](img/9a17179ed5aa892529116710d447abbd.png)

小型视频编码器架构图

## 源代码结构

迷你视频编码器平台的源代码可以在 [Github](https://github.com/PatrickKalkman/MiniVideoEncoder) 上找到。该项目包含六个文件夹，每个文件夹包含该特定服务的源代码。

迷你视频编码器平台的文件夹结构

每个文件夹都包含一个 Docker 文件，用于构建特定服务的 Docker 映像。根目录中的 docker-compose.yml 文件用于启动所有 docker 容器。

# Docker 容器中的 MongoDB

工作流引擎将编码平台的状态存储在 MongoDB 数据库中。MongoDB 运行在一个 **Docker** 容器中，数据库本身安装在主机的文件系统上。关于工作流数据库的所有信息可以在[工作流数据库文件夹](https://github.com/PatrickKalkman/MiniVideoEncoder/tree/master/WorkflowDatabase)中找到。

## 创建数据库用户

在开发过程中，我使用 Docker Compose 来启动 MVE 平台的所有服务。Docker Compose 使安装和启动 MVE 变得容易。如果工作流数据库和用户不存在，则在容器启动时创建。

我用的是最新的官方 [MongoDB Docker 镜像](https://hub.docker.com/_/mongo?tab=tags)，这个时候是`Mongo:4.0.14`。MongoDB Docker 映像支持在启动时通过环境变量创建根用户。

用于启动 MongoDB 的 docker-compose.yml

环境变量`MONGO_INITDB_USERNAME`和`MONGO_INITDB_ROOT_PASSWORD`创建一个新的用户和密码。Dockerfile 在`admin`认证数据库中创建这个用户，该用户具有 root 角色，即超级用户角色。`MONGO_INITDB_DATABASE`环境变量指定了在`/docker-entrypoint-initdb.d/`文件夹中执行 javascript 创建脚本时使用的数据库名称。

我使用 javascript 文件`init-mongo.js`来创建工作流引擎用来连接数据库的用户。Docker Compose 使用 Docker 容器中的卷来映射`init-mongo.js`文件。MongoDB 在启动时执行脚本并创建`mve-workflowengine`用户。

在数据库启动期间创建数据库用户

## 验证创建的用户

用`docker-compose up`启动 MongoDB 服务器后。用户`mveroot`和`mve-workflowengine`都被创建。`mveroot`用户拥有对 MongoDB 服务器所有数据库的超级用户访问权限，而`mve-workflowengine`用户拥有对`workflow-db`数据库的读/写访问权限。

您可以通过使用 MongoDB GUI 来验证是否可以访问数据库。我用的是 [MongoDB Compass](https://www.mongodb.com/download-center/compass) 的社区版。您可以使用下面的连接字符串通过`mve-workflowengine`用户连接到`workflow-db`数据库。

```
mongodb://mve-workflowengine:mve-workflowengine-password@localhost:27017/?authSource=workflow-db&readPreference=primary&authMechanism=DEFAULT&appname=MongoDB%20Compass%20Community&ssl=false
```

# 工作流引擎 REST API

![](img/638d96d48f421b185d73a1bd90bbef91.png)

工作流引擎是一个 Node.js 应用程序，它使用 [Fastify](https://www.fastify.io/) 提供 REST API。Fastify 是一个用于 Node.js 的快速、低开销的 web 框架。MVE 平台的服务使用其 REST API 连接到工作流引擎。

我使用 Fastify 插件[**Fastify-register-routes**](https://github.com/israeleriston/fastify-register-routes)来注册路线。Fastify-register-routes 让我在单独的路由文件中定义路由，并用一条语句读取它们。

自动注册 route 文件夹中的所有*Routes.js 文件

在`routes`文件夹中包含路线的每个文件都被自动注册。这有助于我保持项目的组织性和可维护性。见下面的文件`jobRoutes.js`，它定义了所有与工作相关的路线。

jobRoute.js 定义了所有与工作相关的路线

在`jobRoutes.js`中，我定义了一个包含`api/job`端点上所有路由的数组。在单条路线的下面，带有解释路线的每个字段的注释。

带解释的单条路线

在`jobController.js,`中，我实现了`getJobs()`方法，该方法包含从数据库中检索所有工作流作业的逻辑。但是在我解释`jobController`中的`getJobs()`之前，我们必须深入了解一下我们将如何与 MongoDB 数据库进行交互。

# Mongoose vs mongodb 驱动程序

当你想从 Node.js 与 MongoDB 交互时，你有两个选择:使用 **mongodb** 驱动或者使用**mongose**。当然，选择取决于您的用例；Mongoose 构建在 mongodb 驱动程序之上，帮助您对数据建模，从而编写更少的代码。另一方面，mongodb 驱动程序更容易上手，并且提供了最好的[性能](https://medium.com/@bugwheels94/performance-difference-in-mongoose-vs-mongodb-60be831c69ad)。

对于 MVE，我使用 Mongoose，因为此时数据库性能不是问题，我发现验证很重要。

## 猫鼬

从 Mongoose 开始，使用`Schema()`命令描述模型的模式。

job.js 定义了作业文档的模式

在模式中，定义文档的属性。对于每个属性，您可以定义类型以及它是否是必需的。这使得 Mongoose 能够在插入新文档时执行验证。`{ timestamps: true }`是 Mongoose 的一个选项，它自动向文档添加和填充`createdAt`和`updateAt`属性。我创建了两个索引，一个在 status 字段，另一个在 name 字段。

# 在控制器中结合 Fastify 和 Mongoose

定义了路由和模式后，我们现在可以创建`JobController`。`JobController`由`JobRoutes`引用，并使用作业模式执行数据库操作。

## 添加新作业

`JobController`上的`addJob()`方法负责在数据库中存储一个新任务。

addJob()在数据库中添加一个新作业

在第二行，我直接从请求体创建了一个作业。我不验证传入的请求，因为 Mongoose 在`save()`期间执行验证。如果缺少一个必需的字段，API 将返回一个 BadRequest，指出缺少哪个字段。

缺少必填字段时 API 的响应

`JobController`的`getJob()`方法用猫鼬的`findById()`方法找工作。通过 id 搜索是可能的，因为 MongoDB 为每个插入的文档添加了一个名为`_id`的惟一 id 字段。

getJob()从数据库中返回一个特定的作业

我总是试图通过返回正确的 HTTP 响应代码来处理错误。如果`findById()`没有找到工作，我通过调用第八行的`reply.notFound()`返回一个 HTTP 404 状态代码。

## 更新作业

`JobController`的`updateJob()`方法使用 Mongoose 的`findByIdAndUpdate()`在一条语句中搜索、更新并返回更新后的作业。

updateJob()更新数据库中的作业文档

默认情况下，`findByIdAndUpdate()`返回更新前的文档。通过发送选项`{ new: true }`，它返回更新的作业。在第四行，我使用[扩展操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)来创建对象的副本。

如果您对其余两个方法`deleteJob()`和`getJobs()`的实现感兴趣，请查看 [Github](https://github.com/PatrickKalkman/MiniVideoEncoder/blob/master/WorkflowEngine/lib/controllers/jobController.js) 上的`jobController.js`。

# 将工作流引擎和工作流数据库归档

在继续实现工作流引擎和工作流数据库之前，我想确保它们都运行在 Docker 容器中，并且可以相互通信。这意味着用工作流引擎扩展`docker-compose.yml` 文件，并为工作流引擎项目添加 Dockerfile。

## 工作流引擎的 Dockerfile

[使用我在生产](https://medium.com/better-programming/docker-for-node-js-in-production-b9dc0e9e48e0)中运行 Node.js 应用程序的清单，我为工作流引擎创建了以下 docker 文件。

有了这个 docker 文件，你可以用`docker build -t workflowengine:0.6.0 .`构建 Docker 映像。我们现在有两个 Docker 映像，可以使用 Docker Compose 启动。

## 扩展 docker-compose 文件

我已经展示了工作流数据库的一个`docker-compose.yml`文件。我添加了工作流引擎，这样两者都可以使用 Docker Compose 启动。

docker-编写用于启动工作流数据库和工作流引擎的文件

Docker Compose 创建了两个服务，第 3 行的`workflowdb`和第 17 行的`workflowengine`。我用正确的连接字符串设置了环境变量`STORAGE_HOST`，以便工作流引擎能够连接到数据库。我用服务名`workflowdb`来表示主机。这样我就不必在 Docker 容器中找到 MongoDB 服务器的 IP 地址。

`- STORAGE_HOST=mongodb://mve-workflowengine:mve-workflowengine-password@**workflowdb**:27017/workflow-db?authMechanism=DEFAULT&authSource=workflow-db`

两个服务都连接到网络`mve-network`以确保它们可以相互通信。

*当 Fastify 服务器在 Docker 容器中运行时，我在从主机连接到它时遇到了一些问题。似乎默认 Fastify 监听 127.0.0.1 地址，这在 Docker 容器中不起作用。在将其更改为****0 . 0 . 0 . 0****后，我可以从主机访问它。*

# 下一步做什么？

好吧，我们现在在哪里？我们在 Docker 中运行了工作流引擎和工作流数据库，它们能够相互通信。我们可以使用 REST API 插入、更新或删除视频编码作业。这两个容器都可以使用 Docker Compose 启动。

接下来，我将使用管理编码任务的附加端点来扩展工作流引擎，添加身份验证，并开始处理视频编码器。