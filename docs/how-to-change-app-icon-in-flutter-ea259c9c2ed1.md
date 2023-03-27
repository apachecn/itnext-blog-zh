# 如何在 Flutter 中更改应用程序图标

> 原文：<https://itnext.io/how-to-change-app-icon-in-flutter-ea259c9c2ed1?source=collection_archive---------0----------------------->

## 用 Flutter 启动器图标改变你的应用程序图标

![](img/d2ec75bae56358bed9fd87c493bcd549.png)

当我们创建一个 Flutter 项目时，它带有默认的 Flutter 图标。为了在谷歌 Play 商店、苹果应用商店等商店发布应用程序，可以更改默认图标。

包装就像这样解释它自己；

> 一个命令行工具，简化了更新你的 Flutter 应用程序的启动器图标的任务。完全灵活，允许您选择您希望为哪个平台更新启动器图标，并且如果您愿意，可以选择保留您的旧启动器图标，以防您希望在将来的某个时候恢复。

## 使用

该软件包的使用非常简单。

我们先把包添加到`pubspec.yaml`。

```
dependencies:
  flutter_launcher_icons: ^0.9.2
```

然后你需要做的就是给出你的图标路径，就这样。你也可以给 Android 和 IOS 不同的路径。

下面显示的是您可以在 Flutter 启动器图标配置中指定的属性的完整列表。

## android:和 ios:

*   `true`:覆盖指定平台的默认现有颤振启动器图标
*   `false`:忽略为这个平台制作启动器图标
*   `icon/path/here.png`:这将为平台生成一个新的应用程序图标，并使用您指定的名称，而不会删除旧的默认现有 Flutter 启动器图标。

## 图像路径:

要用作应用程序启动器图标的图标图像文件的位置

*   `image_path_android`:特定于 Android 平台的图标图像文件的位置(可选——如果未定义，则使用 image_path)
*   `image_path_ios`:特定于 IOS 平台的图标图像文件的位置(可选-如果未定义，则使用 image_path)

***注意:iOS 图标应该*** [***填充整个图像***](https://stackoverflow.com/questions/26014461/black-border-on-my-ios-icon) ***并且不包含透明边框。***

接下来的两个属性仅在生成 Android 启动器图标时使用

*   `adaptive_icon_background`:用于填充自适应图标背景的颜色(如`"#ffffff"`)或图像资产(如`"assets/logo.png"`)。
*   `adaptive_icon_foreground`:将用于自适应图标的图标前景的图像资产

编辑完像这样的`pubspec.yaml`文件后，你需要做的就是在你的终端上运行这段代码。

```
 flutter pub run flutter_launcher_icons:main 
```

仅此而已。你已成功根据需要更改了你的应用图标。

本文到此为止。感谢您的阅读。

如果你有任何问题，请随时评论。👏👏👏