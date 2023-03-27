# 在 ARM32v7 K3S Oracle 集群上运行 Rust。

> 原文：<https://itnext.io/running-rust-on-arm32v7-k3s-oracle-cluster-25e3ecabec52?source=collection_archive---------1----------------------->

![](img/a0ae6ccc808cdddaa854fd4e0a9b7e84.png)

## 介绍

在本文中，我们将探索如何创建一个示例 [Rust](https://www.rust-lang.org/) 项目和 Dockerfile，以便在 [ARM32v7 架构](https://github.com/docker-library/official-images#architectures-other-than-amd64)上运行它。

为了配置集群，我使用了这个项目 [k3s-oci-cluster](https://github.com/garutilorenzo/k3s-oci-cluster) ，因为 Oracle 为 ARM 工作负载提供了非常慷慨的免费层，您不妨尝试一下，或者使用您的 raspberry pi 集群。

这篇文章的来源是这里 [RCV](https://github.com/kainlite/rcv/) 和 docker 图像是[这里](https://hub.docker.com/repository/docker/kainlite/rcv)。

**先决条件**

集群是可选的，如果你有任何设备使用 ARM32v7 或 ARM64v8 上的 linux，你应该能够使用 docker 的例子。

*   [k3s-OCI-集群](https://github.com/garutilorenzo/k3s-oci-cluster)
*   [码头工人](https://hub.docker.com/?overlay=onboarding)
*   [铁锈](https://www.rust-lang.org/tools/install)

# 让我们跳到这个例子

## 创建项目

让我们用 Cargo 创建一个新的 Rust 项目，正如你可能注意到的，我们将得到一个非常基本的项目，它将帮助我们开始:

## 我们的例子和依赖关系

我在考虑处理 markdown 文件并将其显示为 html，所以这基本上是代码所做的，虽然远非最佳，但足以说明这个例子，首先让我们为 web 服务器添加一些板条箱( [Actix](https://actix.rs/docs/server/) )并将 [markdown 转换为 html](https://github.com/johannhof/markdown.rs) 。

## 让我们添加一些代码

这个简单的代码片段只监听`GET`对`/`的请求，记录一行 unix 时间戳和 IP，并返回文件`cv.md`的内容，这是我的简历。

## 示例日志

示例日志

至此，我们已经有足够的东西可以在本地运行和测试了，但是其他架构呢？(我运行的是 linux-amd64)，想运行`cargo run`可以本地测试。

## ARM32v7 Dockerfile

这个 docker 文件可以通过多种方式进行优化和保护，但为了简单起见，开始做一些事情就足够了，我们还将通过 kubernetes APIs 提供运行时的安全性。这里我们需要考虑两件事，首先我们需要使用 Rust 创建一个 ARM32v7 二进制文件，然后我们需要一个用于该架构的 Docker 映像，这就是 Docker 文件的基本功能。

## 构建和推送(docker 图像)

所以让我们建立它，并把它推到这里。

# 让我们快速回顾一下清单

清单相当简单，您可以在那里看到它们，正如您可以看到的，我们使用 pod 和容器的 SecurityContext 来限制容器的用户和权限。

## 部署它

假设您已经启动并运行了一个群集，可以像这样进行部署，您将看到一个部署、一个服务和入口资源，如果您想像我一样使用它，还需要一个 DNS 条目:

## 额外的

你可以在这里看到它运行，一个非常基本的 HTML 简历。

要了解更多细节并了解一切是如何组合在一起的，我鼓励您克隆回购协议，测试它，并修改它以制作您自己的回购协议。

# 清理

要清理资源，您可以这样做:

## 结束语

如果你想了解更多的例子，一定要查看链接，我希望你喜欢，在 [twitter](https://twitter.com/kainlite) 或 [github](https://github.com/kainlite) 上再见！

这篇文章的来源是[这里](https://github.com/kainlite/rcv/)

# 正误表

如果您发现任何错误或有任何建议，请给我发消息，以便解决问题。

此外，您还可以在这里查看源代码和[生成代码](https://github.com/kainlite/kainlite.github.io)和[源代码](https://github.com/kainlite/blog)的变化

*原载于 2022 年 9 月 2 日*[*https://tech squad . rocks*](https://techsquad.rocks/blog/rust_on_arm32v7/)*。*