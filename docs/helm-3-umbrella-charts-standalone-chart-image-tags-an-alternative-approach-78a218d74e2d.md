# Helm 3 伞状图表和独立图表图像标签——一种替代方法

> 原文：<https://itnext.io/helm-3-umbrella-charts-standalone-chart-image-tags-an-alternative-approach-78a218d74e2d?source=collection_archive---------1----------------------->

![](img/dcda151be4d4c287372b82b3010ad989.png)

对于那些不熟悉的人来说，Helm 伞形图将松散耦合的 Kubernetes 组件的可部署集合描述和封装为一个更高阶的 Helm 图表。换句话说，软件元素的集合，每个元素都有自己单独的图表，但是由于某种原因(例如，设计选择、易于部署、版本复杂性)，必须作为原子单元安装或升级。

伞状图的一个简单用例可以是一个 web 应用程序，它有一个单独的 web-scraper 组件来填充数据库。在这个简单的例子中，web 应用程序和 scraper 将分别在它们自己的舵图中进行描述，这些舵图可以单独部署。出于示例的目的，让我们假设一个应用程序离开另一个就无法启动，并且由于某些遗留原因，这两个应用程序不能分开发布。这是伞状图的一个很好的用例，因为伞状图将两个应用程序封装到一个可部署的单元中。与 Helm 命令行标志(如 *atomic* )一起，Helm 将确保一个组件安装或升级失败时，两个组件都回滚到之前的状态。

# 关注点分离

在上面的例子中，web 应用程序及其附带的 scraper 应用程序都在各自的 Helm 图表中进行了描述。当然，包装这两个图表的高阶伞状图表将简单地在 *Charts.yaml* 文件的依赖项部分包含这些图表的正确版本，如下所示:

```
**dependencies:
  -name: web-application
   version: 4.3.8
  -name: web-scraper
   version: 2.2.9**
```

单个应用程序的 *values.yaml* 文件通常包含构建时生成的两个应用程序工件的图像和标记版本。

```
**images:
  name: web-application
  tag: 7.6.1.327**----------------------------**images:
  name: web-scraper
  tag: 9.2.7.426**
```

伞状图引用舵图本身的版本，而不是容器图像的基础版本。这意味着对映像版本的任何更改都会导致对单个组件图表的图表修改。

虽然这可以工作，但实际上这不是特别有效，而且在我看来，当专门使用伞状图来部署底层应用程序时，这不是一个实际的好主意。首先，需要利用伞状图的应用程序堆栈通常有两个以上的依赖关系。这样做的最终结果是，对于软件组件的每个版本，单个 Helm 图表中标签的版本号都需要更新。如果您正在使用图表博物馆，这意味着更新已经构建的每个图表中的图像标签版本，并将这些图表的新版本推送到图表博物馆，通常带有新的图表版本号。子图表的新版本号也需要在伞状图表中更新。当然，你可能会说 CI 工具可以很容易地做到这一点——而且它们可以做到(任何事情都可以编写脚本),但是它们应该做到吗？

我问自己的问题是——从 Kubernetes 的角度看，发生了什么变化。我的结论是——很少！是的，软件的版本发生了变化，但是在大多数情况下，软件版本更新对应用程序部署到 Kubernetes 环境的方式没有影响。由于 Helm 描述了将应用程序与其他环境变量一起安装到环境中的方式，因此对我来说，软件的版本不是需要更新 Helm 图表版本的结构元素，也不一定会被推送到图表博物馆。

总的来说，我们仍然需要让软件的新版本反映在 Helm umbrella 图表中，以便部署它——那么我们如何实现这一点呢？为了在我自己的真实项目中实现这一点，我再次使用了 *values.yaml* 文件的*全局*部分。此部分是伞状图表所独有的，允许子图表引用伞状图表中的变量。你可以参考[上一篇关于如何使用全局变量的文章](https://medium.com/@chris.parker.za/helm-3-umbrella-charts-global-values-in-sub-charts-666437d4ed28)。考虑以下从伞状图表 *values.yaml* 文件中提取的内容。

```
**global:
  image:
    tags:**
      **web-application: 7.6.1.327
      web-scraper: 9.2.7.426**
```

通过允许子图表从伞状图表继承标签版本，如在上面的例子中，必须维护子图表版本的开销被减轻，并且每个单独的子图表不必在每次发布完成时被改变。因此，伞状图是描述多应用程序“发布”的中心位置。

虽然这种方法可能会偏离舵图的预期版本，但它使伞图的工作变得容易得多。通常，子图表版本不同于图表描述的底层软件版本。当引用伞状图表中的子图表时，应用程序版本因此不明显。为了减轻这种情况，当然可以更改子图表版本以匹配图像版本，但是在我看来，只有在进行重大更改时，这才失去了对图表进行版本控制的基本原理。

# 这哪里不适用？

在仅单独部署独立/子图表的项目中，通常情况是直接在每个图表中更新舵图表图像标签版本。这当然应该保持这种方式，因为没有更高阶的图表部署子图表。

即使在使用伞状图表时，如果项目的一部分使用子图表作为独立的图表，人们可能仍然希望控制子图表中图像的版本。

# 结论

当利用伞状图部署多组件堆栈时，尤其是在堆栈的子组件很少(如果有的话)被独立部署的情况下，将图像标签版本迁移到伞状图并在此维护它们是有意义的。这种项目的例子是，严格的发布治理到位，发布经理打包一组图表以发布到生产环境中。以我的经验来看，大公司和政府部门肯定是这种情况，当然，也不总是这样。

通过将子图表视为仅用于部署的方法，单个图表的版本控制和发布被保持在最低限度，并且仅在需要对单个子图表进行开发时才是必要的，即，改变它被部署到 Kubernetes 环境的方式，而不是每次应用程序版本改变时。

让我知道你的想法。你喜不喜欢这种效率？请在评论中告诉我。