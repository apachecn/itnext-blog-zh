# 四处移动 Docker 容器和图像

> 原文：<https://itnext.io/moving-docker-containers-and-images-around-cf604757d8b9?source=collection_archive---------2----------------------->

[![](img/a0e34fec13599cf2d6b30f1899f61a05.png)](http://www.giantswarm.io)

Giant Swarm 在`registry.giantswarm.io`运行一个 [Docker 容器注册表](https://github.com/docker/docker-registry)，为已经注册了[私有 alpha 测试](https://giantswarm.io/request-invite/)的用户提供便利。当您注册一个帐户时，您也会在我们托管的注册表中获得一个帐户。您可以使用这个*私有*注册表来存储您在 Giant Swarm 上运行的服务的容器映像。

我们还不支持的一个功能是从远程*私有*注册中心获取图像的能力，比如由 [Docker Hub](https://hub.docker.com) ，CoreOS 的 [Quay.io](https://quay.io/) 或[Google Container Registry](https://cloud.google.com/container-registry/)托管的注册中心。存储在这些服务上的公共图像对我们来说很好，我们计划在不久的将来增加对远程私人注册的支持。

在与一个潜在客户合作时，我发现自己需要在 Giant Swarm 上运行图像，这些图像存储在 Docker Hub 上的一个私人存储库中。在[和](http://stackoverflow.com/questions/26026931/setting-up-a-remote-private-docker-registry) [围绕](http://stackoverflow.com/questions/23935141/how-to-copy-docker-images-from-one-host-to-another-without-via-repository) [纠结了一会儿](http://stackoverflow.com/questions/28734086/how-to-move-docker-containers-between-different-hosts)之后，我终于想出了解决办法，并想在这里分享一下。

# 移动图像回购回购

如果您只是想将图像从一个存储库移动到另一个存储库，最简单的方法就是使用`docker tag`和`docker push`方法。我们将从 Docker Hub 中提取一个图像开始，这是`docker`使用的默认存储库:

```
$ docker pull ubuntu Using default tag: latest latest: Pulling from library/ubuntu d3a1f33e8a5a: Pull complete c22013c84729: Pull complete d74508fb6632: Pull complete 91e54dfb1179: Already exists library/ubuntu:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security. Digest: sha256:fde8a8814702c18bb1f39b3bd91a2f82a8e428b1b4e39d1963c5d14418da8fba Status: Downloaded newer image for ubuntu:latest
```

接下来，我们用所需的目标存储库标记图像:

```
$ docker tag ubuntu registry.giantswarm.io/kord/ubuntu
```

最后，我们将映像推送到新的注册表中:

```
$ docker push registry.giantswarm.io/kord/ubuntu The push refers to a repository [registry.giantswarm.io/kord/ubuntu] (len: 1) Sending image list Pushing repository registry.giantswarm.io/kord/ubuntu (1 tags) d3a1f33e8a5a: Image successfully pushed c22013c84729: Image successfully pushed d74508fb6632: Image successfully pushed 91e54dfb1179: Image successfully pushed Pushing tag for rev [91e54dfb1179] on {https://registry.giantswarm.io/v1/repositories/kord/ubuntu/tags/latest}
```

**注意**:在推送私人回购之前，您需要登录对方注册中心！

# 将图像从主机移动到主机

在上面的例子中，将图像从一个注册表移动到另一个注册表需要通过互联网拉和推图像。如果您只是需要将图像从一台主机移动到另一台主机，就像与办公室中的某个人共享图像一样，您可以在没有上传和下载图像开销的情况下获得类似的结果。

# 导出与保存

Docker 支持两种不同类型的方法将容器图像保存到一个 tarball 中:

*   `docker export` -将容器的运行或暂停的*实例*保存到文件中
*   `docker save` -将未运行的容器*图像*保存到文件中

# 使用导出

让我们先来看看`docker export`的方法。我们首先让 Bob 从 Docker Hub 下载一个 Ubuntu 图像:

```
$ docker pull ubuntu <snip> Status: Downloaded newer image for ubuntu:latest
```

现在鲍勃在交互模式下运行*实例*，并在根目录下添加一个名为`this`的文件:

```
$ docker run -t -i ubuntu /bin/bash root@defc757ce803:/# ls bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var root@defc757ce803:/# touch this root@defc757ce803:/# ls bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys this tmp usr var
```

如果 Bob 退出这个提示，他的容器将被销毁。因此，在另一个 shell 中，Bob 运行`docker export`命令来导出*实例的*图像:

```
$ # output truncated slightly $ docker ps CONTAINER ID IMAGE COMMAND defc757ce803 ubuntu "/bin/bash" $ docker export defc | gzip > ubuntu.tar.gz $ ls -lah ubuntu.tar.gz -rw-r--r-- 1 bob staff 63M Aug 28 13:56 ubuntu.tar.gz
```

我们现在可以让 Bob 使用 [sneakernet](https://en.wikipedia.org/wiki/Sneakernet) 将他电脑上的`ubuntu.tar.gz`文件上传到 Alice 的电脑上。Bob 完成后，Alice 可以在图像上使用一个`docker import`,并在这个过程中给它一个新的标签:

```
$ ls -lah ubuntu.tar.gz -rw-r--r-- 1 alice staff 63M Aug 28 13:56 ubuntu.tar.gz $ gzcat ubuntu.tar.gz | docker import - ubuntu-alice e7a0d776251fb5c7d61a6aa481f1aa8cf6f8c2936e893030ac481ff4499ec129
```

现在 Alice 可以运行容器并验证 Bob 的`this`文件是完整的:

```
$ docker run -t -i ubuntu-alice /bin/bash root@3f7f57eb9959:/# ls bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys this tmp usr var
```

# 使用保存

或者，Bob 可以简单地在 Ubuntu 映像上做一个`docker save`来给 Alice 一个容器的*映像*的副本，Alice 可以在没有 Bob 修改的情况下使用它:

```
$ docker save ubuntu | gzip > ubuntu-golden.tar.gz
```

然后爱丽丝会拿着那个副本`load`它，而不是做一个`import`:

```
$ gzcat ubuntu-golden.tar.gz | docker load
```

现在 Alice 运行容器并注意到 Bob 的`this`文件不在那里:

```
$ docker run -i -t ubuntu /bin/bash root@e11b3abb67de:/# ls bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```

我在这里注意到，如果 Alice 做了一个`docker import - ubuntu`而不是`docker load`，那么 docker **将存储零投诉的图像**。Docker 甚至会尝试启动一个导入的*实例*，该实例是用`docker export`保存的*。当它这样做时，它将无法运行容器中的任何内容。*

# 增加 boot2docker 的默认磁盘空间

在撰写本文的过程中，我遇到了一个关于`boot2docker`的*有趣的*问题，因为我的虚拟机用完了存储图像的空间。这就造成了拉图的神秘挂。

如果您要移动大量映像，您可能希望增加由`boot2docker`分配的虚拟机存储量:

```
$ boot2docker init -s 40000
```

显示的单位是千兆字节。

[![](img/ef438cb76c5d031f7368edcbd56d8e8d.png)](https://www.giantswarm.io/guide-cloud-native-stack?utm_campaign=Blog%20CTA%20Conversion&utm_source=Cloud%20native%20stack%20guide_Blog&utm_medium=Blog%20CTA&utm_term=cloud%20native%20stack%20guide)