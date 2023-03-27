# with Flutter 入门

> 原文：<https://itnext.io/getting-started-with-flutter-forweb-c0647ed51b88?source=collection_archive---------2----------------------->

## 使用 Dart 和 Flutter 构建出色的 web 体验

![](img/0dbdf837dfd5aa581f0bb6730b62f215.png)

## 介绍

F lutter 自 2018 年末首次亮相以来，一直作为移动开发 SDK 受到欢迎，随着`web`作为目标的加入，这款优秀的 SDK 现在可供 web 开发人员使用，以创建令人惊叹的高质量 web 体验，充分利用最新的 web APIs。

在本文中，我们将了解如何使用 Flutter for Web 创建一个简单的网页，该网页由基本布局、一些文本和图像以及一些滚动动画组成，以增加效果。举个简单的例子，它不太可能赢得任何 UX 设计挑战，但它完成了演示的工作。

## 环境

要为使用 Flutter 构建 web 应用程序建立并运行一个工作环境，您需要安装以下工具:

*   **颤振** : [安装页面此处](https://flutter.dev/docs/get-started/install)
*   **Stagehand** : `$ pub global activate stagehand`(创建新应用)
*   **IDE**:IDE 或代码编辑器，如 [VS 代码](https://code.visualstudio.com/)与[Dart 扩展](https://marketplace.visualstudio.com/items?itemName=Dart-Code.dart-code)

要验证你有一个正常工作的 Flutter 安装，运行`$ flutter doctor`，并确保下载任何缺失的扩展或其他依赖项。

这个项目的源代码可以在 GitHub 上找到。确保在项目目录中运行`$ flutter pub get`(除非您有一个支持自动解析依赖关系的环境)。

## 项目结构

这个项目是用命令`$ stagehand web-simple`创建的，它建立了一个空的 web 项目，还没有 *flutter_web* 包作为依赖项，所以我们将把它们添加到项目的 **pubspec.yaml** 文件中:

这个 **pubspec.yaml** 是一个非常典型的 Flutter 项目，添加了一些用于构建 web 应用程序的内容，以及带有 git repo url 和路径的依赖覆盖，因为 *flutter_web* 包尚未发布到[pub.dartlang.org](https://pub.dev)库，如果没有这些覆盖，`pub`将会失败。

## 应用入口点

让我们来看看主源文件， **lib/main.dart** :

对于任何 Flutter 应用程序来说，这都是一个相当标准的主应用程序文件，它实现了 [StatelessWidget](https://gist.github.com/kenreilly/21d08d7fe3afb72e66742d4cc99e425b) ，构建了一个 [MaterialApp](https://api.flutter.dev/flutter/material/MaterialApp-class.html) ，后者包装了 **HomePage** Widget，我们接下来会看到这个 widget。

## 主页

接下来，我们将查看 **lib/home.dart** 中的主页:

这里是我们进入一些真实代码的地方。 **HomePage** 类扩展了一个 [StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html) ，允许它用可变的非最终属性维护自己的内部状态。有一个 *_cards* 属性，它接受导入的部分定义并返回一个要显示的**部分**对象的列表。_controller 属性包含对一个 [ScrollController](https://api.flutter.dev/flutter/widgets/ScrollController-class.html) 的引用，该控件将用于驱动应用程序其余部分中的动画。

这个小部件的 *build* 方法返回一个 [Scaffold](https://api.flutter.dev/flutter/material/Scaffold-class.html) ，它包含一个 [Stack](https://api.flutter.dev/flutter/widgets/Stack-class.html) (用于在 z 轴上堆叠小部件)，一个 **Background** 用于接收 ScrollController 来驱动视差动画，一个 [ListView](https://api.flutter.dev/flutter/widgets/ListView-class.html) 用于显示要滚动的页面部分的列表，在这种情况下，它使用提供的控制器来驱动列表(而不是默认在内部创建一个)。对于在 Flutter 中管理滚动效果来说，这是一个相当标准的配置，因为它允许创建一个具有自定义行为的控制器，然后用于直接驱动滚动元素，并通过 [AnimatedWidget](https://api.flutter.dev/flutter/widgets/AnimatedWidget-class.html) (或通过使用相关概念，如 [AnimatedBuilder](https://api.flutter.dev/flutter/widgets/AnimatedBuilder-class.html) )驱动任意数量的动画效果。

## 背景

接下来是 **lib/background.dart** 中的网站背景:

**背景**小部件是 AnimatedWidget 的一个实现，它接受一个[图像](https://api.flutter.dev/flutter/dart-ui/Image-class.html)和一个 Listenable 对象(或者在本例中是一个 ScrollController，它是实现 [Listenable](https://api.flutter.dev/flutter/foundation/Listenable-class.html) 的许多类中的一个)。listenable 的值被传递给超类，以允许应用程序在发生滚动事件时刷新动画小部件。在*构建*方法中，执行计算以确定用户在 ListView 上滚动了多远(使用相同的 ScrollController ),并且将该值作为 y 偏移提供给图像上的 alignment 属性，这导致缓慢的滚动运动，当与前景中的滚动内容结合时，产生良好的视差效果。此外，还会执行各种空检查，因为当这个小部件第一次呈现时，控制器可能还没有完全初始化和准备好。

## **页面部分定义**

现在让我们检查一下 **lib/section-def.dart** 中的页面定义:

**SectionDef** 类用于定义页面部分及其名称、描述和图像。此外，该文件中还有部分定义列表，主页使用该列表创建要渲染的**部分**对象列表。

## 页面部分类

最后，我们来看看 **lib/section.dart** 中页面部分的代码:

这个文件中有两个类，一个是**内容**类，它基本上是一个小的实用工具，在这里用来防止代码块的过度嵌套，这在一个 Flutter 项目中可能会很快失控；另一个是实际的**部分**类本身，它绘制页面部分，并将以类似方式获得的不透明度应用于背景中的视差滚动效果计算。

**节**类构造函数取当前项的*索引*(在节列表中的位置)*总数(节数)、项*(节定义)*listanable*(驱动动画)。这个类根据它在节定义列表中的位置以及当前的滚动位置来计算它自己的不透明度。这里的目标是在 *0.2 (* 用于当前不在视图中的项目)和 1.0(用于滚动到视图中的项目)之间缩放不透明度。这实际上是从数字 *1.0(最大不透明度)*减去项目和当前滚动位置之间距离的绝对值得出的。这样，当项目向任一方向滚动出视图时，它们的不透明度会增加。

## 结论

在写这篇文章的时候，Flutter for Web 还在开发中，但是 Google 的优秀人员正在努力将它合并到 Flutter 的主分支中，作为 iOS 和 Android 的另一个构建目标。

请随意克隆这个项目的[模板 repo](https://github.com/kenreilly/flutter-web-example) 并对其进行实验以了解更多信息，并查看官方的 Flutter for 网页以了解更多信息和该项目的最新开发成果。

感谢阅读！

> 肯尼斯·雷利( [8_bit_hacker](https://twitter.com/8_bit_hacker) )是 [LevelUP](https://lvl-up.tech/) 的 CTO