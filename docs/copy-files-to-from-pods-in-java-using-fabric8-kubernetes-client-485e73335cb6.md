# 使用 Fabric8 Kubernetes 客户端在 Java 中复制文件

> 原文：<https://itnext.io/copy-files-to-from-pods-in-java-using-fabric8-kubernetes-client-485e73335cb6?source=collection_archive---------0----------------------->

![](img/6f8fc6c970d4c4727c35fbb5b811bc63.png)

[Fabric8 Kubernetes 客户端](https://github.com/fabric8io/kubernetes-client)

有时候，我们希望将一些文件复制到运行在 Kubernetes 中的应用程序窗格中，或者从应用程序窗格中将一些文件下载到本地系统中。你可以在网上看到各种关于`kubectl cp`的文章，但是这篇文章主要关注使用 [Fabric8 Kubernetes Client](https://github.com/fabric8io/kubernetes-client) 用 Java 以编程方式完成所有这些工作

# **获取客户端:**

你可以像往常一样在 [maven central](https://search.maven.org/search?q=g:io.fabric8%20a:kubernetes-client) 上找到客户端。您可以将它作为依赖项添加到您的 [maven](https://maven.apache.org/) 项目中:

fabric 8 Kubernetes POM . XML 中的客户端依赖项

如果你正在使用 [gradle](https://gradle.org/) ，你可能想使用这个:

Fabric8 Kubernetes 通过 gradle groovy DSL 实现客户端依赖性

一旦添加为依赖项，您就可以开始使用客户端了。文件复制操作包括压缩/解压缩文件/目录。Fabric8 Kubernetes 客户端包括 commons-codec 和 commons-compress 作为可选的依赖项。因此，您需要将它们添加到您的项目中，以便利用这些功能:

向类路径添加公共编码和公共压缩的可选依赖项

# **将文件从 Pod 复制到本地(下载):**

。让我们假设我有一个 pod 在 Kubernetes 中运行，如下所示:

一个简单的 [Quarkus](https://quarkus.io/) 应用程序舱正在运行

我想使用 Kubernetes 客户端以编程方式将这个文件`/deployments/quarkus-1.0.0-runner.jar`复制到我的本地机器上。我会这样做:

从 Pod 下载文件到您的机器。参见代号[DownloadFileFromPod.java](https://github.com/rohanKanojia/kubernetes-client-demo/blob/master/src/main/java/io/fabric8/DownloadFileFromPod.java)

# **将文件从本地复制到 Pod(上传):**

假设您在本地系统中有一个路径为`/home/rohaan/work/k8s-resource-yamls/jobExample.yml`的文件，您想将该文件复制到路径为`/tmp/jobExample.yml`的 pod 中。你可以这样做:

上传文件到 Pod。UploadFileToPod.java 见

# 通过 InputStream 从 Pod 读取文件(下载):

让我们试着读取我们在前面的上传文件到 pod 的例子中复制的相同文件，但是使用 java 流。你可以这样做:

使用 InputStream 从 Pod 读取文件。参见 ReadFileAsStream.java 的

# **将本地目录复制到 Pod(上传):**

我有一个包含这两个文本文件的小目录:

我想把这个文件夹`test-dir`复制到我的 Pod 中。我会这样做:

上传一个目录到 Kubernetes Pod。参见[UploadDirectoryToPod.java](https://github.com/rohanKanojia/kubernetes-client-demo/blob/master/src/main/java/io/fabric8/UploadDirectoryToPod.java)

# 向/从多容器窗格复制时指定容器:

您还可以使用为您过滤容器的`inContainer(String containerName)` DSL 方法来指定选择哪个容器。它适用于从 pod 上传和下载文件。以下是从 Pod 下载文件时选择容器的示例:

从具有多个容器的 Pod 中复制文件。参见[DownloadFileFromMultiContainerPod.java](https://github.com/rohanKanojia/kubernetes-client-demo/blob/master/src/main/java/io/fabric8/DownloadFileFromMultiContainerPod.java)

我今天的博客到此结束。我希望它是有用的。非常感谢您抽出时间阅读。你可以在我的 Github 库中找到上面的 gists 共享代码:

[](https://github.com/rohanKanojia/kubernetes-client-demo) [## rohanKanojia/kubernetes-客户端-演示

### 该项目包含 Fabric8 Kubernetes 客户端不同用法的各种示例。我通常在我的…

github.com](https://github.com/rohanKanojia/kubernetes-client-demo) 

# 加入我们:

你喜欢 Fabric8 Kubernetes 的客户，并希望参与项目的开发。随意参与方法:

*   当某些东西不能按预期工作时，创建 [Github 问题](https://github.com/fabric8io/kubernetes-client/issues)
*   向我们发送[拉请求](https://github.com/fabric8io/kubernetes-client/pulls)以修复错误/添加增强功能
*   在我们的 [Gitter 频道](https://gitter.im/fabric8io/kubernetes-client?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)上与我们聊天
*   在 [Twitter](https://twitter.com/fabric8io/) 上关注我们

最初发表于 [Wordpress](https://r0haan.wordpress.com/2020/09/25/copy-files-to-from-pods-in-java-using-fabric8-kubernetes-client/)