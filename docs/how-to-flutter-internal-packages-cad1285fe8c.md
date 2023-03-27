# 如何，抖动内部包

> 原文：<https://itnext.io/how-to-flutter-internal-packages-cad1285fe8c?source=collection_archive---------3----------------------->

![](img/847230a95b4372e515bc9a775d79a6e5.png)

这是一个如何使用内部包作为内部插件的指南。您可能不想公开您的内部库。有一种方法可以让内部包模仿内部插件，我会在这篇文章中向你展示。

# **背景**

由于 Flutter pubspec 和 builds 的工作方式，任何具有可访问 URL 的包、模块等都可以使用。而且，只要您将内部包嵌套在想要使用这些包的项目组中，相对 URL 就可以工作。

# **步骤**

我使用的代码来自日志，专家之路文章:

[https://medium . com/codex/logging-the-expert-way-5b b5 c 967 e 44](https://medium.com/codex/logging-the-expert-way-5beb5c967e44)

创建包项目后，我们需要修改 pubspec 以

放入主页:

```
homepage: "https://fredgrott.medium.com"
```

然后，您只需放入代码类或函数，在 projectname.dart 文件中，您将放入:

```
library lumberjack;export 'initlog.dart';export 'logexception.dart';export 'loggertypes.dart';export 'logpens.dart';export 'types.dart';
```

库名就是项目名。导出会导出所有项目库文件。要在项目中使用它，只需使用一个相对 URL，使用 pubspec 中声明的路径:

```
lumberjack: path: "../../internal_packages/lumberjack"
```

# **结论**

它并不完美，因为你必须将使用内部包(插件)的项目分组在一起，但它确实允许人们使用内部插件而不完全公开它们。而且，这恰好符合大多数用例；因此，这是一项值得了解和使用的好技术。

# **资源**

特定于文章的资源:

颤振设计和 arch Rosetta https://github . com/Fred grott/flutter _ design _ and _ arch _ Rosetta

一般颤振和飞镖资源；

https://flutter.dev/community 颤振社区资源

颤振 SDK https://flutter.dev/docs/get-started/install

Android Studio IDE https://developer.android.com/studio

微软的 Visual Studio 代码 https://code.visualstudio.com/

https://flutter.dev/docs 颤振博士

https://dart.dev/guides dart Docs

谷歌 Firebase 移动设备测试实验室 https://firebase.google.com/docs/test-lab

# **商标公告**

Google LLC 拥有以下商标:飞镖，颤振，机器人，机器人，诺托。苹果公司拥有 iOS、MacOSX、Swift 和 Objective-C 的商标。苹果公司拥有 SF Pro、Sf Compact、SF mono 和 New York 字体的商标。JetBeans Inc .拥有 JetBeans、IntelliJ 和 Kotlin 的商标。甲骨文公司拥有 Java 商标。微软公司拥有微软视窗操作系统和 Powershell 的商标。Gradle 是 Gradle Inc .的商标。Git 项目拥有 Git 的商标。Linux 基金会拥有 Linux 的商标。智能手机 OEM 自有商标到其手机产品名称。尽我所能，我遵守上述商标的品牌和使用指南。

# **关于弗雷德·格罗特**

我是一个疯狂的 SOB，作为一名前 android 移动开发者，我开始写关于 flutter 移动应用程序开发、设计和生活的文章(见 [Eff 新冠肺炎和 GOP](https://fredgrott.medium.com/eff-covid-and-the-gop-e912db0548b8) )。我会达到关键的每月 100 万观众大关吗？坐下来看。在[邢](https://www.xing.com/profile/Fred_Grott/cv)、 [LinkedIN](https://www.linkedin.com/in/fredgrottstartupfluttermobileappdesigner/) 、 [Keybase](https://keybase.io/fredgrott) 、 [Twitter](https://twitter.com/fredgrott) 等社交平台上找我。