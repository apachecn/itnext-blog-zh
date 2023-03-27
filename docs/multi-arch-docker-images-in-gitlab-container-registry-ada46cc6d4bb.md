# Gitlab 容器注册表中的多拱 docker 图像

> 原文：<https://itnext.io/multi-arch-docker-images-in-gitlab-container-registry-ada46cc6d4bb?source=collection_archive---------1----------------------->

即使用与 x86/x64 相同的标签将 ARM 映像推送到 Docker 注册表

![](img/276fad08a799dd3dd4f01b7d43bcf615.png)

安托万·佩蒂特维尔在 [Unsplash](https://unsplash.com/s/photos/containers-architecture?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

# TL；速度三角形定位法(dead reckoning)

使用标准的 *buildx* 命令([引用](https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/))，可以构建并推送多拱映像(x86、ARM、amd64)到 Gitlab 容器注册表。

上面的例子使用了一个[样例 repo](https://gitlab.com/nicosingh/demo-flask-application) ，在这里您可以检查 *Dockerfile* 根本没有特定于架构的定义。为多个架构构建映像的技巧是使用一个已经支持所有架构的基础映像( *python:3.8-alpine* )。

# 完整版

本帖是[本文](https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/)的延伸，讲解了如何用 Gitlab 容器注册表替代 Docker Hub 推送多拱 Docker 图片。

多拱映像允许我们在各种架构的设备中使用相同的 docker 映像，为在 ARM 设备(如 Raspberry Pis 或基于 AWS Graviton 的 EC2 实例)中部署轻量级和高度可扩展的服务带来了一个很好的替代方案(便宜！).

直奔主题，有两种方法可以构建多拱图像:使用清单或者使用 [buildx 命令](https://docs.docker.com/buildx/working-with-buildx/)。为了简单起见，我们将使用 *buildx* 来构建& push 我们的多拱图像到 Gitlab。

除了使用 Gitlab 作为我们的 Docker 注册表而不是 Docker Hub 之外，参考文章中详细描述的步骤没有太大区别。这意味着第一步是使用我们的个人凭证或访问令牌登录 Gitlab 容器注册表:

然后，我们可以使用如下命令构建并推送我们的多拱映像:

正如我们可以在这个示例报告中看到的，[docker file](https://gitlab.com/nicosingh/demo-flask-application/-/blob/master/Dockerfile)与该架构没有任何特殊关系，因为它基于另一个多拱图像:

如您所见，这个 Docker 图像使用 python 创建了一个示例应用程序。在本例中，我们使用 Flask 创建了一个简单的 web 服务器:

Buildx 的*平台*选项允许我们指定构建我们的映像的架构作为列表，而*推送*选项允许我们在构建阶段一结束就推送映像。

# CI 作业

事实证明，在使用 docker-in-docker (dind)的 GitlabCI 管道中，使用 *buildx* 命令可以完美地工作😀

以下示例显示了一个简单的 CI 作业，它使用官方 Docker 映像下载 *buildx* ，然后使用它来构建&将 Flask 应用程序推送到容器注册表:

# 包扎

就是这样！从现在开始，可以从使用相同图像标签的多个设备(笔记本电脑、raspberry pi、EC2 实例等)使用 docker 图像。

✅请记住，所有这些步骤都适用于任何支持多拱图像的 Docker 注册表(例如 DockerHub、AWS ECR、)。

我真的很感激任何类型的反馈，疑问或评论。在这里随意 ping 我[，或者在这个帖子里发表任何评论。](https://www.linkedin.com/in/nicosingh)

[](https://github.com/sponsors/nicosingh) [## GitHub 赞助商上的赞助商@nicosingh

### 非常感谢你的支持😊我感谢你对做开源软件和分享知识的认可…

github.com](https://github.com/sponsors/nicosingh)