# Web 颤振:创建和运行 Web 应用程序的完整指南

> 原文：<https://itnext.io/flutter-for-web-c75011a41956?source=collection_archive---------0----------------------->

![](img/711b8a0c5081cb06cf6d8ec131614b11.png)

来源:[https://cdn-images-1.medium.com/max/1600/0*gD64Y8ECWBBuSZrx](https://cdn-images-1.medium.com/max/1600/0*gD64Y8ECWBBuSZrx)

在谷歌 I/O 2019 开发者大会上，谷歌发布了 1.5 版的颤振，这是一个开源的移动用户界面框架，可以帮助开发者为安卓和 iOS 构建本地界面。但这不再是事实:移动框架现在是一个多平台用户界面框架，支持网络、桌面、移动甚至嵌入式设备。旋舞的任务已经扩展到建立“为任何屏幕开发美丽体验的最佳框架”。

# 介绍

Flutter 于 2017 年 5 月在谷歌 I/O 上首次发布，当时有一个 alpha 工具包，2018 年在谷歌 I/O 上，它终于推出了未来新产品的 1.0 版，名为*蜂鸟*。我们都很兴奋，迫不及待地想知道它的发布日期。2019 年 5 月 7 日，在谷歌 IO 2019 上，谷歌终于宣布提供了**Web 颤振**预览版。

Flutter 的创建是为了给开发人员一个快速的开发框架，并给用户一个非常吸引人的快速体验。web 颤振是一种代码兼容的颤振实现，使用基于标准的 web 技术:HTML、CSS 和 JavaScript 进行渲染。有了 into for web，您可以将 Dart 中编写的现有 window 代码编译成客户端体验，嵌入到浏览器中并部署到任何 web 服务器上。您可以使用颤振的所有功能，并且不需要浏览器插件。

# 如何安装

为了开发 web，您需要 winp 1.5 及以上版本，它支持使用 winp 瞄准 web，包括将 Dart 编译成 JavaScript。要使用带有`flutter_web`预览的颤振软件开发工具包，确保您已经通过在机器上运行`flutter upgrade`将颤振升级到至少`v1.5.4`。

```
$ flutter upgrade
```

要安装为**腹板颤振**提供构建工具的`webdev`包，请运行以下命令:

```
$ flutter packages pub global activate webdev
```

确保`$HOME/.pub-cache/bin`目录在您的路径中，然后您可以直接从您的终端使用`webdev`命令。

为了将`$HOME/.pub-cache/bin`添加到您的路径中，通过从您的终端运行下述命令打开路径文件。

```
$ touch ~/.bash_profile; open ~/.bash_profile
```

它会用文本编辑打开文件，确保路径中的所有组件都有引用并保存。如果你再次打开它，你会发现你的编辑。

```
**flutter sdk:** 
export PATH=$PATH:[Path to your flutter directory]/flutter/bin**dart sdk:** 
export PATH=$PATH:[Path to your flutter directory]/flutter/bin/cache/dart-sdk/bin**webdev:
mac:** export PATH=$PATH:$HOME/.pub-cache/bin
**windows:** %USERPROFILE%\AppData\Roaming\Pub\Cache\bin
**linux:** $HOME/flutter/.pub-cache/bin
```

> 注意:如果配置`webdev`直接运行有问题，请尝试:
> `flutter packages pub global run webdev [command]`。

现在我们已经完成了开发环境的设置，让我们进入下一步，创建一个 web 项目。

# 颤振腹板开发的工具支持

一旦环境设置完成，您将需要一个 IDE 来开始 web 开发。选择您最喜欢的 IDE，并按照下面的逐步说明进行操作:

## Visual Studio 代码

Visual Studio 代码通过 3.0 版本的 [Flutter 扩展](https://dartcode.org/)支持 Flutter web 开发。

*   [安装](https://flutter.dev/docs/get-started/install)颤振 SDK
*   [设置](https://flutter.dev/docs/get-started/editor?tab=vscode) VS 代码
*   配置 VS 代码指向你的本地 Flutter SDK
*   从 VS 代码中运行`Flutter: New Web Project`命令
*   创建项目后，按 F5 或“调试->开始调试”运行您的应用程序
*   VS 代码将使用`webdev`命令行工具来构建和运行你的应用；一个新的 Chrome 窗口应该会打开，显示你正在运行的应用程序

## 从 IntelliJ 使用

*   [安装](https://flutter.dev/docs/get-started/install)颤振 SDK
*   [设置](https://flutter.dev/docs/get-started/editor)您的 IntelliJ 或 Android Studio 副本
*   配置 IntelliJ 或 Android Studio 指向您的本地 Flutter SDK
*   创建新的 Dart 项目；注意，对于 Flutter for web 应用程序，您希望从 Dart 项目向导开始，而不是从 Flutter 项目向导开始
*   从 Dart 项目向导中，为应用程序模板选择“Flutter for web”选项
*   创建项目；`pub get`会自动运行
*   项目创建完成后，点击主工具栏上的`run`按钮
*   IntelliJ 将使用`webdev`命令行工具来构建和运行你的应用程序；一个新的 Chrome 窗口应该会打开，显示你正在运行的应用程序

## **使用 Android Studio**

在 Android Studio 中，没有直接的插件或模板来创建 web 项目，相反，您可以使用 **Stagehand** 包来帮助您设置 web 项目。Stagehand 基本上是一个 Dart 项目脚手架生成器，灵感来自于 Web Starter Kit 和 Yeoman 等工具。为了用 **Stagehand、**创建一个 web 项目，你需要按照下面的说明操作:

*   [安装](https://flutter.dev/docs/get-started/install)颤振 SDK
*   [设置](https://flutter.dev/docs/get-started/editor)您的 Android Studio 副本
*   配置 Android Studio 指向您的本地 Flutter SDK
*   现在从您的终端运行以下命令来安装 Stagehand `$ pub global activate stagehand`
*   安装 Stagehand 后，您可以使用它在所需的目录中生成项目框架。例如，下面是如何使用 Stagehand 创建一个简单的 web 项目:

```
$ mkdir flutter_web_project
$ cd flutter_web_project
$ stagehand web-simple
```

并列出所有项目模板:

```
$ stagehand
```

*   项目创建完成后，在 Android Studio 中打开该项目，并在您的`pubspec.yaml`文件中添加以下依赖项

```
dependencies:
  flutter_web: any
  flutter_web_ui: any

dev_dependencies:
  # Enables the `pub run build_runner` command
  build_runner: ^1.1.2
  # Includes the JavaScript compilers
  build_web_compilers: ^1.0.0

# flutter_web packages are not published to pub.dartlang.org
# These overrides tell the package tools to get them from GitHub
dependency_overrides:
  flutter_web:
    git:
      url: https://github.com/flutter/flutter_web
      path: packages/flutter_web
  flutter_web_ui:
    git:
      url: https://github.com/flutter/flutter_web
      path: packages/flutter_web_ui
```

*   从你的终端运行`pub get`，这将下载所有必要的软件包
*   一旦完成；在项目的根目录下创建一个`lib`文件夹
*   现在在`lib`文件夹中创建一个`main.dart`文件，并将以下代码粘贴到其中:

```
import 'package:flutter_web/material.dart';

void main() => runApp(Text('Hello World', textDirection: TextDirection.ltr));
```

*   一旦你完成了；打开 web 文件夹中的`main.dart`文件，并将以下代码粘贴到其中:

```
import 'package:fancy_proj/main.dart' as app;
import 'package:flutter_web_ui/ui.dart' as ui;

main() async {
  await ui.webOnlyInitializePlatform();
  app.main();
}
```

*   一旦一切都完成了，您就可以测试您的 web 项目了。通过在终端中键入以下命令运行您的应用程序:

```
$ webdev serve[INFO] Generating build script completed, took 331ms
...
[INFO] Building new asset graph completed, took 1.4s
...
[INFO] Running build completed, took 27.9s
...
[INFO] Succeeded after 28.1s with 618 outputs (3233 actions)
Serving `web` on http://localhost:8080
```

在 Chrome 中打开 [http://localhost:8080](http://localhost:8080/) ，你应该会在左上角看到红色文本的`Hello World`。

## 使用`webdev`进行(无状态)热重装

要将`webdev`用于热重装，请在项目目录中运行以下命令:

```
$ webdev serve --auto restart
```

您会注意到类似于`flutter packages pub run build_runner serve`的输出，但是现在对应用程序代码的更改应该会导致应用程序在保存时快速刷新。

> 注意:`*--hot-reload*`选项并不完美。如果您注意到意外行为，您可能需要手动刷新页面。
> 
> 注意:`*--hot-reload*`选项目前是“无状态”的。重新加载时，应用程序状态将会丢失。我们确实希望在网络上提供“有状态的”热重装——我们正在积极努力！

## 后台模板

Stagehand 有许多不同的模板，下面列出了所有项目模板:

*   `console-full` -命令行应用程序示例。
*   `package-simple`-Dart 库或应用程序的起点。
*   `server-shelf` -使用货架包构建的 web 服务器。
*   `web-angular` -包含材料设计组件的网络应用。
*   `web-simple` -一个只使用核心 Dart 库的 web 应用。
*   `web-stagexl`-2D 动漫游戏的起点。

# 有用的资源

[](https://github.com/flutter/flutter_web) [## 颤动/颤动 _ 网

### 将您的 Flutter 代码带到 web 浏览器中。通过创建一个关于…的帐户，为 flutter/flutter_web 开发做出贡献

github.com](https://github.com/flutter/flutter_web) [](https://flutter.dev/web) [## 腹板颤振

### 宣布 web 版 Flutter 的预览版。

颤振. dev](https://flutter.dev/web) 

***本文到此为止，如果你喜欢这篇文章，别忘了拍拍手👏尽可能多的表达你的支持，留下你的评论并与你的朋友分享。***