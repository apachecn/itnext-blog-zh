# 詹金斯的码头工人

> 原文：<https://itnext.io/docker-inside-docker-for-jenkins-d906b7b5f527?source=collection_archive---------0----------------------->

![](img/9a36a946d65409ccc4167f51d3d698a6.png)

[SaladDayZ](https://www.reddit.com/user/SaladDayZ/) 原图发布于[https://www . Reddit . com/r/mildlyinterising/comments/3g menb/this _ is _ what _ shipping _ containers _ look _ like/](https://www.reddit.com/r/mildlyinteresting/comments/3gmenb/this_is_what_shipping_containers_look_like/)

在我们公司，我们目前在 dockers 上运行所有内部服务，例如 Gitlab、YouTrack 和 Jenkins。它提供了高度的灵活性，但也产生了一些特定的问题。今天我想谈谈其中的一个——docker inside docker。我假设你已经知道 Docker、Jenkins 和 Ubuntu(或者其他 Linux)，因为我们用它们做例子。

对于我们的 CI，我们决定使用詹金斯。我们用的是 dockerized 版本:【https://hub.docker.com/r/jenkins/jenkins/[。我们构建多个不同的项目，这些项目需要不同的环境。因此，我们选择在单独的 dockers 中运行所有任务。这样，我们可以为每个项目设置不同的设置。这种环境由开发人员管理，与他们的机器和预览服务器上的环境相同。当我们这样做时，我们在 docker 内部运行 docker。经过一些研究，我们决定用一个主机守护进程来运行 docker，而不是在容器内部运行一个容器。这样，我们就遇到了对 *docker.sock* 的权限问题。让我们走完每一步，一起解决所有问题。](https://hub.docker.com/r/jenkins/jenkins/)

# 这是怎么回事？

我们先来了解一下是怎么回事。

我们的主机是詹金斯容器的主机。当 Jenkins 需要运行任务时，它会构建一个 docker 容器，并在其上运行特定的命令。所以我们和詹金斯一起在码头工作。如果您尝试在虚拟机上运行虚拟机，您会遇到许多问题。

但是有办法在我们的容器之外直接在主机上运行所有的 dockers。在这种情况下，容器内部调度的所有 docker 命令都将在主机上处理。怎么会这样呢？所有 docker 命令都由 docker 服务运行，docker 服务可通过 socket 获得。当我们运行 cli 时，应用程序通过该套接字向服务发送命令。默认情况下，socket 是作为一个文件公开的，例如 */var/run/docker.sock* 。因此，如果我们在容器上运行 cli，它将与主机上的服务通信，而不是与容器上的服务通信，那么我们将有可能从容器中在主机上运行 docker。我们需要做的唯一事情是在容器中安装暴露的套接字，而不是它的默认套接字。让我们和詹金斯一起做吧。

# 调整 Jenkins 容器

首先，我们需要为 jenkins 容器创建自己的图像。以下是 Dockerfile:

```
FROM jenkins/jenkins
USER root
RUN apt-get -y update && \
 apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common && \
 curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | apt-key add — && \
 add-apt-repository \
 “deb [arch=amd64] [https://download.docker.com/linux/$(.](https://download.docker.com/linux/$(.) /etc/os-release; echo “$ID”) \
 $(lsb_release -cs) \
 stable” && \
 apt-get update && \
 apt-get -y install docker-ce docker-ce-cli containerd.io
RUN curl -L “[https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname](https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname) -s)-$(uname -m)” -o /usr/local/bin/docker-compose && \
 chmod +x /usr/local/bin/docker-compose && \
 ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
USER jenkins
```

首先，我们声明我们的基础映像。接下来，我们切换到根用户。我们基于的镜像不包含 docker，所以我们安装它。最后，我们默认切换到 jenkins 用户来使用它。现在我们需要使用该映像来运行我们的容器。为此，我们将使用 docker-compose。下面是我们的 *docker-compose.yml* :

```
version: '3.1'
services:
    jenkins:
        build:
            context: ./
        restart: unless-stopped
        volumes:
            - ${HOST_DOCKER}:/var/run/docker.sock
            - ${HOST_JENKINS_DATA}:/var/jenkins_home
        ports:
            - "${HOST_WWW}:8080"
            - "${HOST_OTHER}:50000"
```

现在让我们用*完成 docker 环境变量的配置。env* 文件:

```
HOST_WWW=8180
HOST_OTHER=8181
HOST_DOCKER=/var/run/docker.sock
HOST_JENKINS_DATA=/srv/www/jenkins
```

正如您在主机上看到的，我们将有 */srv/www/jenkins* ，它将作为 */var/jenkins_home* 安装在容器上。这使我们有可能在重启容器时保留所有 Jenkins 配置。主机上的端口 8180 将被映射为容器上的 8080，端口 8181 将被映射为 50000。我将不解释 HTTP 在主机上的配置，因为这不是那篇文章的主题。这里最重要的是从主机上安装 */var/run/docker.sock* ，作为容器上的 */var/run/docker.sock* 。这样，容器中的 docker cli 将与主机上的 docker 服务进行通信。

当我们准备好了，我们运行`docker-compose build`和`docker-compose up`。我们用`docker-compose exec jenkins bash`连接到容器。现在在容器内部，当我们运行`docker ps`命令时，我们应该看到所有的容器都运行在我们的主机上，而不是运行在我们的 jenkins 容器上。但是当我们运行该命令时，我们收到了权限被拒绝错误。让我们修理它。

# 拒绝解决 docker.sock 权限

在大多数情况下，我们使用特定的用户运行 docker 守护进程，其他所有用户都无权访问它。

先说主机配置。在我们的例子中，我们创建了具有 sudo 权限的用户 jenkins。让我们以该用户身份登录并运行`docker ps`命令。我们应该在 docker.sock 上获得被拒绝的许可。如果我们用 sudo 运行该命令，它将会工作。有一个具有所需权限的 docker 用户组，因此我们将 jenkins 用户添加到该组并重新启动 docker:

```
usermod -aG docker jenkins
sudo service docker restart
```

我们需要注销并重新登录，然后重新运行`docker ps`。这一次，主机上的一切都正常，但是当我们在容器上运行`docker ps`时，仍然会得到相同的错误。为什么？

通常，容器和主机是两个独立的环境。在两者中创建的所有用户也是独立的。如果我们想要相同的用户，我们需要正确地配置他们。我们在主机和容器上已经有了 jenkins 用户，但是他们是不同的用户。用户由 uid 标识，而不是用户名。所以我们需要确保两个用户有相同的 uid。
首先，我们需要验证主机上的 uid:

```
> id -u jenkins
1004
```

如您所见，我们收到了 uid — 1004。我们将更改 docker 文件来重新配置容器上的 jenkins 用户。我们通过在 Dockerfile 中添加两行来实现。首先在文件的顶部(在基本映像声明之后)定义可接受的构建参数:

```
ARG HOST_UID=1004
```

其次，在我们切换到 jenkins 用户之前，要更改用户 uid:

```
RUN usermod -u $HOST_UID jenkins
```

接下来，我们更改 docker-compose.yml 中的构建定义:

```
services:
    jenkins:
        build:
            context: ./
            args:
                HOST_UID: ${HOST_UID}
```

最后，我们在。环境:

```
HOST_UID=1004
```

现在我们的 uid 是正确的，但是我们仍然有一个问题——为什么？主机和容器都把我们的用户看做是一样的(他们还是不同的用户，只是我们骗过了用户验证)。权限是为用户组而不是特定用户设置的。容器中还有哪些 jenkins 用户不属于该组。我们讨论 docker 用户群。主机和容器中的组具有相同的名称，但组 id 不同(与用户的问题相同)。首先，我们需要知道主机上 docker 组的 gid:

```
>getent group | grep docker
docker:x:999:jenkins
```

数据的第三部分包含 gid —在我们的例子中是 999。接下来，我们调整 docker 文件，以确保 docker 组在主机和容器上具有相同的 id，并且 jenkins 用户在该组中。首先，我们添加一个新的参数来构建:

```
ARG HOST_GID=999
```

接下来，我们更改组并修改用户，然后切换到 jenkins 用户:

```
RUN groupmod -g $HOST_GID docker
RUN usermod -aG docker jenkins
```

我们再次更改 docker-compose.yml 中的构建定义:

```
services:
    jenkins:
        build:
            context: ./
            args:
                HOST_UID: ${HOST_UID}
                HOST_GID: ${HOST_GID}
```

我们还在中添加了正确的变量。环境文件:

```
HOST_GID=999
```

现在我们再次构建 docker 映像并运行容器(`docker-compose build`、`docker-compose up`)。从现在开始，我们的 Jenkins 可以在主机上运行 docker 的容器，不会有任何问题。

如果您不能使用 jenkins due 权限运行容器来处理 jenkins 文件，您需要在主机上调整文件的所有权。正如你所记得的，我们对用户 id 做了一点改动，这可能是一个问题。在我们的示例中，我们需要在主机上运行:

```
sudo chgrp -R /srv/www/jenkins jenkins
sudo chown -R /srv/www/jenkins jenkins
```

就这些了。有了这个配置，我们可以在 docker 中运行 Jenkins，并使用 docker 进行构建。你可以在这里的[https://github.com/smoogie/jenkins_docker_example](https://github.com/smoogie/jenkins_docker_example)中找到 *Dockerfile* 和 *docker-compose.yml* 的例子