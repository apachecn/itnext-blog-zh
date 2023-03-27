# 我秋，秋，选择你集装箱图像第 2 部分。

> 原文：<https://itnext.io/i-cho-cho-choose-you-container-image-part-2-44b45e47a1f7?source=collection_archive---------8----------------------->

![](img/d26efe9dc7b708cc3d83ec467012910d.png)

在这个系列的第一部分[中，我们看了可以用作基本容器图像的不同类型的图像。在这个系列的第二篇文章中，我们将在我们的基础图片上添加一个真实的应用程序。正如我们之前的帖子中提到的，我们将选择一种编译语言，在这种情况下，它将是](https://medium.com/@scott.coulton/i-cho-cho-chose-you-container-image-part-1-fa6671d9ae1f) [Golang](https://golang.org/) 。在本系列的下一篇文章中，我们将探讨一种动态语言(Python)。

所以我们将采用一个非常简单的 Golang 应用程序,它基本上是为一个静态 HTML 文件服务的。不要太沉迷于应用程序的构造。最重要的是，它可以编译，体积小，构建速度快，所以我们可以看到什么样的容器基础映像最适合使用。

在我们选择从哪种操作系统开始之前，对于编译语言来说，一个最佳实践是 Docker 多阶段构建。多阶段构建给我们的是在一个容器中构建二进制文件的能力。这通常也包括所有的构建工具。然后将二进制文件复制到一个新的容器中。因此，抛弃运行生产应用程序所需的所有构建工具和不需要的包。有关多阶段构建的更多信息，请访问[此处](https://docs.docker.com/develop/develop-images/multistage-build/)

对于所有不同的基础映像，我们将使用多阶段构建来尽量保持我们的可部署映像尽可能小。因此，在每个 docker 文件中，我们将在构建容器中编译 Go 二进制文件，然后使用多阶段构建将其移动到最后一个容器，这将是我们的部署工件。

现在，我们将选择的第一个基本容器映像是完整的操作系统，因为如果您不熟悉容器，这将是最容易启动和运行的。

```
FROM golang:1.11-stretch as build

WORKDIR /go/src/github.com/scottyc/webapp

COPY web.go web.go

RUN CGO_ENABLED=0 GOOS=linux go build -o ./bin/webapp github.com/scottyc/webapp

FROM debian:stretch

RUN mkdir -p /web/static/ 

COPY --from=build /go/src/github.com/scottyc/webapp/bin/webapp /usr/bin
COPY index.html /web/static/index.html

WORKDIR /web

EXPOSE 3000

ENTRYPOINT ["webapp"]
```

这个 docker 文件将生成一个图像，该图像是`107mb`并且记得在第一个帖子中图像中有一些严重的漏洞。现在让我们来看看超薄操作系统映像。

```
FROM golang:1.11-stretch as build

WORKDIR /go/src/github.com/scottyc/webapp

COPY web.go web.go

RUN CGO_ENABLED=0 GOOS=linux go build -o ./bin/webapp github.com/scottyc/webapp

FROM debian:stretch-slim

RUN mkdir -p /web/static/ 

COPY --from=build /go/src/github.com/scottyc/webapp/bin/webapp /usr/bin
COPY index.html /web/static/index.html

WORKDIR /web

EXPOSE 3000

ENTRYPOINT ["webapp"]
```

现在你会注意到，我们使用完整的操作系统作为构建映像，这完全没问题，因为我们将丢弃它，这个构建将产生一个大小为`61.9mb`的映像。因此，在不改变任何应用程序代码的情况下，这是一个相当大的减少。我们仍然需要记住映像中有易受攻击的包。现在让我们试试 Alpine Linux。

```
FROM golang:1.11.2-alpine3.8 as build

WORKDIR /go/src/github.com/scottyc/webapp

COPY web.go web.go

RUN CGO_ENABLED=0 GOOS=linux go build -o ./bin/webapp github.com/scottyc/webapp

FROM alpine:3.8

RUN mkdir -p /web/static/ 

COPY --from=build /go/src/github.com/scottyc/webapp/bin/webapp /usr/bin
COPY index.html /web/static/index.html

WORKDIR /web

EXPOSE 3000

ENTRYPOINT ["webapp"]
```

现在这个构建产生了一个大小为`11mb`的映像，并且在映像中没有易受攻击的包。如您所见，迁移到 Alpine 极大地减小了图像大小，并增强了我们的安全态势。最后但同样重要的是，让我们看看 scratch。

```
FROM golang:1.11.2-alpine3.8 as buildWORKDIR /go/src/github.com/scottyc/webappCOPY web.go web.go
COPY index.html /web/static/index.htmlRUN CGO_ENABLED=0 GOOS=linux go build -o ./bin/webapp github.com/scottyc/webappFROM scratchCOPY --from=build /go/src/github.com/scottyc/webapp/bin/webapp /usr/bin/webapp
COPY --from=build /web/static/index.html /web/static/index.htmlEXPOSE 3000ENTRYPOINT ["/usr/bin/webapp"]
```

现在你会注意到我们使用 Alpine 来构建我们的二进制文件，因为 scratch 没有工具或包管理器。因此，我们将使用 Alpine 中的构建工具，然后将二进制文件移动到暂存容器中。暂存容器将只包含 Go 二进制文件。这将产生具有最佳安全状态的图像尺寸`6.59mb`。因此，正如您所看到的，通过选择不同的容器基础映像，我们用相同的应用程序代码产生了非常不同的结果。如果你想试驾这些图片，请访问我的 [Github 页面](https://github.com/scotty-c/container-image-examples)。在本系列的下一篇文章中，我们将关注动态语言并使用 Python。

为了进一步了解容器或 Kubernetes，请查看[我的 OSS 工作室](https://github.com/scotty-c/kubernetes-on-azure-workshop)