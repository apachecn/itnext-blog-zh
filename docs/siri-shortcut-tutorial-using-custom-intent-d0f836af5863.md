# 使用自定义意图的 Siri 快捷教程

> 原文：<https://itnext.io/siri-shortcut-tutorial-using-custom-intent-d0f836af5863?source=collection_archive---------1----------------------->

S 从一个简单的[天气应用](https://github.com/ji3g4kami/Weather)开始，列出了四个城市，我期待当我让 Siri 告诉我这个城市的天气时，Siri 可以直接告诉我天气而不用打开应用。此外，当我在 Siri 中点击窗口时，它会带我去想要去的城市，并提供天气信息。这里我将介绍添加自定义 Siri 快捷方式的过程。

![](img/8c13313eb8dad0e341b6506e94e629cf.png)

> 注意:你可以从 master 分支的[天气 App](https://github.com/ji3g4kami/Weather) 中克隆启动项目。[最终项目在 SiriShortcut 分支](https://github.com/ji3g4kami/Weather/tree/SiriShortcut)。此外，它的屏幕应大于或等于 5.5 英寸。

# 应用程序结构

> 像以前一样，要使用 SiriKit，我们需要对我们的应用程序进行意向应用程序扩展。这个扩展将处理在后台运行的快捷方式。此外，苹果建议 [**将扩展和应用共享的代码拆分到一个框架**。](https://medium.com/flawless-app-stories/wwdc-2018-for-ios-developers-siri-shortcuts-e8e4a78f0ad7)这个框架应该包含所有管理和贡献快捷方式的代码。

来自 [Apple](https://developer.apple.com/documentation/sirikit/media_intent_shortcuts/playing_media_through_siri_shortcuts) 和 [Ray Wenderlich](https://www.raywenderlich.com/6462-siri-shortcuts-tutorial-in-ios-12) 的两个样本项目，AudioCast 和 TheBurgeoningWriter，都符合这个 App 结构，就这么干吧！

![](img/82e5cce8959b1fb710cf514708b57c02.png)

**将扩展和应用之间共享的代码拆分到一个框架中**

在天气 App 里面，我们先添加目标。

![](img/3003f907676827ce063ed140fceb644e.png)

包括单元测试，我们将把之前的单元测试和代码一起迁移到框架中。

![](img/61d2c3dd1c0ed16898f0d9e3ea1dc329.png)

将*服务、支持和型号*文件夹移动到*天气套件*中。然后将目标成员资格更改为 *WeatherKit* 。我们还需要制作 APIManager 和 WeatherManager。

![](img/256cceb848b7d0823256dc33b7a23378.png)

然后我们用通用的 iOS 设备构建 *WeatherKit* ，然后在 *WeatherViewController 的顶部添加`import WeatherKit`。运行应用程序，它应该完全正常！*

与我们对 *WeatherKit* 所做的一样，我们将把我们的测试转移到 *WeatherKitTests* 。确保将目标成员资格更改为 *WeatherKitTests* 并将`@testables`更改为`@testable import WeatherKit`

![](img/2480af729829b888520e2fe9f79eb3ce.png)

按下⌘ + U 并测试它。做一些小的但必要的修改，感觉有点琐碎是很自然的。

顺便说一下，如果我们研究代码覆盖率，我们会发现 *WeatherKitTests* 现在已经从*天气应用*中分离出来。

![](img/b577aed39f023b289b68a22cf7d32614.png)

太好了！我们已经成功地**将扩展和应用程序之间共享的代码分割成一个框架**。

# 添加自定义意图

![](img/9712ee86f60bfe2b6c427f92a4776545.png)

[https://developer.apple.com/documentation/sirikit](https://developer.apple.com/documentation/sirikit)

我们可以通过 Siri 与用户交流，而不必打开带有**自定义意图**的应用程序。要让您的自定义意图发挥作用，需要做一些事情:

*   启用 Siri 功能。
*   添加 Siri Intents 和 Intents UI (UI 是可选的，取决于您的需要)。
*   用 Siri 意图定义文件定义自定义意图。
*   使用 Xcode 自动生成的`IntentHandling`协议处理意图。
*   使用 AppDelegate 中的`restorationHandler`允许应用程序在所需的视图中打开。

## 启用 Siri 功能

![](img/9c98f807269d36e976595c5f49b43824.png)

## 添加 Siri 意图和意图 UI

![](img/8ae7fa38048a8f94480dd9bcb4b7bd31.png)

这里我将命名我的意图 *CityWeatherIntent* 。此外，当 Xcode 询问时激活模式。

![](img/1ec286cc6abdfbdb63a00a0aaf2c26ff.png)![](img/2e612a6e33fe323271aa5237921fb9b6.png)

*CityWeatherIntent (* 和 *CityWeatherIntentUI)* 应该在**链接框架和库**中包含共享框架。

![](img/142e2fed72e52eaeb9fd1c4dcf6012cc.png)

## 使用 Siri 意图定义文件定义自定义意图

![](img/f1fa6a2e6592d3c3c013fef2a31eef2e.png)

只有框架使用**意图类**，其他应该使用**没有生成的类**。

添加您的新自定义意向并完成信息。

![](img/d060ba220dabf5e6be757532d43c8441.png)

Xcode 会自动为您生成这些代码

![](img/51c4c332e1a53ba88b91652a1f648e18.png)

填写回应信息。

![](img/ff5fb6e9bb574afa5f51bd1a1cf1301b.png)

Xcode 的自动生成代码

![](img/1498c30a4b8f1290f7cb4b6bb07c145d.png)

## 使用 Xcode 自动生成的`IntentHandling`协议处理意图

![](img/d3a8eb13fc975792cf9913dba46ddaea.png)

转到 *CityWeatherIntent* 组，用以下代码添加*city weather handler . swift*。

将 *IntentHandler* 的返回类改为`CityWeatherIntentHandler()`

## 使用 AppDelegate 中的`restorationHandler`允许应用程序在所需的视图中打开

将以下代码添加到 *AppDelegate 中。*我们通过 userActivity 得到我们的`city`，然后创建想要的视图控制器。

## 添加 Siri 按钮

![](img/1d1124a74aedfb7f5188798ece6f8435.png)

要添加一个 Siri 按钮，我们需要`import Intents`和`import IntentsUI`，将`INShortcut`指定给我们的按钮，并符合委托。

键入`siriButton.delegate = self`后，编译器会显示警告。只需在下面添加以下代码。

耶！就是这样。运行该应用程序，它的工作作为魅力。🎊

## 参考资料:

*   [为 iOS 12 开发自定义 Siri 快捷键的初学者指南](https://medium.com/@pietropizzi/a-beginners-guide-to-developing-custom-intent-siri-shortcuts-for-ios-12-a3627b7011af)
*   [iOS 12 中的 Siri 快捷键教程](https://www.raywenderlich.com/6462-siri-shortcuts-tutorial-in-ios-12)
*   [SiriKit——如何将 Siri 快捷方式添加到您的应用程序中](https://ramdankorkelia.com/swift-language-blog/2018/12/18/sirikit-how-to-add-siri-shortcut-with-intent-to-your-app)
*   [面向 iOS 开发者的 WWDC 2018:Siri 快捷方式](https://medium.com/flawless-app-stories/wwdc-2018-for-ios-developers-siri-shortcuts-e8e4a78f0ad7)
*   [IOS 10 逐日::第 10 天::SIRIKIT INTENTS UI](https://www.shinobicharts.com/blog/ios-10-day-by-day-day-10-sirikit-intents-ui/)