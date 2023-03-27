# 在你的笔记本电脑上使用 Multipass 和 Ubuntu 20.04 创建 Docker Swarm

> 原文：<https://itnext.io/creating-a-docker-swarm-using-multipass-and-ubuntu-20-04-on-your-laptop-fbf7ea85547c?source=collection_archive---------4----------------------->

Docker 是一个很酷的系统，可以将应用程序部署为可重用的容器，而 Docker Swarm 是一个 Docker Orchestrator，让我们可以在多台机器上扩展容器的数量。Multipass 是一个运行在 Windows、Linux 和 macOS 上的轻量级虚拟机管理器应用程序，它让我们可以轻松地在笔记本电脑上设置多个 Ubuntu 实例，而不会影响性能。因此，Multipass 可以作为一种手段，轻松地在您的笔记本电脑上试验 Docker Swarm，了解它如何工作，设置网络等。

![](img/ff425b23e3a54f73056b9ed646ba0bb9.png)

理论上这并不难，因为在 Multipass 上启动 Ubuntu 实例就像在 Ubuntu 上安装 Docker 和 Docker Swarm 一样简单。但是有足够多的活动部分，所以回顾一下如何在 Multipass 上安装 Docker，然后初始化 Docker Swarm 是很有用的。此外，设置脚本来轻松初始化 Docker 群组实例也很有用，因为群组在跨多个系统时当然更有用。

Docker Swarm 是众多 Docker 编排平台之一。它处理跨一组节点分发 Docker 容器，通过根据需要重新部署容器来确保群集的可靠性等等。Multipass 为我们提供了一种不用离开笔记本电脑就能试验 Docker Swarm 的简单方法。

