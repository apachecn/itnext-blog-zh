# 将 SpringBoot 应用程序归档

> 原文：<https://itnext.io/dockerize-a-springboot-app-1755e49fe3d9?source=collection_archive---------1----------------------->

## 让我们从 docker 图像和容器开始，让我们展示如何对 SpringBoot 应用程序进行 Docker 化并上传到 Docker 注册表

在这个故事中，我们将探索如何简单地将一个 SpringBoot 应用程序 dockerize 并运行到一个容器中。然后，我们将看到如何在我们自己的 docker hub 中的公共 docker 注册表中推送它，以便稍后在 k8s pod 中使用它。

![](img/997afd0cd86c5908152e54831b234b78.png)

托马斯·李普克在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

首先，需要注册 docker hub。这使我们能够创建公共或私有存储库来推送我们的 docker 映像。因此，让我们访问 [Docker hub 官方网站](https://hub.docker.com)并注册。请记住用户名(docker id)和密码，以便进行后续步骤。

之后，我们可以从[这里](https://docs.docker.com/docker-for-mac/install/)开始为我们的平台系统分别下载 Docker 就我而言，我会下载 mac 发行版。让我们安装下载的软件包。它将为我们提供能够与 Docker 生态系统交互的完整命令。

我们可以使用下面的命令

```
docker -v
```

测试安装是否成功。我正在使用的当前版本是

```
Docker version 19.03.5, build 633a0ea
```

听起来很棒！现在，我们已经准备好并能够创建我们的第一个 Docker 图像。

在这个故事中，我不会给你任何 SpringBoot 应用程序。你可以决定从任何你想要的应用程序中创建一个图像。

所以，一旦你决定了哪个是 dockerize 的起点，让我们来介绍一下 ***Dockerfile*** 。为了定义在容器中成功启动和运行该文件(以及映像)所需的所有步骤，该文件必须出现在项目中。

Docker 文件必须直接存放在项目根目录下；pom.xml 文件所在的同一文件夹中。在我们的例子中，文件如下所示:

我们现在应该逐行解释这个文件的含义。

**第一行** *。设置基本图像容器来保存和运行项目。在我们的例子中，我们需要有一个 JDK 8* [*高山*](https://hub.docker.com/_/alpine) *版本。*

**第三行**。*复制。/usr/src/demo 文件夹中的 jar 文件。*

**第 5 行**。*将当前工作目录设置到上面的同一个文件夹中，其中。jar 文件已被复制。这意味着接下来的命令如* ***run、cmd 等……****将把当前工作目录作为基本路径。*

**第 7 行**。*暴露端口 8080。*

**第 9 行。** *执行命令‘Java-jar demo-0 . 0 . 1-snapshot . jar’运行程序。*

干得好。现在是时候了解一些最常见的 Docker 命令来管理我们的图像和容器了。

第一个命令是以这种方式使用的 **docker build**

```
docker build -t <your-docker-id>/<your-image-name> .
```

该命令从 Docker 文件所在的同一文件夹中的命令行启动，创建整个项目的 Docker 图像，并将一个标签关联到所创建的图像；在这种情况下*<your-docker-id>/<your-image-name>*。

执行这个命令，Docker 创建带有相关标签的图像。使用下面的命令可以列出所有的图像

```
*docker images* or *docker image ls*
```

在这里，你还可以看到他们唯一的 ID。

每当我们想要删除一幅图像，我们可以这样做，感谢

```
docker delete image <image-id>
```

最后，我们可以看到 run 命令。它能够将我们的图像放入容器中。

```
docker run --name <your-name-here> -p 8080:8080 -d <image-tag>
```

1.  - name <your-name-here>设置容器的名称</your-name-here>
2.  -p 设置并公开端口 8080，并执行本地机器到容器之间以及本地机器 8080 端口和容器的 8080 端口之间的请求转发
3.  -d 以守护模式执行容器
4.  <image-tag>表示在容器内运行的 Docker 图像</image-tag>

我们终于创建了我们的第一个 Docker 映像，并且能够在 Docker 容器中运行它。现在，让我们向[邮差](https://www.getpostman.com/)提出请求，享受我们做出的伟大作品吧！

在这个故事的最后一章中，我们将看到如何将 Docker 镜像推送到 docker hub 中的远程公共注册中心。

首先，让我们登录 docker

```
docker login --username <your-docker-id>
```

该命令将在隐藏模式下要求您输入密码。

请注意，您可以使用

```
--password <your-docker-hub-password>
```

在上面的命令中；但是，通过这种方式，您的密码在您的命令历史记录中仍然可见。

然后，以这种方式标记你的图像

```
docker tag <image-id> <docker-id>/<custom-image-name>
```

简单地说，执行下面的命令

```
docker push <docker-id>/<custom-image-name>
```

这个过程可能需要一段时间，但是最终，您应该会看到您的第一个存储库上传到您的私有 hub 中！

![](img/ded716c610ab93f31b09d1bfe7e109cd.png)

# 结论

在这个故事中，我们看到了如何从 SpringBoot 应用程序开始创建和管理 Docker 图像。用同样的方法，可以创建 Node.js 应用程序的映像等等。我们还讨论了如何在 docker hub 的公共注册表中上传图片，以备使用，例如在 k8s 中。

非常感谢您的阅读。如果你喜欢这个故事，请鼓掌跟随。

下次见！