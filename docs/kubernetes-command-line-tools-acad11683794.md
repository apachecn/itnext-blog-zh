# Kubernetes 命令行工具

> 原文：<https://itnext.io/kubernetes-command-line-tools-acad11683794?source=collection_archive---------1----------------------->

![](img/90cc1bb74ffdb2d725ceb19eba091d01.png)

如果您经常与 Kubernetes 集群进行交互，您可能会发现自己对输入重复命令的时间感到沮丧。减少这种挫折感的一种方法是通过使用针对`kubectl`的 CLI 工具，Kubernetes 命令行界面。本文将重点介绍几种用于简化`kubectl`的使用并节省时间的工具。

将涵盖以下工具:

*   [外壳自动完成](https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion):自动完成`kubectl`
*   kubectx & kubens :在 Kubernetes 上下文和名称空间之间来回切换
*   [kube-ps1](https://github.com/jonmosco/kube-ps1) : Kubernetes 提示 bash 和 zsh:上下文/名称空间信息到您的 shell 提示
*   [kubectl 别名](https://github.com/ahmetb/kubectl-aliases):创建自己的别名与`kubectl`交互

**先决条件**:确保您已经安装了`kubectl`命令行实用程序，并且可以访问 Kubernetes 集群。关于安装`kubectl`的说明，请查看本文。按照这个[友好指南](http://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/oke-full/index.html)使用 Oracle Container Engine for Kubernetes(OKE)访问 Kubernetes 集群。

# Shell 自动完成:kubectl 的自动完成

如果您发现自己花了太多时间编写长的`kubectl`命令，那么配置 shell 自动完成功能是值得的。这类似于 bash、zsh 等中的 shell 补全。这个工具很有用，因为它可以用来自动完成所有基本的`kubectl`命令以及部署、服务等。从您的集群。Autocomplete 从您当前的上下文中提取信息，使您不必键入可能很长的部署名称。查看本文中关于如何为`kubectl`配置 shell 自动完成的说明。

要使用带有`kubectl`的 shell 自动补全功能，只需在编写命令时按下**标签**。例如，我们可以输入`g`，然后按下**标签**来自动完成`get`。

```
$ kubectl g [TAB]
$ kubectl get
```

当我们键入`s`时，您可能会期望按下 **tab 键**会自动完成服务，但是在这种情况下，以`s`开头的单词有很多选项。

```
$ kubectl get s [TAB]
secrets                                signalfxs.config.istio.io
serviceaccounts                        solarwindses.config.istio.io
servicecontrolreports.config.istio.io  stackdrivers.config.istio.io
servicecontrols.config.istio.io        statefulsets.apps
serviceentries.networking.istio.io     statsds.config.istio.io
servicerolebindings.rbac.istio.io      stdios.config.istio.io
serviceroles.rbac.istio.io             storageclasses.storage.k8s.io
services
```

如果您写出`service`，您可以 **tab 键**来显示您当前名称空间中所有可用的服务。

```
$ kubectl get service [TAB]
details      helloworld   kubernetes   productpage  ratings      reviews
```

要查看可用命令的完整列表，请键入`kubectl`后跟两个 **tab'** 。

```
$ kubectl [TAB][TAB]
annotate       certificate    create         explain        plugin         set
api-resources  cluster-info   delete         expose         port-forward   taint
api-versions   completion     describe       get            proxy          top
apply          config         diff           label          replace        uncordon
attach         convert        drain          logs           rollout        version
auth           cordon         edit           options        run            wait
autoscale      cp             exec           patch          scale
```

# kubectx & kubens:在 Kubernetes 上下文和名称空间之间来回切换

如果您发现自己在多个 Kubernetes 上下文和/或多个名称空间之间来回切换，使用工具来缩短这个过程会很有帮助。使用`kubectx`您可以轻松地列出、切换、重命名和删除上下文。`kubens`同样允许您列出名称空间并在名称空间之间切换。这两种工具也支持**标签**完成。

关于如何配置`kubectx`和`kubens`的说明，请查看本文。

此外，您可以通过添加交互式菜单，如`fzf`，使`kubectx`和`kubens`更加方便。按照这些步骤安装`fzf`。`fzf`是一个命令行模糊查找器，它为您提供了一个上下文或名称空间的交互式菜单供您选择。

# 使用 kubectx

要列出上下文:

```
$ kubectx
context-ctdazdbmi3t
docker-for-desktop
```

要切换上下文:

```
$ kubectx docker-for-desktop
Switched to context "docker-for-desktop".
```

要返回到上一个上下文:

```
$ kubectx -
Switched to context "context-ctdazdbmi3t".
```

要重命名上下文:

```
$ kubectx oracle=context-ctdazdbmi3t
Context "context-ctdazdbmi3t" renamed to "oracle".
```

当`kubectx`与`fzf`一起使用时，同样的命令也适用。唯一的区别是交互式菜单的外观。

```
$ kubectx> docker-for-desktop
  context-ctdazdbmi3t
  2/2
```

# 使用 kubens

要列出名称空间:

```
$ kubens
default
istio-system
kube-public
kube-system
sample
spinnaker
```

要切换名称空间:

```
$ kubens spinnaker
Context "context-ctdazdbmi3t" modified.
Active namespace is "spinnaker".
```

要返回到上一个名称空间:

```
$ kubens - 
Context "context-ctdazdbmi3t" modified.
Active namespace is "default"
```

# 将 kubectx 与 fzf 一起使用

当`kubens`与`fzf`一起使用时，同样的命令也适用。与`kubectx`一样，唯一的区别是交互式菜单的外观。

```
$ kubens > default
  istio-system
  kube-public
  kube-system
  sample
  spinnaker
  6/6
```

# kube-ps1: Kubernetes 对 bash 和 zsh 的提示

kube-ps1 将当前的 Kubernetes 上下文和名称空间添加到提示字符串中。这个工具让您不必在每次需要检查上下文或确定当前名称空间时都键入`kubectl config current-context`。这是一个简单的改变，可以使与集群的交互变得更加容易。

另外，如果您厌倦了在提示字符串中看到您的上下文和名称空间，您可以简单地在 shell 会话中使用`kubeoff`关闭 kube-ps1。它可以用`kubeon`重新打开。要使这一更改在 shell 会话中全局生效，请在命令末尾添加`-g`标志。

查看[这篇文章](https://github.com/jonmosco/kube-ps1)了解如何配置 kube-ps1 的说明。

安装完成后，您的 shell 应该看起来像这样:

```
(⎈ |[context]:[namespace]) $ 
```

例如，我的提示字符串如下所示:

```
(⎈ |context-ctdazdbmi3t:default) $
```

# kubectl 别名

bash、zsh 等的 Shell 别名。可以用来缩短任何和所有的 shell 命令，而不仅仅是那些与`kubectl`相关的命令。在这种情况下，您的环境中不需要额外安装任何东西。将与`kubectl`相关的别名分离到它们自己的文件中，并在您的 shell 运行时配置(例如 bashrc)中找到它，这是一个很好的实践。要创建自己的一组`kubectl`别名，首先在$HOME 目录中创建一个文件，例如一个名为。kube _ 别名:

```
$ vi .kube_alias
```

首先添加一个简单的别名，比如`alias k=’kubectl’`，然后保存文件。编辑您的。巴沙尔或者。zshrc 文件，包含以下内容:

```
[ -f ~/.kubectl_alias ] && source ~/.kubectl_alias
```

这将告诉您的 shell 引用您新创建的别名文件。下次您在 shell 中输入`k`时，它将被视为您输入了`kubectl`。

可以创建别名来简化任意数量的命令。例如，如果您发现自己经常键入`kubectl get pods --all-namespaces`来获得集群中每个名称空间中运行的 pods 列表，那么您可以编写一个别名，比如`kgpall`来缩短命令。在你的。kube_alias 文件，这个别名应该是这样的:`alias kpgall=’kubectl get pod --all-namespaces’`。

[这个页面](https://github.com/ahmetb/kubectl-aliases)有很多有用的别名，可以和 Kubernetes 一起使用。即使你选择不使用这个列表，它也可以给你一些有用的想法，让你自己写别名。

# 结论

读完这篇文章后，我希望您对命令行工具有更好的理解，这些工具可以让您更容易地与 Kubernetes 交互。我还希望您能受到启发，探索可用工具的生态系统，甚至创建自己的工具。