要安装 Multipass，请进入 [https://multipass.run](https://multipass.run) ，找到您系统的安装程序。或者，您会发现在一些包管理系统中可以使用 Multipass。

在 Ubuntu 20.04 上设置 Docker Swarm 的步骤如下:

*   在 Multipass 上初始化 Ubuntu 20.04 实例
*   在实例内部，执行步骤来初始化 Ubuntu 20.04 上的 Docker
*   在第一个这样的实例中运行`docker swarm init`
*   在后续实例中，运行命令使实例加入群

在 Ubuntu 上安装 Docker 的官方说明位于:

*   [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)—在 Ubuntu 上安装 Docker-CE
*   [https://docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/)—运行 Linux 的安装后步骤

而在 Windows 上我们安装 Docker for Windows，在 macOS 上我们安装 Docker for macOS，在 Linux 上我们只是安装 Docker。那是因为 Docker 原生运行在 Linux 上。

我们的目标是将官方指令转换成 shell 脚本。因此，我们可以随时轻松地创建新实例，并快速构建任何规模的群集群。

# 用于初始化 Ubuntu/Docker/Docker 群实例的 Shell 脚本

我将官方指令翻译成可重复脚本的实验让我创建了两个脚本。

首先，创建`init-instance.sh`包含:

```
NM=$1 multipass launch --name ${NM} focal
multipass transfer setup-instance.sh ${NM}:/home/ubuntu/setup-instance.sh
multipass exec ${NM} -- sh -x /home/ubuntu/setup-instance.sh
```

实例的名称将在命令行上传递。在`multipass launch`中，我们指定了一个图像名`focal`，它指的是 Ubuntu 20.04。在撰写本文时，2020 年 5 月，20.04 映像是最新的，但是它还不像 LTS 版本那样支持多通道。

这个脚本的其余部分将脚本`setup-instance.sh`转移到新创建的实例，然后执行该脚本。虽然`setup-instance.sh`中的大部分命令都可以使用`multipass exec`来执行，但有些命令并不能很好地执行，因此有必要转而使用 shell 脚本。

创建`setup-instance.sh`包含:

```
sudo apt-get update 
sudo apt-get upgrade -y 
sudo apt-get -y install \
     apt-transport-https ca-certificates \
     curl gnupg-agent software-properties-commoncurl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -sudo apt-key fingerprint 0EBFCD88 sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" sudo apt-get update 
sudo apt-get upgrade -y 
sudo apt-get install -y docker-ce docker-ce-cli containerd.io 
sudo groupadd docker
sudo usermod -aG docker ubuntu
sudo systemctl enable docker
```

这是官方说明中列出的命令，有几个小小的音译。第一部分更新已安装的包，然后给`apt-get`添加从 HTTPS 服务器加载包的能力。然后我们为 Docker 设置 Ubuntu 包存储库，然后从该存储库安装 Docker。最后，我们确保`docker`组存在，并将`ubuntu`用户 ID 添加到该组，以便我们可以运行`docker`命令，并通过确保`docker`守护进程在系统启动时运行来完成。

这不做任何 Docker 群命令。但是它确实让 Docker 运行起来，您可以这样验证:

```
$ multipass exec NAME -- docker run hello-world
```

这是验证 Docker 初始化的标准方式，以运行`hello-world`容器。如果配置正确，它将下载`hello-world`图像，然后实例化一个容器，这将打印出一条有用的消息。替换您指定的实例名`NAME`。

# 在多遍 Ubuntu 实例上建立 Docker 群

有了这些脚本，我们可以启动一个 Ubuntu/Docker 实例，并初始化 Swarm。

```
$ sh -x init-instance.sh swarm1 
+ NM=swarm1 
+ multipass launch --name swarm1 focal Launched: swarm1 
+ multipass transfer setup-instance.sh swarm1:/home/ubuntu/setup-instance.sh 
+ multipass exec swarm1 -- sh -x /home/ubuntu/setup-instance.sh 
+ sudo apt-get update 
... much output 
+ sudo systemctl enable docker 
Synchronizing state of docker.service with SysV service script with /lib/systemd/systemd-sysv-install. 
Executing: /lib/systemd/systemd-sysv-install enable docker
$
```

在一分钟左右的时间内，这将启动一个 Multipass 实例并设置 Docker。

```
$ multipass list 
Name State IPv4 Image 
swarm1 Running 192.168.64.14 Ubuntu 20.04 LTS $ multipass exec swarm1 -- docker run hello-world 
Unable to find image 'hello-world:latest' locally 
latest: Pulling from library/hello-world 0e03bdcc26d7: Pull complete 
Digest: sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb426ad9a20c7ee045538ef34847f44f1 
Status: Downloaded newer image for hello-world:latest 
Hello from Docker! 
...
```

`swarm1`实例正在运行，我们可以执行 Docker。让我们初始化 Docker Swarm。

```
$ multipass exec swarm1 -- docker swarm init Swarm initialized: current node (xki1yma9q91401zv0i0yksrht) is now a manager.To add a worker to this swarm, run the following command:      docker swarm join --token SWMTKN-1-10r3749fp92qk7bpszvv2ejvdrbc4vkoaeks7kkpqdlytwbcq8-33a4kq3xpb315zrwhghs68atx 192.168.64.14:2377 To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions. $ multipass exec swarm1 -- docker node ls 
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION 
xki1yma9q91401zv0i0yksrht *   swarm1              Ready               Active              Leader              19.03.9
```

这个群体中有一个节点，这个节点是这个群体的领导者。当你是唯一投票的人时，很容易被选为领袖，嗯？

输出包括一个命令，用于您希望添加到集群的任何新 Docker 系统。您不需要记录命令字符串，而是需要学习这两个命令:

```
$ multipass exec swarm1 -- docker swarm join-token manager 
$ multipass exec swarm1 -- docker swarm join-token worker
```

`docker swarm join-token {manager,worker}`命令为您提供了一个节点作为管理节点或工作节点加入集群的命令字符串。

群集群中有两种节点。

*   *管理器*节点维护群集群。它们相互通信，并自动维护群集中所有节点的共享状态。管理器间通信通道被称为*控制平面*。
*   *Worker* 节点不参与集群的管理，只是在集群内执行任务。管理器节点也可以像工人一样执行任务，这使得 Docker Swarm 比大多数工作场所更加平等。

# 向群中添加第二个管理器

```
$ sh -x init-instance.sh swarm2 
... much output 
$ multipass exec swarm1 -- docker swarm join-token manager 
To add a manager to this swarm, run the following command:      docker swarm join --token SWMTKN-1-10r3749fp92qk7bpszvv2ejvdrbc4vkoaeks7kkpqdlytwbcq8-4bfaew5hps8mntqubxt9kkc4n 192.168.64.14:2377 $ multipass exec swarm2 -- docker swarm join --token SWMTKN-1-10r3749fp92qk7bpszvv2ejvdrbc4vkoaeks7kkpqdlytwbcq8-4bfaew5hps8mntqubxt9kkc4n 192.168.64.14:2377 
This node joined a swarm as a manager. $ multipass exec swarm1 -- docker node ls 
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION xki1yma9q91401zv0i0yksrht *   swarm1              Ready               Active              Leader              19.03.9 wgcj54zs2u3k9x1th3s20s1lm     swarm2              Ready               Active              Reachable           19.03.9
```

现在有两个节点。

# 向群中添加一个工作节点

```
$ sh -x init-instance.sh swarm3 
... $ multipass exec swarm1 -- docker swarm join-token worker 
To add a worker to this swarm, run the following command: docker swarm join --token SWMTKN-1-10r3749fp92qk7bpszvv2ejvdrbc4vkoaeks7kkpqdlytwbcq8-33a4kq3xpb315zrwhghs68atx 192.168.64.14:2377 $ multipass exec swarm3 -- docker swarm join --token SWMTKN-1-10r3749fp92qk7bpszvv2ejvdrbc4vkoaeks7kkpqdlytwbcq8-33a4kq3xpb315zrwhghs68atx 192.168.64.14:2377 
This node joined a swarm as a worker. $ multipass list 
Name                    State             IPv4             Image swarm1                  Running           192.168.64.14    Ubuntu 20.04 LTS 
swarm2                  Running           192.168.64.15    Ubuntu 20.04 LTS 
swarm3                  Running           192.168.64.16    Ubuntu 20.04 LTS  $ multipass exec swarm1 -- docker node ls 
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION xki1yma9q91401zv0i0yksrht *   swarm1              Ready               Active              Leader              19.03.9 wgcj54zs2u3k9x1th3s20s1lm     swarm2              Ready               Active              Reachable           19.03.9 vk3c3etx53zq4hhzk7q6kgo3y     swarm3              Ready               Active                                  19.03.9
```

我们现在有三个节点，两个经理和一个工人。

# 向集群添加工作负载

在 Docker Swarm 中，我们创建了一个*服务*，它又包含*任务*，每个任务包含一个*容器*。容器是我们使用`docker run`命令创建的同一个容器，其余部分是为了让 Swarm 可以跨一组 Swarm 节点管理一组容器。

```
$ multipass exec swarm2 -- docker service create --name nginx --replicas 3 -p 80:80 nginx 
l0z5s0e1rpoy35gksnioikf5w 
overall progress: 3 out of 3 tasks 
1/3: running 
2/3: running 
3/3: running 
verify: Service converged
```

这里，我们请求在一个管理节点(`swarm2`)上创建一个名为`nginx`的服务。我们请求该服务的三个实例，它将使用来自 Docker Hub 的`nginx`图像。它也将从容器中暴露端口 80。

```
$ multipass exec swarm1 -- docker service ls 
ID NAME MODE REPLICAS IMAGE PORTS 
l0z5s0e1rpoy nginx replicated 3/3 nginx:latest *:80->80/tcp
```

为了证明群节点可以相互通信，让我们列出当前在集群上运行的服务，但是这些服务是从`swarm1`节点请求的。

```
$ multipass exec swarm1 -- docker service ps nginx 
ID NAME IMAGE NODE DESIRED STATE CURRENT STATE ERROR PORTS wcoiz9p6cmck nginx.1 nginx:latest swarm1 Running Running 5 minutes ago 
yud37ipf4lwb nginx.2 nginx:latest swarm2 Running Running 5 minutes ago 
wrze6s2csz62 nginx.3 nginx:latest swarm3 Running Running 4 minutes ago
```

我们可以获得`nginx`服务的状态，显示三个任务在哪里运行。也就是说，集群在节点之间分配任务，每个节点一个任务。

`nginx`图像显示了一个默认的 HTML 页面，因此我们可以在浏览器中访问位于`http://192.168.64.14/`、`http://192.168.64.15/`和`http://192.168.64.16/`的三个 Multipass 实例，以验证它们是否都工作正常。

不那么明显的是，Swarm 负责交叉连接服务，这样每个发布的服务在每个节点上都是可见的，即使该服务没有在该节点上运行。

# 探索跨群节点的服务的交叉连接

为了探索服务是如何跨群节点交叉连接的，让我们创建一个服务，看看它是如何工作的。

```
$ multipass exec swarm1 -- docker service create --name httpd --replicas 1 -p 90:80 httpd 
qnodrnbmpd5hy360fn99hgdg5 
overall progress: 1 out of 1 tasks 
1/1: running 
verify: Service converged $ multipass exec swarm1 -- docker service ls 
ID NAME MODE REPLICAS IMAGE PORTS 
qnodrnbmpd5h httpd replicated 1/1 httpd:latest *:90->80/tcp 
l0z5s0e1rpoy nginx replicated 3/3 nginx:latest *:80->80/tcp $ multipass exec swarm1 -- docker service ps httpd 
ID NAME IMAGE NODE DESIRED STATE CURRENT STATE ERROR PORTS 
yb6dv4ur8eq2 httpd.1 httpd:latest swarm1 Running Running 49 seconds ago
```

为了给 Nginx 的竞争以同等的时间，我们启动了 Apache HTTPD 的一个实例。蜂群把它分配给了`swarm1`节点。我们将其设置为在端口 90 可见。通过仅使用一个副本，我们确保了 HTTPD 服务仅位于一个节点上。

并且，访问`http://192.168.64.14:90/`、`http://192.168.64.15:90/`和`http://192.168.64.16:90/`确实显示 Apache 服务通过每个节点可见。因此，Docker Swarm 的安排使得服务在每个节点上都是可见的，即使它只部署在其中一个节点上。

# 修改已部署服务的状态

我们可以修改已经部署的服务的状态，然后群将发布修改。

```
$ multipass exec swarm1 -- docker service update --replicas 2 httpd httpd 
overall progress: 2 out of 2 tasks 
1/2: running 
2/2: running 
verify: Service converged $ multipass exec swarm1 -- docker service ps httpd 
ID NAME IMAGE NODE DESIRED STATE CURRENT STATE ERROR PORTS 
yb6dv4ur8eq2 httpd.1 httpd:latest swarm1 Running Running 4 minutes ago 
9uxc9p9wbvdu httpd.2 httpd:latest swarm2 Running Running 10 seconds ago
```

这增加了实例的数量，现在有两个`httpd`实例被分配给`swarm1`和`swarm2`。

# 探索部署到每个节点的容器

另一个命令是`docker container`，它有许多管理单个容器的命令。有了它，我们可以探索部署在每个节点上的容器。

```
$ multipass exec swarm1 -- docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b6fc2b1000e3        httpd:latest        "httpd-foreground"       7 minutes ago       Up 7 minutes        80/tcp              httpd.1.yb6dv4ur8eq2epuae97i34ij1
f950a9c2db96        nginx:latest        "nginx -g 'daemon of…"   19 minutes ago      Up 19 minutes       80/tcp              nginx.1.wcoiz9p6cmckvb8il9n7uwyvk

$ multipass exec swarm2 -- docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
649d821a4e78        httpd:latest        "httpd-foreground"       3 minutes ago       Up 3 minutes        80/tcp              httpd.2.9uxc9p9wbvdu8l9h1lnesqjhc
44f12c4b4c5f        nginx:latest        "nginx -g 'daemon of…"   20 minutes ago      Up 20 minutes       80/tcp              nginx.2.yud37ipf4lwb43idtgu2bfhs7

$ multipass exec swarm3 -- docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
be8bcd336ad1        nginx:latest        "nginx -g 'daemon of…"   20 minutes ago      Up 20 minutes       80/tcp              nginx.3.wrze6s2csz62cdoc2ozwb0g03
```

请注意，在名称列中，我们看到类似于`nginx.2.yud37ipf4lwb43idtgu2bfhs7`的名称。为了使容器名称在集群中是唯一的，Docker Swarm 将那个 *nonce* 添加到容器名称中。

另一件要注意的事情是，在容器处，HTTPD 容器暴露端口 80。从端口 90 到端口 80 的映射发生在更高的级别。

# 删除服务

要清理和删除服务，请执行以下操作:

```
$ multipass exec swarm2 -- docker service rm nginx
nginx

 multipass exec swarm2 -- docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
qnodrnbmpd5h        httpd               replicated          2/2                 httpd:latest        *:90->80/tcp

$ multipass exec swarm1 -- docker service rm httpd
httpd
$ multipass exec swarm2 -- docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
```

`service rm`命令启动关闭指定服务的过程。它通过向集群分发命令来关闭容器。

# 管理多路实例

您可以停止单个实例，如下所示:

```
$ multipass stop swarm3
$ multipass stop swarm2
$ multipass stop swarm1
```

或者删除它们:

```
$ multipass delete swarm3
$ multipass delete swarm2
$ multipass delete swarm1
$ multipass purge
```

不同之处在于，停止的实例可以重新启动，但删除的实例处于炼狱状态。`purge`命令删除与任何已删除实例相关的数据。

```
$ multipass start swarm3
$ multipass start swarm2
$ multipass start swarm1
```

如果您想重启一个节点，只需使用`start`命令。

要试验的是有运行实例的群，当你在其中一个实例上运行`stop`时会发生什么。例如，swarm 会根据需要重新部署容器。如果您已经停止了 Leader 实例，将会选出一个新的 Leader。

# 表演

这个实验是在一台 2012 款 MacBook Pro 上进行的，它有 16 GB 的主内存和一个 SSD 引导驱动器。也就是说，这是一台已经使用了八年的电脑，已经扩展到了极限。目前运行的是 Chrome(有多个顶层窗口和几百个打开的标签页)，Firefox 和 Safari 浏览器，加上几个 Visual Studio 代码窗口，以及几个其他应用程序。换句话说，这不是一台负载很轻的计算机。

但是同时运行三个多通道实例并不会导致系统明显变慢。

Multipass 团队声称这是一个极其轻量级(意味着低系统影响)的虚拟机管理器，支持成熟的 Ubuntu 实例。该实验表明“低系统影响”部分是正确的。虽然我启动的 Nginx 和 HTTPD 容器并没有对系统造成太大影响，但我确实有三台虚拟机，其中三个 Nginx 实例和两个 HTTPD 实例同时运行。在一台有八年历史的 MacBook Pro 上使用 VirtualBox 试试吧，它同时运行着许多其他进程。

我对 VirtualBox 的记忆是，它能迅速把台式电脑变成糖蜜。

# 摘要

Multipass 看起来是在开发人员的笔记本电脑上启动 Ubuntu 实例的一个非常有用的工具。它非常强大，一分钟左右你就能得到一个成熟的 Ubuntu 实例。据说 Multipass 支持其他操作系统是可能的，但似乎 Ubuntu 是目前唯一可用的版本。

Docker Swarm 是一款非常强大的 Docker orchestrator。学习 Swarm 是值得的，只要你对 Docker 编排有更好的理解，从而对其他 Docker 编排有更清晰的了解。Multipass 看起来是一个很好的平台，可以在这个平台上不用离开你的笔记本电脑就可以体验 Docker Swarm。

# 关于作者

[**大卫·赫伦**](https://techsparx.com/about.html) :大卫·赫伦是一名作家和软件工程师，专注于技术的明智使用。他对太阳能、风能和电动汽车等清洁能源技术特别感兴趣。David 在硅谷从事了近 30 年的软件工作，从电子邮件系统到视频流，再到 Java 编程语言，他已经出版了几本关于 Node.js 编程和电动汽车的书籍。

*原载于*[*https://techsparx.com*](https://techsparx.com/software-development/docker/swarm/multipass.html)*。*