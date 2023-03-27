# chroot、cgroups 和名称空间—概述

> 原文：<https://itnext.io/chroot-cgroups-and-namespaces-an-overview-37124d995e3d?source=collection_archive---------1----------------------->

![](img/3c05b91b2da21d1353a59fa9cf65eced.png)

照片由 [Aron Van de Pol](https://unsplash.com/photos/hXOGHaGCtdA?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/search/photos/linux?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

# 介绍

随着所有与 Docker、容器和虚拟化相关的讨论，程序员了解这些将在日常编程中帮助他们的技术变得越来越重要。我将尝试记录 Docker 类应用程序背后的技术。

# 根和色根

在类 Unix 操作系统中，根目录(/)是顶层目录。根文件系统位于根目录所在的同一磁盘分区上。所有其他文件系统都是在这个根文件系统之上挂载的。所有文件系统条目都从这个根目录分支出来。这是系统的实际根目录。

但是每个进程都有自己的根目录概念。默认情况下，它是实际的系统根，但是我们可以通过使用`chroot()`系统调用来改变它。我们可以有一个不同的根，这样我们就可以创建一个单独的运行环境，这样就可以更容易地运行和调试流程。或者也可以为该过程使用遗留的依赖项和库。

`**chroot**` **改变当前运行进程及其子进程的表观根目录。**

似乎通过使用`chroot()`分离一个进程，我们通过限制进程不能访问其环境之外的内容来确保安全性。但事实上，这并不完全正确。`chroot()`简单地修改进程及其子进程的路径名查找，将新的根路径添加到以`/`开头的任何名称之前。当前目录未被修改，相对路径可以引用新根目录之外的任何位置。

**所以，** `**chroot()**` **不提供安全的沙箱来测试一个软件。**

# cgroups-隔离和管理资源

控制组(cgroups)是一个 Linux 内核特性，它限制、隔离和测量一组进程的资源使用情况。可以设置内存、CPU、网络和 IO 的资源配额。在 Linux 2.6.24 中，这些是 Linux 内核的一部分。

尽管 Linux 在处理和共享进程间的可用资源方面表现出色，但有时我们希望更好地控制资源。我们希望为一组进程分配或保证一定数量的资源。我们用 cgroups 做这个。这隔离了应用程序/组的资源。

假设我们有一个想要隔离使用的应用程序。让我们称它为 A1。我们将创建一个控制组，并对其分配资源限制:比如 3GB 的内存限制和 70%的 CPU。然后，我们可以将必需的应用程序的进程 id 添加到组中，现在应用程序资源的使用得到了控制。虽然应用程序可能会超出正常情况下的限制，但在系统面临资源短缺的情况下，它会被限制回预设的限制。当我们处理在一台机器上运行的许多虚拟机时，这就更有意义了——为虚拟机建立一个 cgroup，并在发生资源争用时将它们单独限制在一个设定的限制范围内。

*   定义问题的解决方案
*   创建一个 cgroup 来处理分配
*   将应用程序添加到组中。
*   继续监视组(作为 cgroups 的一部分发生，我们不需要显式处理)

要安装 cgroups，

`sudo apt-get install cgroup-bin cgroup-lite cgroup-tools cgroupfs-mount libcgroup1`

我们可以看到创建了一个`/cgroup`目录:它被用作 cgroup 虚拟文件系统的挂载点。`etc/cgconfig.conf`文件给出了所有坐骑的预期信息。所有控制器都挂载到/cgroup，后跟控制器名称。要安装必要的控制器，运行`sudo service cgconfig restart`。接下来我们看到/cgroup 中的目录，每个目录都可以用来管理一个 cgroup 子系统。

## 创建 cgroups

现在我们看到控制器安装在/cgroup 上，让 cd 进入其中的任何目录，比如/cgroup/memory，并在其中创建一个子目录。

`mkdir mytest`

当我们`cd`进入这个子目录时，除了一个文件`release_agent`之外，它几乎与父目录相似。现在我们在`cgroup`下有了一个内存子系统，它的子节点`mytest`是 cgroup。让我们创建一个测试函数来测试我们的节流假设:

```
#include<iostream>#include <new>#include <cstdlib>int main(){int i=0;char* ptr =NULL;while(i<50){if ((ptr =(char*)malloc(1048576)) == NULL) {///1MB allocatedstd::cout << "Allocation fails at " << i << "MB\n";return 0;}std::cout << "Allocated "<< i+1 << "MB\n";i++;}std::cout << "Finished allocation";return 0;}
```

编译并运行上面的 C++代码，我们看到所有的 50 MB 都已分配，最后一行“完成分配”被打印出来。这没有分配任何内存配额。让我们把`cd`变成`cgroup/memory/mytest`。我们将使用两个文件分别将物理内存和交换内存的内存限制设置为 2MB:`mytest/memory.limit_in_bytes`和`mytest/memory.memsw.limit_in_bytes`。设置交换限制是为了实施一个硬限制，这样一旦达到 2MB 内存限制，它就不会开始消耗交换空间。

```
echo 2097152 > /cgroup/memory/mytest/memory.limit_in_bytes
echo 2097152 > /cgroup/memory/mytest/memory.memsw.limit_in_bytes
```

现在，我们可以使用 [cgexec](https://linux.die.net/man/1/cgexec) 命令运行我们的上述程序。

```
cgexec -g memory:mytesttest ./<binary_name>
```

这将运行内存测试组中的代码，当达到 2MB 分配限制时，进程将被终止。

像内存一样，我们可以为 CPU、磁盘 IO 或网络 IO 分组，并根据预定义的限制来调节应用程序。

# Linux 名称空间

Linux 进程形成了一个单一的层次结构，所有进程都以`init`为根。通常这个树中的特权进程可以跟踪或终止其他进程。Linux 名称空间使我们能够拥有许多带有自己的“子树”的进程层次结构，这样一个子树中的进程就不能访问甚至不知道另一个子树中的进程。

名称空间包装全局资源，使得在该名称空间中的进程看起来具有它们自己的所述资源的隔离实例。让我们以 PID 名称空间为例。如果不涉及命名空间，所有进程都从 PID 1(init)开始分层下降。如果我们创建一个 PID 名称空间，并在其中运行一个进程，那么第一个进程将成为该名称空间中的 PID 1。在本例中，我们包装了一个全局系统资源(进程 id)。创建命名空间的进程仍保留在父命名空间中，但使其子进程成为新进程树的根。

但这仅意味着新命名空间内的进程看不到父进程，但父进程命名空间可以看到子命名空间。新名称空间中的进程现在有两个 PID:一个用于新名称空间，一个用于全局名称空间。

Linux 内核现在使用 upid 结构而不是单一的 pid 值来跟踪进程的 pid。upid 结构告诉我们 pid 和该 pid 有效的名称空间。

```
struct upid {
	int nr;			/* moved from struct pid */
	struct pid_namespace *ns;	/* the namespace this value
						 * is visible in
						 */
	...
    };
    struct pid {
	atomic_t count;
	struct hlist_head tasks[PIDTYPE_MAX];
	struct rcu_head rcu;
	int level;		/* the number of upids */
	struct upid numbers[0];
    };
```

## 命名空间的类型

Linux 提供了以下名称空间:

*   cgroup:这将隔离 Cgroup 根目录( **CLONE_NEWCGROUP** )
*   IPC:隔离 System V IPC、POSIX 消息队列( **CLONE_NEWIPC** )
*   网络:隔离网络设备、端口等( **CLONE_NEWNET** )
*   挂载:隔离挂载点( **CLONE_NEWNS** )
*   PID:隔离的进程 id(**CLONE _ NEWPID**)
*   用户:隔离用户和组 id(**CLONE _ new User**)
*   UTS:隔离主机名和 NIS 域名( **CLONE_NEWUTS** )

作为名称空间管理的一部分，Linux 提供了以下 API:

*   ***【克隆()】***——普通旧克隆()创建新流程。如果我们将一个或多个 CLONE_NEW* 标志传递给 CLONE()，那么将为每个
    标志创建新的名称空间，并且子进程将成为这些
    名称空间的成员。
*   ***setns()*** -允许一个进程加入一个已有的名称空间。命名空间由引用一个`proc/[pid]/ns`文件的文件描述符指定。
*   unshare()-将调用进程移动到根据 CLONE_NEW*参数创建的新名称空间。可以指定多个这样的标志。

注意:对于 PID 命名空间，必须调用 clone()，因为只有在创建新进程时才能创建 clone()。克隆()产生的进程的 pid 为 1。对于 PID 命名空间，unshare()没有用。

## 网络命名空间

假设我们有了新的 PID 名称空间。位于这个新名称空间中的进程在端口 80 上监听传入的请求。这意味着整个系统中的所有其他进程**。被阻止监听它。这不是很有帮助的隔离。这就是网络名称空间的由来。**

**网络名称空间帮助内部进程看到不同的网络接口集——包括`lo`接口！但这只是故事的一半。当我们有新的网络名称空间时，我们必须设置跨越许多名称空间的虚拟网络接口，以及在全局名称空间中运行的路由过程，以处理流量并将其路由到正确的名称空间。**

**与上面的名称空间一样，其他名称空间隔离特定的全局资源，并将内部进程的访问限制在自己的沙箱中。**

# **结论**

**我们看到了 chroot、cgroups 和 namespaces 的简要概述，它们为 Linux 开发人员提供了将进程隔离到他们自己的“容器”中的方法。这些技术是现在无处不在的 Docker 或 Linux 容器的基础。我将试着在这篇文章之后介绍 Docker 更具体的内部情况。**

**参考资料:**

**[](https://wiki.archlinux.org/index.php/cgroups) [## cgroups-archi wiki

### cgroups(又名 control groups)是一个 Linux 内核特性，用于限制、监管和计算一组…

wiki.archlinux.org](https://wiki.archlinux.org/index.php/cgroups) [](https://sysadmincasts.com/episodes/14-introduction-to-linux-control-groups-cgroups) [## Linux 控制组(Cgroups)简介

### 在这一集里，我们将回顾控制组(cgroups ),它提供了一种机制来方便地管理和…

sysadmincasts.com](https://sysadmincasts.com/episodes/14-introduction-to-linux-control-groups-cgroups) [](http://man7.org/linux/man-pages/man7/namespaces.7.html) [## 名称空间(7) - Linux 手册页

### 名称空间将全局系统资源包装在一个抽象中，使其对名称空间内的进程可见…

man7.org](http://man7.org/linux/man-pages/man7/namespaces.7.html) [](https://lwn.net/Articles/259217/) [## 2.6.24 内核中的 PID 名称空间[LWN.net]

### 即将到来的 2.6.24 内核中的新特性之一将是 OpenVZ 团队开发的 PID 名称空间支持…

lwn.net](https://lwn.net/Articles/259217/)**