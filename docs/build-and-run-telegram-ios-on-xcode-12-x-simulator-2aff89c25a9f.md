# 在 Xcode 12.x 模拟器中构建并运行 Telegram-iOS v7.2

> 原文：<https://itnext.io/build-and-run-telegram-ios-on-xcode-12-x-simulator-2aff89c25a9f?source=collection_archive---------3----------------------->

> [hubo.dev 的镜像](https://hubo.dev/2020-11-20-build-and-run-telegram-on-xcode-12-x-simulator/)

![](img/0de27c4d800b3c7c0e240255a9919e80.png)

[哈维·卡夫雷拉](https://unsplash.com/@xavi_cabrera?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

Telegram 启动了针对 iOS 和 Android 的[ [可复制构建项目](https://core.telegram.org/reproducible-builds)，以更有规律的时间表发布其客户端源代码。虽然这是一个伟大的举措，但你可能会发现如何以最小的影响构建和运行 iOS 项目仍然令人困惑。[官方指南](https://core.telegram.org/reproducible-builds#reproducible-builds-for-ios)用于验证构建是否可复制，这需要在 Parallels 虚拟机中安装 macOS Catalina、Xcode 11.x 和其他工具。

对于想要使用代码的开发人员来说，该指南存在一些问题:

-运行 macOS 和 Xcode 的另一个克隆需要一个强大的机器。
- Xcode 11.x 已经过时，最新的 Xcode 12.x 在代码完成方面比[快 15 倍](https://developer.apple.com/documentation/xcode-release-notes/xcode-12-release-notes)。
-它运行自动化脚本文件来生成 IPA 文件，而不显示如何在 Xcode IDE 中打开项目。
-不包括如何在模拟器中构建和运行。

本教程演示了我解决这些问题的方法。它包括安装必要的工具和修改部分项目代码，以使其工作。如果不想看细节，可以快进到最后一节。

> 如果您使用 macOS Big Sur，请参考[使用 Bazel 在模拟器中构建并运行 Telegram-iOS v7.3。](https://bohu.medium.com/build-and-run-telegram-ios-v7-3-in-simulator-with-bazel-fe36f305bd55)

# 安装降压装置

我的开发机器运行的是 macOS Catalina 10.15.7 和 Xcode 12.2 (12B45b)。Telegram-iOS 使用 [**buck**](https://buck.build/) 构建第三方库，生成 Xcode 工作区文件。我们可以通过自制软件安装巴克和它的附属设备 JDK 8:

```
brew tap AdoptOpenJDK/openjdk
brew cask install adoptopenjdk8brew tap facebook/fb
brew install buck
```

请检查您的终端会话，以确保它使用 JDK 8 和降压工程:

```
$ java -version
openjdk version "1.8.0_272"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_272-b10)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.272-b10, mixed mode)$ buck --version
buck version 2020.10.21.01
```

# 构建和生成 Xcode 工作空间

## #1 查看代码

```
git clone --recursive [https://github.com/TelegramMessenger/telegram-ios.git](https://github.com/TelegramMessenger/telegram-ios.git)
cd telegram-ios
```

## #2 设置环境变量

由于 Telegram-iOS 使开发人员能够定制和构建他们自己的客户端变体，构建系统需要一堆环境变量来提供设置。我们可以重用可再生构建指南中的设置，因为我们只想在模拟器中运行它。

我对 [build-system/verify.sh](https://github.com/openaphid/Telegram-iOS/blob/main/build-system/verify.sh) 中的原始设置做了一些小的调整:

*   添加`BUCK`变量。`makefile`依靠它来定位降压命令。
*   添加`BUILD_NUMBER`变量。
*   将`IS_INTERNAL_BUILD`翻转到`true`，将`IS_APPSTORE_BUILD`翻转到`false`。

## #3 启用死代码剥离

我们需要通过将死代码添加到 [Config/utils.bzl](https://github.com/openaphid/Telegram-iOS/blob/main/Config/utils.bzl) 来为所有模块启用死代码剥离:

否则，在使用 Xcode 构建项目时，您可能会遇到以下错误:

## #4 调整 mozjpeg 的构建设置

一些第三方库的构建设置没有模拟器要求的`x86_64`目标，除非你有一台苹果 M1 电脑。需要我们自己加。

是其中一个需要改进的地方。在[第三方/mozjpeg/BUCK](https://github.com/openaphid/Telegram-iOS/blob/main/third-party/mozjpeg/BUCK) 内部，声明了`arm64`和`armv7`两个目标。我们可以将`armv7`改为`x86_64`，因为我们在模拟器和大多数设备上不需要`armv7`。

## #5 针对 webrtc-ios 的修复版本

Telegram-iOS 将其视频通话建立在`webrtc-ios`之上，这是今年引入的一个巨大模块。它应该在生成工作空间文件之前由 buck 构建到一个框架中。不幸的是，官方回购中的代码无法与 Xcode 12.x 编译，也不支持`x86_64`目标。

第一个变化是关于 python 脚本 [find_sdk.py](https://github.com/ali-fareed/webrtc-ios/blob/782743c7931d09c1d2e4a0cf6cd349ee45452f1d/src/build/mac/find_sdk.py) ，它是`webrtc-ios`构建系统的一部分。它有助于在 Xcode.app 中定位 SDK 路径。原始正则表达式`^MacOSX(10\.\d+)\.sdk$`不适用于 Xcode 12.x，因为 SDK 路径名已被更改为`Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX11.0.sdk`。我们需要将表达式改为`^MacOSX(1[01]\.\d+)\.sdk$`，它匹配`MacOSX10`和`MacOSX11` SDK 名称。

另一个修正是添加如下`x86_64`目标:

## #6 生成工作空间文件

现在可以运行下面的命令来构建库并生成工作区文件了:

```
make project
```

这可能需要一段时间才能完成，因为它需要构建像 openssl、ffmpeg 和 webrtc-ios 这样的库。一旦完成，它会在`Telegram/Telegram_Buck.xcworkspace`打开 Xcode IDE，并显示生成的工作空间文件。

# 在模拟器中构建和运行

看到 Xcode 正在运行令人兴奋，但是我们还需要最后一个改变。否则，在模拟器中运行的构建过程将失败，并出现许多链接阶段错误:

```
Undefined symbols for architecture x86_64:
  "static UIKit.UIPointerShape.defaultCornerRadius.getter : CoreGraphics.CGFloat", referenced from:
      Display.(PointerInteractionImpl in _B39DD7E7F85F24DF4F7212BCFE28692F).pointerInteraction(_: __C.UIPointerInteraction, styleFor: __C.UIPointerRegion) -> __C.UIPointerStyle? in PointerInteraction.o
  "enum case for UIKit.UIPointerShape.roundedRect(UIKit.UIPointerShape.Type) -> (__C.CGRect, CoreGraphics.CGFloat) -> UIKit.UIPointerShape", referenced from:
      Display.(PointerInteractionImpl in _B39DD7E7F85F24DF4F7212BCFE28692F).pointerInteraction(_: __C.UIPointerInteraction, styleFor: __C.UIPointerRegion) -> __C.UIPointerStyle? in PointerInteraction.o
  "enum case for UIKit.UIPointerEffect.highlight(UIKit.UIPointerEffect.Type) -> (__C.UITargetedPreview) -> UIKit.UIPointerEffect", referenced from:
      Display.(PointerInteractionImpl in _B39DD7E7F85F24DF4F7212BCFE28692F).pointerInteraction(_: __C.UIPointerInteraction, styleFor: __C.UIPointerRegion) -> __C.UIPointerStyle? in PointerInteraction.o
```

链接器找不到关于指针交互的符号。不知道背后的真正原因。这可能是 Xcode 的问题，或者 buck 可能会生成一些不正确的项目文件。我的解决方案是在[子模块/Display/Source/pointer interaction . swift](https://github.com/openaphid/Telegram-iOS/blob/main/submodules/Display/Source/PointerInteraction.swift#L44)中使用指针交互注释掉代码，因为我不打算在模拟器中测试该特性。

现在，您应该能够在 Simulator 中构建、运行和调试 Telegram-iOS。

# 快进

如果你没有时间享受与复杂构建系统战斗的乐趣，并且想要快进到最后阶段，我已经将这些更改作为补丁文件放在我的[forged repo of Telegram-iOS](https://github.com/openaphid/Telegram-iOS)中。您可以查看官方回购，并运行命令来下载和使用简化过程的补丁程序: