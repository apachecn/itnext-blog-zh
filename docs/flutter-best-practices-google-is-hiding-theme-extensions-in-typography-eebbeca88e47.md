# 谷歌隐藏的最佳实践，排版中的主题扩展

> 原文：<https://itnext.io/flutter-best-practices-google-is-hiding-theme-extensions-in-typography-eebbeca88e47?source=collection_archive---------5----------------------->

![](img/acc24fd53971f46f79261a0947d287a1.png)

因此，我想介绍的第一个最佳实践允许我们扩展设计范围，我们可以以某种方式扩展主题，不仅可以在 Flutter SDK 更新之前更新到 Material Design 3，还可以轻松地扩展主题以主题化新的 UI 组件。

所以让我们开始吧。

# **准备**

为此，我们需要使用应用程序树顶部的主题继承小部件，例如:

这将允许使用 Flutter Platform Widgets Platform ThemeData 函数来引用和访问我们在 UI 组件中添加到 theme data 主题的字段，以进一步定制它们，以便迁移到 Material Design。在这个具体的例子中，我们将处理排版。

材料设计三的排版规格在这里:

[](https://m3.material.io/styles/typography/tokens) [## 印刷术-材料设计 3

### 材料设计类型比例包括一系列支持产品需求的对比风格

m3.material.io](https://m3.material.io/styles/typography/tokens) 

完整的材料设计三项规格如下:

[](https://m3.material.io/) [## 材料设计

### 更快地构建美观、可用的产品。材料设计是一个适应性强的系统——由开源代码支持——有助于…

m3.material.io](https://m3.material.io/) 

请记住，当我们打电话时:

**(语境)**

或者在使用来自 Flutter 平台小部件的 platformThemeData 调用的情况下，我们正在获取一个在 Data 参数中提供的 ThemeData 的新实例。

因此，我们需要一种方法来授予父 ThemeData 实例，以便我们可以添加额外的字段。我将使用两件东西，一个具有额外字段的类构造和一个扩展。让我们来看看实际情况。

# **实现**

添加的字段:

请注意，我并没有给他们命名当他们改变字体设计类时 Google 将会使用什么。还要注意，因为我没有扩展一个类接口，所以我没有将我的解决方法直接耦合到 Flutter SDK 更改。

现在进行扩展:

我所做的是得到一个参考印刷黑色主题参数，其中将包含一个空白的文本主题，然后将收到我的额外字段与他们设置。

我还需要创建几个扩展来覆盖排版类的所有参数，并保持方法名的不同，这样就不会有方法名冲突。完整的来源是:

现在让我们来看看它的作用

# **使用文本主题排版扩展**

首先，我们有排版功能:

然后每个文本样式看起来像这样:

请记住，字体粗细低于 MD2 的字体现在必须设置行高。计算方法是从 MD3 规格中取出行高，然后除以字体大小:

要使用它:

**#**#**结论**

无论什么样的更新即将到来，你现在有一个技术来推动设计的优势，不再可以坐在你的桂冠上等待颤振更新到新的材料设计规格。当你在代码和设计上都可以超越极限的时候，这不是很有趣吗？

# 关于我，弗雷德·格罗特

从 Android Java-Kotlin 开发转移到 Flutter 应用程序开发。我为 Flutter 社区插件做出了贡献，例如:

[](https://pub.dev/packages/flutter_platform_widgets) [## Flutter _ platform _ widgets | Flutter 包

### 这个项目是一个尝试，看看是否有可能创建平台感知的小部件。目前为了…

公共开发](https://pub.dev/packages/flutter_platform_widgets) [](https://pub.dev/packages/catcher) [## 捕捉器|颤动组件

### Catcher 是一个 Flutter 插件，它自动捕捉错误/异常并处理它们。捕手提供多种方式…

公共开发](https://pub.dev/packages/catcher) 

在此报告中，添加了 Flutter 最佳实践的 Flutter Skeleton 应用程序正在构建中:

[](https://github.com/fredgrott/flutter_starterapp_raw) [## GitHub-Fred grott/Flutter _ starter app _ raw:Flutter starter app

### 此时您不能执行该操作。您已使用另一个标签页或窗口登录。您已在另一个选项卡中注销，或者…

github.com](https://github.com/fredgrott/flutter_starterapp_raw) 

您可以关注我

[](https://keybase.io/fredgrott) [## 弗雷德·格罗特(弗雷德·格罗特)

### Keybase 是什么鬼东西？Keybase 为您提供了管理身份、创建安全聊天和…

keybase.io](https://keybase.io/fredgrott) [](https://twitter.com/fredgrott) [## JavaScript 不可用。

### 编辑描述

twitter.com](https://twitter.com/fredgrott) [](https://github.com/fredgrott) [## fredgrott -概述

### 我是创业前端开发设计师。我正在构建一些 flutter 应用程序和编写一些 flutter 应用程序开发人员…

github.com](https://github.com/fredgrott)