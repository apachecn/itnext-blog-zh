# IntelliJ IDE 线性任务插件简介

> 原文：<https://itnext.io/introducing-linear-task-plugin-for-intellij-ide-9fe9818cf487?source=collection_archive---------6----------------------->

## 一个 IntelliJ 任务服务器插件，用于在 IDE 中查看和管理线性问题

# 什么是线性？

[Linear.app](https://linear.app/) 是一款快速、易用、美观、有趣的现代问题跟踪工具。其以键盘为中心的设计提供了许多快捷方式，使您更有效地管理您的项目、任务、路线图和冲刺。使用 Github、GitLab、Slack、Figma、Sentry、Zapier、Zendesk 和 Google Sheets 等各种集成工具，与您喜欢的服务进行集成。如果这还不够，您甚至可以使用基于 GraphQL 的 API 创建自己的定制工作流。

[](https://linear.app/) [## 线性-您喜欢使用的问题跟踪工具

### Linear 有助于简化软件项目、冲刺、任务和错误跟踪。它专为高绩效团队打造。

linear.app](https://linear.app/) 

# 为什么是插件？

像许多其他开发人员一样，我在 IDE 中花费了大量时间。我想“如果我可以在我的 IDE 中直接管理线性任务会怎么样？”IntelliJ 中的通用任务服务器应该可以完成这项工作，但不幸的是，它不支持 GraphQL。这就是为什么我为基于 IntelliJ 的 IDE 创建了[线性任务插件](https://plugins.jetbrains.com/plugin/16769-linear-tasks)，使用它你可以**查看任务列表，打开和关闭任务**，在你的 IDE 中更新问题状态。

在本文中，我们将介绍插件的特性，安装插件并对其进行配置。然后我们将看到插件在运行，查看任务列表，任务描述，打开和关闭任务，当我们执行这些操作时，观察任务在线性板上改变它的状态。我们开始吧！

# 特征

## 查看任务

查看 IDE 中的任务列表以及说明和注释。

![](img/af5e02afb5b40f481c7166b187d4518b.png)

## 在浏览器中查看

想要在浏览器中查看任务。只需点击任务描述中的链接，在线性网站上查看即可。

![](img/4c4307ae459a3b06b8403e1ee19215c3.png)

## 打开任务

双击列表中的任何任务，开始处理它。从对话框中创建分支和变更列表。通过从下拉列表中选择一个线性状态来更新任务状态。无需转到线性页面手动更新状态。

![](img/993735886162b38a0c83b974db5a8d2b.png)

## 关闭任务

当您完成工作后，关闭提交更改、合并分支和更新线性状态的任务。

![](img/a97a9ba2f070e6533f6c6451a468d595.png)

# 装置

1.打开任何 JetBrains IDE
2。导航至设置/首选项|插件|市场

![](img/fc7f07d3de5df85fcc5c39847510e26e.png)

3.搜索**线性任务**

![](img/76e76d17a31592a409d7e21abff2ab7f.png)

4.安装插件并单击应用

![](img/04ef93fa6bd44f02e6e8810768f69271.png)

# 配置

1.导航到设置|工具|任务|服务器

![](img/3020c8d57f68774530a8ab3675f9e6ae.png)

2.点击添加按钮，并从下拉列表中选择线性。

![](img/d2c4f403b03b67592ae4780878975a4f.png)

3.设置您的线性团队 ID 和 API 键

![](img/36f9f551229bf58fd206e7eafa2fade8.png)

4.单击测试以验证设置

![](img/604549435bd8e7c52bda290c14d3a12a.png)

5.应用更改，然后单击确定

# 查看任务

*   要查看任务列表，您可以使用以下两种方法之一:

![](img/f054591ab783374020aabb8bf55c81e6.png)

1.使用工具栏中的下拉菜单，点击*打开任务*

![](img/0f6dd87f5be5c6aa002689cef4377f8f.png)

2‎.使用菜单栏。导航到工具|任务和上下文|打开任务

*   将打开一个显示任务的对话框

![](img/37c64faad249b8c708f60d6ddf174d8f.png)

*   您可以通过使用*快速文档查找*快捷方式查看任何任务的详细信息，包括描述和注释，例如，对于 Mac 使用 F1，对于 Windows 和 Linux 使用 Ctrl + Q

![](img/89ab8a6d0ea7e97862e1b5407b37a1ac.png)

# 打开任务

*   从列表中选择任务并双击它。*打开任务*对话框将出现，有各种选项，如创建一个新的分支和变更列表。

![](img/fa353dd3404f8784645866bcc8a0db75.png)

*   检查**更新发布状态**。下拉列表显示线性板中的所有状态。从下拉列表中选择您希望问题转移到的状态。让我们选择进行中的*和*

![](img/df61d2c5cdfd18e6b08018a00fcc43be.png)

*   您将会看到问题已移动到线性板上的选定状态

![](img/24de325cd91af84caa4055e4cf072945.png)

# 关闭任务

*   要关闭任务，请转到菜单栏并导航到工具|任务和上下文|关闭活动任务

![](img/d8af896b4565a54824f1afdc56f11b4f.png)

*   将打开“关闭任务”对话框，其中包含提交更改、合并分支和更新问题状态的选项

![](img/237f060f6b75e032f10faffba21ba345.png)

*   检查*更新问题状态*并从下拉列表中选择所需的状态。让我们选择*完成*。

![](img/47705ac6affdf4effd85d136c2e4e0eb.png)

*   您将看到问题在线性板上已进入“完成”状态

![](img/46d4540a18379919c5fe01f39267173e.png)

# 那都是乡亲们！

我制作这个插件是为了满足我的需求，但是我决定和大家分享它。希望你喜欢！此外，项目 [linear-app-plugin](https://github.com/mayankmkh/linear-app-plugin) 是完全开源的，所以请随意钻研代码，提出问题或创建 PR，我将等待您的反馈，以使它变得更好。

试一试，分享快乐！现在安装插件。

[](https://plugins.jetbrains.com/plugin/16769-linear-tasks/) [## 线性任务-插件| JetBrains

### 在这个插件的帮助下，将 Linear.app 作为任务服务器添加到 Jetbrains IDE 中。

plugins.jetbrains.com](https://plugins.jetbrains.com/plugin/16769-linear-tasks/) 

# 谢谢

*   [线性](https://twitter.com/linear)用于创建 awesome 工具
*   JetBrains ，世界上一些最好的 ide 的创造者，为创建插件提供支持
*   JetBrains 的米哈伊尔·戈鲁贝夫和[雅各布·克里斯扎诺夫斯基](https://twitter.com/hszanowski)帮助我开始实现这个插件
*   Umbert 和 Ekion 率先采用

*原发布于*[*https://blog . mayankmkh . dev*](https://blog.mayankmkh.dev/introducing-linear-task-plugin-for-intellij-ide)*。*