# 了解如何使用 ConfigMap 配置 Kubernetes 应用程序

> 原文：<https://itnext.io/learn-how-to-configure-your-kubernetes-apps-using-the-configmap-object-d8f30f99abeb?source=collection_archive---------0----------------------->

“配置与代码分离”是 [12 因素应用](https://12factor.net/config)的原则之一。我们将可以改变的东西具体化，这反过来有助于保持我们的应用程序的可移植性。这在 Kubernetes 世界中是至关重要的，在那里我们的应用程序被打包成 Docker 映像。Kubernetes `ConfigMap`允许我们从代码中抽象出配置，最终抽象出 Docker 映像。

这篇博文将为 Kubernetes 中可用的应用配置相关选项提供实践指南。

和往常一样，代码在 GitHub 上[可用。所以让我们开始吧…](https://github.com/abhirockzz/kubernetes-in-a-nutshell)

![](img/8f22ecbedd5cd81331c4c760fd6fdc58.png)

要在 Kubernetes 中配置您的应用程序，您可以使用:

*   好的旧环境变量
*   `ConfigMap`
*   `Secret`——这将在后续的博客文章中涉及

首先，您需要一个 Kubernetes 集群。这可能是一个简单的使用`[minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)`、`[Docker for Mac](https://blog.docker.com/2018/01/docker-mac-kubernetes/)`等的单节点本地集群。或者来自[Azure](https://docs.microsoft.com/azure/aks/?WT.mc_id=medium-blog-abhishgu)、 [Google](https://cloud.google.com/kubernetes-engine/) 、 [AWS](https://aws.amazon.com/eks/) 等的托管 Kubernetes 服务。要访问您的 Kubernetes 集群，您需要`[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)`，它很容易安装。

例如，要为 Mac 安装`kubectl`,您只需

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl && \
chmod +x ./kubectl && \
sudo mv ./kubectl /usr/local/bin/kubectl
```

# 使用环境变量进行配置

让我们从一个简单的例子开始，看看如何通过在我们的`Pod`规范中直接指定环境变量来使用它们。

注意我们如何在`spec.containers.env` — `ENVVAR1`和`ENVVAR2`中分别用值`value1`和`value2`定义两个变量。

让我们从使用上面指定的 YAML 创建`Pod`开始。

> `*Pod*` *只是一个库伯内特的资源或对象。YAML 文件是描述其期望状态以及一些基本信息的东西——它也被称为*`*manifest*`*`*spec*`*(规范的简写)或* `*definition*` *。**

*使用`[kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply)`命令向 Kubernetes 提交`Pod`信息。*

> **为了简单起见，YAML 文件直接从*[*GitHub repo*](https://github.com/abhirockzz/kubernetes-in-a-nutshell)*中引用，但是你也可以把文件下载到你的本地机器上，以同样的方式使用它。**

```
*$ kubectl apply -f   https://raw.githubusercontent.com/abhirockzz/kubernetes-in-a-nutshell/master/configuration/kin-config-envvar-in-pod.yamlpod/pod1 created*
```

*为了检查环境变量，我们需要使用`kubectl exec`在 Pod 的“内部”执行一个命令——您应该看到在`Pod`定义中播种的那些变量。*

*这里，我们使用了`grep`来过滤我们感兴趣的变量*

```
*$ kubectl exec pod1 -it -- env | grep ENVVARENVVAR1=value1
ENVVAR2=value2*
```

> ****什么是*** `***kubectl exec***` ***？*** *简单来说，它允许你在一个* `*Pod*` *内的特定容器中执行一个命令。在这种情况下，我们的* `*Pod*` *有一个单独的容器，所以我们不需要指定一个**

*好了，有了这个概念，我们可以探索一下。*

# *使用`ConfigMap`*

*它的工作方式是在一个`ConfigMap`对象中定义您的配置，然后在一个`Pod`(或`Deployment`)中引用该对象。*

*让我们来看看创建`ConfigMap`的技巧*

## *使用清单文件*

*可以在定义的`data`部分创建一个`ConfigMap`以及作为键值对存储的配置数据。*

*在上面的清单中:*

*   *名为`simpleconfig`的`ConfigMap`包含两个(键值)数据— `hello=world`和`foo=bar`*
*   *`simpleconfig`被一个`Pod` ( `pod2`)引用；键`hello`和`foo`分别作为环境变量`HELLO_ENV_VAR`和`FOO_ENV_VAR`使用。*

> **注意，我们已经把* `*Pod*` *和* `*ConfigMap*` *定义包含在同一个 YAML 中，中间用一个* `*---*`隔开*

*创建`ConfigMap`并确认环境变量已被植入*

```
*$ kubectl apply -f   https://raw.githubusercontent.com/abhirockzz/kubernetes-in-a-nutshell/master/configuration/kin-config-envvar-configmap.yamlconfigmap/config1 created
pod/pod2 created$ kubectl get configmap/config1NAME      DATA   AGE
config1   2      18s$ kubectl exec pod2 -it -- env | grep _ENV_FOO_ENV_VAR=bar
HELLO_ENV_VAR=world*
```

## *使用`envVar`的快捷方式*

*我们通过分别引用配置数据(`foo`和`hello`)来使用它们，但是有一种更简单的方法！我们可以在清单中使用`envFrom`来直接引用`ConfigMap`中的所有键值数据。*

> **当使用* `*ConfigMap*` *数据这种方式时，key 直接用作环境变量名。这就是为什么你需要遵循* [*命名约定*](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#configmap-v1-core) *即每个键必须由字母数字字符、“-”、“_”或“.”组成**

*就像之前一样，我们需要创建`Pod`和`ConfigMap`并确认环境变量的存在*

```
*$ kubectl apply -f   https://raw.githubusercontent.com/abhirockzz/kubernetes-in-a-nutshell/master/configuration/kin-config-envvar-with-envFrom.yamlconfigmap/config2 created
pod/pod3 created$ kubectl get configmap/config2NAME      DATA   AGE
config2   2      25s$ kubectl exec pod3 -it -- env | grep _ENVHELLO_ENV=world
FOO_ENV=bar*
```

*不错的小把戏哈？:-)*

*![](img/6814dd0d502aa059c3c830953ca3e7af.png)*

## *作为文件的配置数据*

*另一种使用配置数据的有趣方式是指向`Deployment`或`Pod`规范的`spec.volumes`部分中的`ConfigMap`。*

> **如果你不知道* `*Volumes*` *(在 Kubernetes 中)是什么，不要着急。他们将在即将到来的博客中讨论。现在，只需要理解卷是一种从底层存储系统抽象容器的方式，例如，它可以是本地磁盘或云中的磁盘，如* [*Azure 磁盘*](https://docs.microsoft.com/azure/virtual-machines/windows/managed-disks-overview?WT.mc_id=medium-blog-abhishgu)*[*GCP 持久磁盘*](https://cloud.google.com/persistent-disk/) *等。***

**在上面的规范中，注意`spec.volumes`部分——注意它指的是现有的`ConfigMap`。`ConfigMap`中的每个键都作为一个文件添加到规范中指定的目录，即`spec.containers.volumeMount.mountPath`，其值就是文件的内容。**

> ***注意，如果* `*ConfigMap*` *改变，卷中的文件会自动更新。***

**除了传统的基于字符串的值，您还可以包括成熟的文件(JSON、文本、YAML 等。)作为`ConfigMap`规格中的值。**

**在上面的例子中，我们已经在我们的`ConfigMap`的数据部分中嵌入了一个完整的 JSON。为了进行试验，创建`Pod`和`ConfigMap`**

```
**$ kubectl apply -f   https://raw.githubusercontent.com/abhirockzz/kubernetes-in-a-nutshell/master/configuration/kin-config-envvar-json.yamlconfigmap/config3 created
pod/pod4 created$ kubectl get configmap/config3NAME      DATA   AGE
config3   1     11s**
```

**作为练习，确认环境变量已经植入`Pod`。几个指针:**

*   **`Pod`的名称是`pod4`**
*   **仔细检查您应该寻找的环境变量的名称**

**您也可以使用`kubectl` CLI 创建一个`ConfigMap`。它可能并不适合所有的用例，但是它确实让事情变得简单多了**

# **使用`kubectl`**

**有多种选择:**

## **使用`--from-literal`植入配置数据**

**我们将下面的键值对植入`ConfigMap` — `foo_env=bar`和`hello_env=world`**

```
**$ kubectl create configmap config4 --from-literal=foo_env=bar --from-literal=hello_env=world**
```

## **使用`--from-file`**

```
**$ kubectl create configmap config5 --from-file=/config/app-config.properties**
```

**这将创建一个`ConfigMap` ( `config5`)与**

*   **与文件同名的密钥，即本例中的`app-config.properties`**
*   **和值作为文件的内容**

**您可以选择使用不同的键(而不是文件名)来覆盖默认行为**

```
**$ kubectl create configmap config6 --from-file=CONFIG_DATA=/config/app-config.properties**
```

**在这种情况下，`CONFIG_DATA`将是关键**

## **从目录中的文件**

**您可以一次将多个文件(在一个目录中)中的数据植入一个`ConfigMap`**

```
**$ kubectl create configmap config7 --from-file=/home/foo/config/**
```

**你最终会得到**

*   **多个密钥将与单个文件名相同**
*   **该值将是相应文件的内容**

# **很高兴知道**

**以下是您在使用`ConfigMap` s 时应谨记的事项(非详尽列表):**

*   **一旦您定义了环境变量`ConfigMap`，您就可以在`Pod`规范的命令部分使用它们，即使用`$(VARIABLE_NAME)`格式的`spec.containers.command`**
*   **您需要确保在`Pod`中被引用的`ConfigMap`已经被创建——否则，`Pod`将*不会启动*。解决这个问题的唯一方法是将`ConfigMap`标记为`optional`。**
*   **另一种可能阻止`Pod`启动的情况是当您引用了一个实际上在`ConfigMap`中不存在的键。**

> ***你也可以参考* `[*ConfigMap API*](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#configmap-v1-core)`**

**这就是本期“果壳中的库伯内特”系列的全部内容。敬请关注更多内容！**

**如果你对使用 [Azure](https://azure.microsoft.com/services/kubernetes-service/?WT.mc_id=medium-blog-abhishgu) 学习 Kubernetes 和 Containers 感兴趣，只需[创建一个**免费**账户](https://azure.microsoft.com/en-us/free/?WT.mc_id=medium-blog-abhishgu)就可以开始了！一个好的起点是使用文档中的[快速入门、教程和代码示例](https://docs.microsoft.com/azure/aks/?WT.mc_id=medium-blog-abhishgu)来熟悉这项服务。我也强烈推荐查看 [50 天 Kubernetes 学习路径](https://azure.microsoft.com/resources/kubernetes-learning-path/?WT.mc_id=medium-blog-abhishgu)。高级用户可能希望参考 [Kubernetes 最佳实践](https://docs.microsoft.com/azure/aks/best-practices?WT.mc_id=medium-blog-abhishgu)或观看一些[视频](https://azure.microsoft.com/resources/videos/index/?services=kubernetes-service&WT.mc_id=medium-blog-abhishgu)以了解演示、主要功能和技术会议。**

**我真的希望你喜欢这篇文章，并从中学到了一些东西！如果你做了，请喜欢并跟随。很高兴通过 [@abhi_tweeter](https://twitter.com/abhi_tweeter) 获得反馈或发表评论。**

**[](https://twitter.com/abhi_tweeter) [## 阿布舍克

### Abhishek 的最新推文(@abhi_tweeter)。云开发者🥑@Microsoft @azureadvocates |…

twitter.com](https://twitter.com/abhi_tweeter)**