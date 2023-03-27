# 我秋，秋，选择你的容器图像第 3 部分。

> 原文：<https://itnext.io/i-cho-cho-choose-you-container-image-part-3-b4eadb4f1573?source=collection_archive---------11----------------------->

![](img/28ac750b9dff7af2dd05a029bde4ac3f.png)

在本系列的前两篇文章([第一部分](/i-cho-cho-chose-you-container-image-part-1-fa6671d9ae1f)和[第二部分](/i-cho-cho-choose-you-container-image-part-2-44b45e47a1f7))中，我们已经了解了作为开发人员我们可以使用哪些基础映像。然后我们用 Golang 处理编译语言。现在，在这篇文章中，我们将使用动态语言来看看我们可用的选项。在我们开始之前，我们应该弄清楚一些事情。如果你一直在关注这个系列，我们已经讨论过了 scratch base 图像。我有一些坏消息，如果你使用动态语言，你将不能使用 scratch，因为你需要安装运行时，不像编译语言。所以在这篇文章中，我们将使用完整的操作系统，超薄操作系统和 Alpine Linux 映像。

如果你想利用动态语言的多阶段构建，那就有点微妙了。因为我们不需要复制一个二进制文件，所以我们仍然可以使用多阶段构建。Python 实际上非常适合在多阶段构建中使用，因为它有 [virtualenv](https://packaging.python.org/guides/installing-using-pip-and-virtualenv/) 提供了运行应用程序所需的依赖关系的逻辑分离。要使用多阶段构建，您需要将 virtualenv 目录从构建容器复制到最终映像。与 Golang 示例一样，我们使用的应用程序相当轻量级。只是一个简单的 Python web 服务器托管静态 HMTL 文件。如果您想查看代码，请点击[这里](https://github.com/scotty-c/container-image-examples/blob/master/python/web.py)。

如前所述，对于动态语言，我们有三种选择，因此我们将从完整的操作系统映像开始。

```
FROM python:3.8.0a2-stretch as build

RUN python3 -m venv /web

COPY . /web

FROM python:3.8.0a2-stretch

COPY --from=build /web /web

WORKDIR /web

EXPOSE 8080

ENTRYPOINT [ "python", "web.py" ]
```

一旦我们使用多阶段构建用上面的 docker 文件构建了我们的基本映像，我们就有了一个映像`945mb`,所以你可以看到由于需要运行时，动态语言容器映像比编译语言要大得多。现在记住我们的第一个帖子，这张图片也包含关键的漏洞。因此，让我们来看看这个苗条的图像，看看我们能节省多少图像尺寸。

```
FROM python:3.8.0a2-slim-stretch as build

RUN python3 -m venv /web

COPY . /web

FROM python:3.8.0a2-slim-stretch

COPY --from=build /web /web

WORKDIR /web

EXPOSE 8080

ENTRYPOINT [ "python", "web.py" ]
```

在使用精简操作系统作为基础映像的构建中，我们得到了一个大小为`159mb`的映像，因此您可以看到，通过从完整的操作系统映像交换到相同操作系统的精简版本，可以节省大量空间。不幸的是，这种图像仍然带有易受攻击的包。那么，如果我们将应用程序迁移到 Alpine Linux，我们会得到什么好处呢？

```
FROM python:3.8.0a2-alpine3.9 as build

RUN python3 -m venv /web

COPY . /web

FROM python:3.8.0a2-alpine3.9

COPY --from=build /web /web

WORKDIR /web

EXPOSE 8080

ENTRYPOINT [ "python", "web.py" ]
```

现在，迁移到 Alpine Linux 给我们带来了最小的映像大小，没有严重的漏洞。现在我们在[第一部](/i-cho-cho-chose-you-container-image-part-1-fa6671d9ae1f)中简要谈到了这个问题。Alpine 没有使用`glibc`，而是使用了`[musl li](https://www.musl-libc.org/)bc`,这对于动态语言来说更加重要，如果你有依赖于`glibc`的依赖关系，你可能会遇到 Alpine 的问题。

如果你想看看这一系列文章的代码，请访问我的 GitHub 页面。要进一步了解容器或 Kubernetes，请访问我的 OSS 工作室，或者你也可以通过 twitter @scottcoulton 联系我