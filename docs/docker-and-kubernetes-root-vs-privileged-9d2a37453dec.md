# Docker 和 Kubernetes —根用户与特权用户

> 原文：<https://itnext.io/docker-and-kubernetes-root-vs-privileged-9d2a37453dec?source=collection_archive---------0----------------------->

我们中的大多数人熟悉 Unix 系统，如 macOS 或 Linux，习惯于通过使用`sudo`随意提升我们的特权给`root`用户。这通常发生在调试开发工具或试图编辑受保护目录中的文件时。对于很多人(包括我)来说，在第一次尝试后尝试一个不成功的命令就像`sudo`一样几乎是默认的行为。).

![](img/909b26ac06cebed5ea9297c7ea863116.png)

理解 Docker 安全性的基础是理解容器实际上是什么(提示:不是 VM 的！)

Docker 提供了一个看似相似的`--privileged`标志，实际上与普通的`sudo`用法有很大的不同，这可能会让你的应用程序面临不必要的风险。我将向您展示这与以`root`身份运行有多大的不同(以及如何避免以 root 身份运行！)以及特权的实际含义。

# **以根用户身份运行**

Docker 允许隔离其主机操作系统上的进程、功能和文件系统，出于实际原因，大多数容器实际上默认作为`root`运行。举例来说，我将在撰写本文时选取 DockerHub 上最受欢迎的三张图片，以防您好奇，寻找指定的用户以及用户 ID:

[**Postgres:**](https://hub.docker.com/_/postgres)

```
$ docker run -it postgres
# whoami
root
# id -u
0
```

[](https://hub.docker.com/_/couchbase)

```
$ docker run -it couchbase sh
# whoami
root
# id -u
0
```

**[**高山:**](https://hub.docker.com/_/alpine)**

```
$ docker run -it alpine sh
# whoami
root
# id -u
0
```

**如您所见，默认情况下，大多数映像都以 root 用户身份运行。这通常允许更容易的调试，特别是如果你打算`exec`进入容器。虽然`root`用户附带了[非常有限的 Linux 功能](https://docs.docker.com/engine/security/security/#linux-kernel-capabilities)，但是最好避免以 root 身份运行。**

# ****避免以 Root 身份运行****

**虽然在容器内部运行`root`实际上很正常，但是如果你试图强化你的容器，还是应该避免。首先，它违反了最小特权原则。其次，从技术上讲，容器将是运行 Docker 命令的同一个用户名称空间的一部分，如果容器能够转义，它将可以访问相同的资源，如卷和套接字。**

**有两种方法可以避免以 root 用户身份运行:**

**通过调整[docker 文件以使用特定用户](https://docs.docker.com/engine/reference/builder/#user):**

```
// DockerfileFROM microsoft/windowsservercore
# Create Windows user in the container
RUN net user /add patrick
# Set it for subsequent commands
USER patrick
```

**或者[在运行时覆盖用户 ID:](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)**

```
$ docker run -it --user 4000 postgres sh
# whoami
whoami: cannot find name for user ID 4000
# id -u
4000
```

# ****特权阶层怎么样？****

**`—-privileged`标志将我们前面看到的用户 ID 0 直接映射到主机的用户 ID 0，并允许它自由访问它选择的任何系统调用。你看，在正常操作中，即使容器内有`root`，Docker 也会限制容器的 [Linux 功能](https://man7.org/linux/man-pages/man7/capabilities.7.html)。例如，这种限制包括像 **CAP_AUDIT_WRITE** 这样的东西，它允许覆盖内核的审计日志——这是您的容器化工作负载极不可能需要的能力。实际上，特权应该只在你真正需要它的特定设置中使用，简而言之，它让容器访问主机(作为 root)可以做的几乎所有事情。本质上，这是一张逃脱容器所包含的文件系统、进程、套接字和其他包含项目的免费通行证。它有特定的用例，比如 Docker 中的 [Docker，其他 CI/CD 工具需求(Docker 容器内部需要 Docker 守护进程)，以及需要极端联网的地方。即便如此，还是有办法避免它——例如，GitLab 提出了一种由谷歌容器工具创建的名为](https://www.docker.com/blog/docker-can-now-run-within-docker/#:~:text=One%20of%20the%20(many!),kernel%20features%20and%20device%20access.) [Kaniko](https://docs.gitlab.com/ee/ci/docker/using_kaniko.html) 的特权 pod 的替代方案。此外， [NestyBox](https://www.nestybox.com/) 产品为用户提供了[安全高效的 Docker 体验。](https://medium.com/@ctalledo/secure-docker-in-docker-with-nestybox-529c5c419582)**

**让我们看一个使用 Ubuntu 映像的例子(在 VM 内部，所以我们不能破坏任何东西):**

**没有特权:**

```
$ docker run -it ubuntu sh
# whoami
root ***# Notice here, we are still root!***
# id -u
0
# hostname
382f1c400bd
# sysctl kernel.hostname=Attacker
sysctl: setting key "kernel.hostname": Read-only file system  ***# Yet we can't do this***
```

**拥有特权:**

```
$ docker run -it --privileged ubuntu sh
# whoami
root. ***# Root again***
# id -u
0
# hostname
86c62e9bba5e
# sysctl kernel.hostname=Attacker
kernel.hostname = Attacker ***# Except now we are privileged***
# hostname
Attacker 
```

**Kubernetes 通过[安全上下文](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)提供相同的功能:**

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    **securityContext:
      privileged: true**
```

**此外，Kubernetes 包含一个名为 [PodSecurityPolicies](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) 的强制机制，它是一个准入控制器(也就是 Kubernetes 在允许容器进入集群之前检查的东西)。一个强烈推荐的政策是不允许特权舱:**

```
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: example
spec:
  **privileged: false ** *# Don't allow privileged pods!*
```

# ****总结****

**希望在本文结束时，您对`root`和`--privileged`标志有了更多的了解，以及它们与“主机”操作系统的关系，并且无论您是试图加强容器的安全性还是调试某个问题，您都知道足够多的信息来保护您的应用程序的安全。防御性安全完全是关于深度防御(像洋葱一样的层)和减少您的攻击面——通过不以 root 身份运行、不以特权身份运行，以及添加 SecurityContext 和 PodSecurityPolicies 是实现更高容器安全性的四个重要层。如果你喜欢阅读这类东西，请一定要关注我。如果你想问任何问题、评论或其他任何事情，请给我发电子邮件到 bryant.hagadorn@gmail.com。谢谢！同样，请在推特上关注我:[https://twitter.com/BryantHagadorn](https://twitter.com/BryantHagadorn)**