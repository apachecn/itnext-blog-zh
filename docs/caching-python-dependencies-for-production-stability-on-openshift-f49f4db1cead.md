# 在 OpenShift 上缓存 Python 依赖项以实现生产稳定性

> 原文：<https://itnext.io/caching-python-dependencies-for-production-stability-on-openshift-f49f4db1cead?source=collection_archive---------2----------------------->

![](img/63fee388d21625ed1dc24d507564648f.png)

照片由 [rawpixel](https://unsplash.com/@rawpixel?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

你准备好了吗？您已经为部署到生产环境的 Python 应用程序做了一个关键的修复，但是 [PyPi](https://pypi.org/) 出现了故障。当涉及到最大化生产中的稳定性和可用性时，隔离与您的依赖项相关的风险是至关重要的。如果您可以采取一个简单的步骤来最小化依赖中断的风险，并且它带来了对您的构建的性能提升的额外好处，您会接受吗？

![](img/1314df090dc660609fb3c31f585f8dc3.png)

埃文·丹尼斯在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

而 [*PyPi* 断电](https://status.python.org/)，看起来不太可能真的发生。你可能会说，“那种情况下没有镜子吗”？而答案是**是的**有[镜子给*PyPi*。您的运营团队了解他们吗？运营团队知道如何为您的应用程序指定工作镜像吗？如果他们不需要担心这一点，不是更好吗？因为您已经用您的应用程序利用的依赖项创建了一个本地镜像。依赖缓存正是](https://jacobian.org/2010/jul/20/when-pypi-goes-down/) [devpi](https://github.com/devpi/devpi) 所提供的(具体来说就是 *devpi-server* )。

# 在 OpenShift 上运行 devpi

让 *devpi-server* 在 [OpenShift](https://www.openshift.com/) 上运行是一个简单的五步过程。这个简单的设置要归功于开源社区。通过定义[部署](https://github.com/saily/openshift-devpi/blob/master/openshift/deployment.yaml)和[服务](https://github.com/saily/openshift-devpi/blob/master/openshift/service.yaml)定义以及[容器映像](https://hub.docker.com/r/widerin/openshift-devpi/)，项目 [openshift-devpi](https://github.com/saily/openshift-devpi) 已经完成了所有的前期工作。

> `git clone https://github.com/saily/openshift-devpi.git
> oc login https://<your-master-node>:8443
> oc new-project devpi
> oc create -f openshift/deployment.yaml
> oc create -f openshift/service.yaml`

# 在 OpenShift 上使用 devpi

现在你有了 *devpi-server* 并在 *OpenShift* 上运行，你如何利用它呢？利用 *devpi* 发挥作用的好处是，您可以在 OpenShift 上构建图像，尤其是在您使用 [Source to Image](https://github.com/openshift/source-to-image) (s2i)来创建图像流的情况下。

[](https://blog.openshift.com/create-s2i-builder-image/) [## 如何创建一个 S2I 建设者形象-红帽 OpenShift 博客

### 源到图像(S2I)是一个独立的工具，在创建生成器图像时非常有用。S2I 也碰巧…

blog.openshift.com](https://blog.openshift.com/create-s2i-builder-image/) 

我将假设，既然你是用 [Python](https://www.python.org/) 开发并部署在 *OpenShift* 上，你就跟上了开发趋势并使用 [Pipenv](https://pipenv.readthedocs.io/en/latest/) 进行依赖管理。如果你还没有使用 *Pipenv* ，请看看它是不是一个真正伟大的工具。 *Pipenv* 具有[**Pipenv _ PYPI _ MIRROR**](https://pipenv.readthedocs.io/en/latest/advanced/#using-a-pypi-mirror)环境变量，用于指定备用镜像。为了利用集群上现在设置的 *devpi* ，您需要将服务端点传递到 **PIPENV_PYPI_MIRROR** 变量中。如果您将 *devpi-server* 部署到示例项目 devpi，那么您将提供*http://devpi . devpi . SVC . cluster . local:3141*作为本地集群访问的输入值。或者，您可以定义一个允许您进行外部访问的路由，并提供该路由的 URL。为了传入环境变量，您必须将其定义为您的 *BuildConfig* 的一部分，参见链接的[示例](https://github.com/project-koku/masu/blob/master/openshift/masu.yaml#L84)。现在，当您创建应用程序时，您可以将它作为参数提供。如果未提供该参数，那么 *PyPi* 将是默认值。有了所有这些，您还应该看到构建时间的显著性能改进。您还可以将 *devpi-server* 上的**DEVPI _ MIRROR _ CACHE _ EXPIRY**值切换为高于(或低于)一天的默认值，这取决于您的依赖关系更改的频率。

# 摘要

希望这个故事已经告诉了您一个有用的新工具， *devpi* ，并为您提供了一组简短的步骤来强化您的生产环境，使其免受依赖性中断的影响。我们讨论了如何通过五个简单的步骤让 *devpi-server* 在 OpenShift 上运行。接下来，我们看看图像创建如何利用 *devpi* 并获得显著的速度提升。虽然这是一个对生产集群很有帮助的设置，但是它也可以用于开发集群或本地集群。如果你是一个在本地系统上运行 *OpenShift* 的开发者，你甚至可以利用在 *OpenShift* 之外进行本地开发时的性能提升，如果你在运行设置时添加一个外部路径并提供镜像。