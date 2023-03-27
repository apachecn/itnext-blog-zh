# OpenShift 中的最低特权容器—使用日志代理的示例

> 原文：<https://itnext.io/minimally-privileged-containers-in-openshift-an-example-using-a-logging-agent-872e21fdce86?source=collection_archive---------3----------------------->

我和一个同事最近有一个在 OpenShift 集群上运行日志收集代理的任务，一个关键要求是以尽可能小的访问权限运行代理。具体来说，我们需要运行这个代理:

*   在非特权容器中(即没有在容器的`securityContext`规范中设置`privileged: true`)
*   作为非超级用户(作为 UID 不是 0 的用户)

但是尽管以最小特权运行，这个代理仍然必须能够收集来自`hostMount`的日志——意思是来自底层工作节点上的文件系统。

这不是 Kubernetes 世界中的新问题或用例。例如，LogDNA 提供了[指令](https://github.com/logdna/logdna-agent-v2/blob/master/docs/OPENSHIFT.md#run-as-non-root)，用于在“常规”Kubernetes 和 Openshift… *上以非根用户身份运行他们的代理，但是*他们仍然指定他们的代理必须在特权容器中运行。那对我们来说不是一个可行的选择。

# 谁是负责人？

这篇文章的副标题可能是“两个…不，三个…不，四个…访问控制机制的故事”Kubernetes 的用户可能已经非常熟悉常规的 Unix 文件系统访问控制机制。更高级的用户/开发人员可能也熟悉通过系统[功能](https://man7.org/linux/man-pages/man7/capabilities.7.html)的自主访问控制的 Linux 实现，它基本上将 root 的超级权限细分为更小、更窄的定制包，可以为需要提升访问权限的进程启用。

当然，Kubernetes 本身引入了容器的 [securityContext](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) 和 [serviceAccount](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) 等构造。

然而，更复杂的是 OpenShift。OpenShift 要求 worker 节点启用 [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux) (安全增强的 Linux)，这在现有 DAC 机制之上强加了一个强制访问控制(MAC)层(*nix 权限+系统功能)。

然后是 OpenShift 的`SecurityContextConstraints (SCC)`，它是一个 Kubernetes 资源，定义了 pod 准入控制器在启动 pod 时使用的约束。pod 中每个容器的`securityContext`与 SCC 进行比较——违反将导致 pod 不被允许启动。此外，SCC 实际上可以强制对容器进行某些更改，比如强制添加或删除特定的 Linux 系统功能。

对于试图确定如何最好地确保他/她的应用程序以所需的访问权限运行的应用程序开发人员来说，所有这些加起来可能是一幅相当混乱的画面。

# 将碎片拼在一起

让多种不同的机制协同工作有点像撬锁——你必须让所有的制动栓和内部零件对齐，才能让锁打开，但如果哪怕只有一个小零件错位，整个机制都不会正常工作。

![](img/f86838346d0251288c9936b50dfd4291.png)

想象一下，打开一扇由 Kubernetes、Linux 和 SELinux 保护的门…

在接下来的章节中，我们将逐一介绍这两个用例，重点是如何确保容器有足够的访问权限，而不需要过分宽松的配置。

最后一点很关键——通过以 root 用户身份运行某个进程或积极开放目录权限来“让事情运转起来”是很诱人的。但是我们希望*以应用程序工作所需的绝对最低访问级别运行我们的应用程序*。在一般实践中，这是一个很好的规则，我们都应该避免只是按下永远诱人的简单按钮的诱惑。毕竟，懒惰是恶意行为者成功攻击的基础。

# 以非 root 用户身份运行容器

第一个挑战是使我们的应用程序——在我们的例子中是一个日志收集代理——能够在不以 root 用户身份运行的情况下访问和读取日志文件。这包括两个部分:作为非 root UID(除 0 以外的 UID)运行，并启用所需的访问级别。

在标准的 Kubernetes 环境中，将容器作为非根 UID 运行非常简单。您可以简单地在容器的`securityContext`中指定 UID:

```
securityContext:
  runAsUser: 10000
  runAsGroup: 10000
```

在这里，我已经指定了我的容器将在 UID 为`10000`和 GID 为相同值的情况下运行。不要求这些条目映射到底层操作系统已知的用户(例如，不要求相应的条目存在于`/etc/passwd`或`/etc/group`)。

但是这个 userid 能够读取日志文件(或者访问应用程序需要访问的任何其他主机文件)吗？例如，在我们的 worker 主机上，logs 目录是用权限`770`设置的:

```
drwxr-x---. 16 root root 4096 Sep  9 14:09 /var/log
```

那么如何让一个容器化的应用程序访问这个目录的文件或者子目录呢？一个选项是`initContainer`(在常规应用程序容器之前，每个 pod 实例运行一次)，以 root 用户身份运行，更改该目录的权限。但这基本上绕过了我们试图遵守的限制。当然，`initContainer`只运行一次，不会成为攻击目标(假设使用`initContainer`运行某种一次性设置代码的正常做法，在我们的例子中是一个基本的 shell 脚本)，但它仍然…不一致，相当难看。而如果`/var/log`下有子目录呢？在这种情况下，我基本上是在劫持构建和配置我们的工人映像的团队的决策，这并不是我真正想要做的。

幸运的是，Linux 提供了系统功能，我在上面提到过。我不会在这里讨论所有的细节，但是对于可能不熟悉它们的应用程序开发人员来说，花时间阅读它们并理解基础知识是值得的—从这里的手册页[开始](https://man7.org/linux/man-pages/man7/capabilities.7.html)。这样做的目的是将 root 的权限细分为内聚的、粒度更细的包，每个包都可以针对给定的应用程序或工作负载单独启用/禁用。在我们的例子中，我们想要的功能是`CAP_DAC_READ_SEARCH`——这个功能在启用时，允许进程在试图读取文件或列出/访问目录时忽略权限。(在您应该说“嘿，现在您的代理可以读取任何内容了”…在我们稍后讨论 SELinux 之前，请保持这种想法。)

为我们的用例启用功能的过程有两个部分:第一，我们必须将该功能标记为允许二进制可执行文件使用，第二，我们必须在运行二进制文件时明确请求该功能。首先，我们将这个命令添加到我们的`Dockerfile`:

```
RUN setcap "cap_dac_read_search+eip" /usr/local/bin/logging-agent
```

*注意:解释* `*+eip*` *后缀的重要性超出了本文的范围——这个后缀与用于计算任何给定流程允许/启用什么功能的算法有关，这在我前面提到的手册页中有描述，但是您可以在这篇博文* *中找到一个非常好的、更实用的解释。*

当我们运行我们的容器时，我们通过更新我们的`securityContext`请求将功能添加到流程中，如下所示:

```
securityContext:
  runAsUser: 10000
  runAsGroup: 10000
  **capabilities:
    add:
    - DAC_READ_SEARCH**
```

## 那么，我们取得了什么成就？

至此，我们已经使我们的容器能够作为非根用户运行，并且仍然可以访问它需要在 Kubernetes worker 主机上读取的任何日志文件和/或目录。但是我们还有两个问题没有解决。首先，我们违反了一个关键原则，特别是只使用所需的最低访问权限的原则。幸运的是，SELinux 为我们提供了一个解决方案。不幸的是，这也使事情变得复杂。另一个是，不幸的是，这在 OpenShift 上实际上是行不通的——open shift 添加了额外的限制，这些限制有助于(通过限制工作负载在没有额外配置的情况下可以做什么)和阻碍(出于完全相同的原因)。所以现在我们必须转向这些额外的层。

# OpenShift 和 SecurityContextConstraints

OpenShift 引入了一种特殊的资源，称为`SecurityContextConstraints`(以下简称 SCC)，正是这样——对容器的`securityContext`所允许的一组约束。它实际上比`securityContext`的范围更广 SCC 可以限制允许 pod 访问的卷的类型、pod 可以使用的主机功能，甚至可以限制 pod 的容器可以使用的 Linux 系统功能。

考虑 SCC 资源的一种方式是将其与 Linux 系统功能相比较——就像 SYSCAP 机制被用来更好地控制超级用户权限一样，SCC 对属于`privileged`容器的大部分内容提供了更好的控制。SCC 建立在 [Kubernetes 安全策略](https://kubernetes.io/docs/concepts/security/pod-security-standards/)的基础上，pod 准入控制器通常使用这些策略来防止 pod 请求(和接收)过多的特权。一些关于 SCC 的好读物通常可以在 OpenShift 的博客文章和文档中找到。实际上想要使用他们自己的 SCC 的开发人员/管理员应该通读 [API 文档](https://docs.openshift.com/container-platform/4.7/rest_api/security_apis/securitycontextconstraints-security-openshift-io-v1.html)，并且可能查看`oc explain securitycontextconstraints`的输出。

## 配置我们自己的安全上下文约束

到现在为止，我将把它作为一条信条，我们都同意特权容器(任何在它的`securityContext`中带有`privileged: true`的容器)应该不惜任何代价避免。特权模式使容器能够利用(在某些情况下，操纵)几乎任何底层系统功能—网络、存储等。—这给系统安全性和完整性带来了巨大的风险。

但是我们确实需要比常规应用程序工作负载更多的访问——我们需要能够从工作主机的文件系统中读取日志文件——所以诀窍是如何在不将我们的容器置于`privileged`模式的情况下实现这一点？在我们的例子中，我们需要配置`SecurityContextConstraints`来给我们这个访问权限，但仅此而已。

OpenShift 目前附带九(9)种预配置的 SCC 类型。您不应该修改它们，因为它们中的许多已经被 OpenShift 系统 pods 使用。相反，找到看起来最接近你想要的那个，复制一份，然后修改它:

```
$ oc get scc hostmount-anyuid -o yaml > myscc.yaml
```

在这里，我们选择了`hostmount-anyuid` SCC 作为最有可能的匹配——它允许装载主机卷，以及作为任何用户 id 运行。看起来很合适，对吧？嗯……不完全是。这个 SCC 既过于宽松(它允许容器请求我们不希望我们的代理拥有的系统功能)，又不够宽松(它不允许我们迫切需要的功能)。该 SCC 包括以下几行:

```
allowedCapabilities: null
requiredDropCapabilities:
- MKNOD
```

第一行表示应用此 SCC 的任何 pod 都只能使用默认情况下允许的那些系统功能。您可以在 [Docker 文档](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)中找到这些功能的列表，重要的是，该列表*不*包括`DAC_READ_SEARCH`。所以首先，我们需要将它添加到列表`allowedCapabilities`(这是一个数组变量):

```
allowedCapabilities:
- DAC_READ_SEARCH
```

接下来，注意在`requiredDropCapabilities`下面列出的唯一功能是`MKNOD`——这里列出的功能会自动从 pod/container 中删除，即使它们在默认情况下是允许的，即使容器请求它们。因此，您可以使用这个变量来指定由特定 SCC 管理的 pod 应该*永远不具备*的能力。您想要禁止的内容可能因使用情形而异，但是在我们的情况下，我们决定将其修改为:

```
requiredDropCapabilities:
- MKNOD
**- FSETID
- KILL
- NET_BIND_SERVICE
- NET_RAW**
```

一些与网络相关的功能实际上是多余的，因为我们的 SCC 已经在其他地方指定:

```
allowHostIPC: false
allowHostNetwork: false
allowHostPorts: false
```

但格外小心也无妨。主要的收获是，即使是在 SCC 中，也有可能指定矛盾的设置，所以在设计时必须小心。我们现在需要添加的最后一件事是一个设置，以确保我们的 pod 可以实际使用这个 SCC。

## 为 pod 选择 SCC

OpenShift 决定哪个 SCC 与 pod 一起使用的方式看起来有点复杂，但是对于我们的目的来说，它并没有那么复杂。每个 SCC 都被授权给一个或多个用户和/或`serviceaccounts`。当您通过联系 API(例如使用`kubectl`)来部署 pod 时，您是以特定用户的身份这样做的。除非您在 pod 规范中指定了`serviceaccount`，否则它将在有问题的名称空间的默认`serviceaccount`下运行。用于请求 pod 和`serviceaccount`的用户将分别被授权使用一些 SCC 集合(可以是空集，在这种情况下，pod 将被完全禁止)。准入控制器将构建这两个集合的并集，然后确定是否至少有一个来自该并集的 SCC 允许 pod 运行。如果有一个以上的 SCC，则控制器将按优先级(一个值，默认为`null`(有效 0)，可在 SCC 定义中指定，较高的值优先)对有效 SCC 进行排序。当不止一个 SCC 具有相同的优先级时，它们将从最严格到最不严格排序，然后根据需要按名称排序。我们大多数人应该这样想:“使用最严格的 SCC，它仍然允许 pod 中的所有容器运行。”

*注意:使用* `*oc*` *部署 pod 时要小心——您通常以集群管理员的身份执行此操作，因此具有更高的权限——除其他外，它会自动导致准入控制器在候选列表中包含限制较少的 SCC。确保使用您的 CI 链*所使用的用户来测试您的 pod 是否能够正确运行。

我们在 pod 规范中指定了`serviceaccount`，因此我们可以通过在 SCC yaml 的`users`部分列出它来授权该帐户使用我们的 SCC:

```
users:
- system:serviceaccount:logger-agent:agent
```

这保证了当我们试图运行我们的 pod 时，准入控制器将至少找到这个 SCC，从而允许我们的 pod 运行。

# SELinux

所以在我们的情况下，我们认为我们已经做了足够的工作来让我们的代理工作。我们已经为代理提供了所需的 SYSCAP，以允许它遍历日志目录，并且我们已经指定了一个 SCC，它将允许我们的 pod 以所需的访问级别运行，但仅此而已。

剧透提醒:没那么多。我们没有考虑到这样一个事实，即 OpenShift 需要启用 SELinux，并且 SELinux 对进程与文件交互的能力施加了另一层限制。我们能够通过登录到 worker 节点并使用`ausearch`查看主机审计日志，寻找`AVC`消息(在我们的例子中，这表示当进程试图读取文件时访问被拒绝)来解决这个问题。

SELinux 将进程分类为它所谓的*域*，将文件/目录分类为*类型*。域和类型的命名通常采用`some_label_t`的形式——常量是后缀`_t`(包括域)。您可以通过在 worker 主机上运行`ps -Z`来查看进程的域:

```
sh# ps -Z -C crio
LABEL                              PID TTY          TIME CMD
system_u:system_r:container_runtime_t:s0 97341 ? 41-16:00:20 crio
```

`crio`进程是一个容器，因此默认情况下运行在一个域`container_runtime_t`中。除非另外指定，否则任何容器都将使用该域运行。我们想要读取的文件呢——例如`/var/log`目录？

```
sh# ls -Z -d /var/log
drwxr-xr-x. root root system_u:object_r:var_log_t:s0   /var/log
```

我们在这里可以看到,`/var/log`目录有一个类型`var_log_t`，事实上它下面的大多数文件和目录都是这样。对我们来说，最重要的文件是那些在`/var/log/containers`下的文件——这些文件是实际日志文件的符号链接，它们本身在`/var/log/pods/$PODNAME`下。因为 Kubernetes 以这种方式记录日志，单个代理可以在新文件出现在`/var/log/containers`下时获取它们，而不必检测新的 pod 子目录。不幸的是，默认的 SELinux 安全策略不允许域`container_runtime_t`下的进程访问主机文件系统上的任何东西(当然也不允许`var_log_t`对象)。碰巧的是，SELinux 提供了另一个名为`container_logreader_t`的域。该域可以访问类型为`container_log_t`的文件(类型为`/var/log/pods/$PODNAME`下的实际日志文件)。为了强制 pod 容器在特定的域中运行，我们在 SCC 对象中指定它(来回的交互可能感觉很复杂):

```
seLinuxContext:
  type: MustRunAs
  seLinuxOptions:
    type: container_logreader_t
```

所以我们用上面的内容更新了我们的 SCC，并重新创建了我们的代理 pod。它起作用了……算是吧。嗯，一点也不。我们所缺少的是，尽管这些文件属于`container_logreader_t`域可以访问的类型，但是*到这些文件的符号链接属于不同的类型，我们的域* ***无法读取*** *。*事实证明，我们实际上必须更深入地了解 SELinux 访问控制机制，并配置一些自定义规则来实现这一点，我们最终做到了。关于配置 SELinux 授权规则的详细讨论超出了本文的范围(这足以保证有一篇自己的文章，我的同事正在写那篇文章——一旦他写好了，我会在这里链接到它)。

## 重叠机制

您可能还记得，启用`DAC_READ_SEARCH`系统功能允许进程在读取文件/目录时忽略文件权限。这似乎是一件坏事，对不对？这似乎允许一个进程在文件系统中漫游，不受约束，能够访问它想要的任何文件。但是，在我们的例子中，它允许我们避免修改文件权限和/或所有权，这是一件好事。但是……这是一个很大的权限。

这就是 SELinux 出手相救的地方。在`container_logreader_t`域下运行(使用我们对 SELinux 授权策略的更新)*只有*允许我们访问`var_log_t`和`container_log_t`类型的对象(文件/目录)——而且这些只能在从`/var/log`开始的树中找到。因此，SELinux 和 Linux 系统功能的结合允许我们读取该目录下的文件，而不必担心和/或更改它们的权限/所有权。这就是我们想要的。它还防止我们的进程读取该目录之外的任何内容——这也是我们想要的。

# 外卖食品

您应该从所有这些中得出的要点是，配置一个安全运行的 pod，同时保持其功能是一个您需要整体处理的主题。花时间阅读文档，熟悉文档，然后花时间试验 SCC，看看什么可行，什么不可行。如果您需要从底层主机访问文件，那么您也需要进入 SELinux。但是总是花时间停下来问自己，“我真正想要完成的是什么？”OpenShift 和 SELinux 为 Kubernetes 添加了一些非常强大的功能，可以帮助您实现更加健壮的安全状态。但是，如果您不花时间预先计划您需要做什么，这可能会非常复杂和令人沮丧，部署者可能会倾向于恢复坏习惯，如以 root 和/或特权模式运行容器，即使有更好的替代方案。