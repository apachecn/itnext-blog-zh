# 在 docker 中使用缓存安装 npm

> 原文：<https://itnext.io/npm-install-with-cache-in-docker-4bb85283fa12?source=collection_archive---------3----------------------->

*添加一个新的* `*npm*` *包然后运行* `*docker build*` *真的很慢！以下是如何加快你的工作流程。*

![](img/2bb55092161db53e5fb13bf6dfc40b10.png)

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fnpm-install-with-cache-in-docker-4bb85283fa12)

# 问题是

除了通过层缓存机制，没有其他方法可以在 docker 构建之间持久化构建工件。您可以使用`VOLUME`动作指定一个或多个卷，但是它不是在构建时使用，而是在运行时使用。(也就是你做`docker run ...`的时候)。

如果你正在使用`npm`进行开发，添加一个新包的时间会变得非常长！对于每个`docker build`,整个`node_modules`被完全重新安装，从互联网上重新获取，不参考任何本地缓存。

在这篇文章中，我将概述一种有帮助的技术。

# 解决方案

首先，我承认在构建期间试图解决这个问题失败了，因为用 docker 在技术上是不可能的。有[工具](https://t.umblr.com/redirect?z=https%3A%2F%2Fgithub.com%2Fgrammarly%2Frocker&t=OWFiZTE3MmY2NjQ0YmFhNDgxMDcxZjRlZjk0NDA2YzZmZWRiNDBjMSxrQ1dZODhEZg%3D%3D&b=t%3AaI2GQmymS14nkWtOZDdQcw&p=http%3A%2F%2Ftimlesallen.tumblr.com%2Fpost%2F151774051780%2Fdocker-npm-cache&m=1)试图解决这个问题，但是它们必然会破坏与 docker 的兼容性(通过用缺失的功能扩展 docker)。我不热衷于打破兼容性，所以我回避这些。

诀窍是转而采用一种制度，在这种制度下，你可以添加新的包，并在需要时在运行时*获取它们。这样，您可以在开发工作流程中快速迭代。*

1.  给你的`package.json`添加一个新的脚本，它将会同时`npm install`和`npm start`。(参见[要点](https://gist.github.com/timlesallen/a7292bd1fb1674abb9a7fede9c08f7cd))
2.  如果您使用的是`docker-compose`，请更改您的`docker.compose.yml`文件以使用该脚本。

这意味着当你添加一个新的包到你的`package.json`中时，你会把这些新的包安装到你的容器中，而不需要重新构建。

当然，使用这种方法，您将获得“增量”(即，自从您上次`docker build`以来的新包，每次您启动一个新容器时都被拉下。

您可以通过告诉`docker-compose`创建一个卷，例如`npm-cache`，并将其映射到容器的`npm`缓存目录中，来减少这一步所需的时间。

要获得容器中的`npm`缓存目录的路径，只需执行`docker-compose run [servicename] npm get cache`。然后你可以将你的`docker-compose.yml`修改为[显示的](https://gist.github.com/timlesallen/a7292bd1fb1674abb9a7fede9c08f7cd)。

(不要忘记将`/root/.npm`替换为您自己的映像的缓存位置，这是您之前执行`npm get cache`时得到的。

这就是全部了！