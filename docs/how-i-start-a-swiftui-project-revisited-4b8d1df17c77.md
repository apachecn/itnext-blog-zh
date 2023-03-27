# 我如何启动 SwiftUI 项目——重温

> 原文：<https://itnext.io/how-i-start-a-swiftui-project-revisited-4b8d1df17c77?source=collection_archive---------8----------------------->

## AppDelegate 回来了

![](img/768d430429e237e8acbc6c8922bff22e.png)

由[坚克·巴图汉·奥扎尔通](https://unsplash.com/@c_b_ozaltun?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

使用 SwiftUI 的新生命周期，不再需要显式创建 AppDelegate。但是在某些情况下，可能仍然需要 AppDelegate。这篇文章解释了为什么。

这篇文章是我第一篇文章[“我如何开始一个 SwiftUI 项目”](https://twissmueller.medium.com/how-i-start-a-swiftui-project-28767bb73df9)的后续。

在为我的教程系列[“DJI 无人机编程”](https://twissmueller.medium.com/programming-dji-drones-97974c18af06)创建一个应用程序时，我遇到了一个奇怪的问题。

我的应用表现出不确定性，这是每个程序员的噩梦。有时它会像预期的那样工作，有时则不然。我已经在 Stackoverflow 的这个帖子里解释了所有的细节。

我只能再次重新介绍“过去的好时光”。

因此，每当你的新 SwiftUI 应用需要使用`AppDelegate`时，请看下面的蓝图。

如果你喜欢这篇文章，请给我买杯咖啡。

```
import SwiftUI

class AppDelegate: UIResponder, UIApplicationDelegate {

    var someClass = SomeClass()

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        someClass.doSomethingThatWouldNotWorkWithoutTheAppDelegate()
        return true
    }
}

@main
struct MyApp: App {

    @UIApplicationDelegateAdaptor(AppDelegate.self) var appDelegate
    @Environment(\.scenePhase) private var scenePhase

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: scenePhase) { (newScenePhase) in
            switch newScenePhase {
            case .active:
                print("active")
            case .background:
                print("background")
            case .inactive:
                print("inactive")
            @unknown default:
                print("default")
            }
        }
    }
}
```

## 资源

*   [SwiftUI 的应用生命周期说明](https://learnappmaking.com/swiftui-app-lifecycle-how-to/)
*   [如何使用 UIApplicationDelegateAdaptor 作为 ObservableObject？](https://stackoverflow.com/a/64484944/1065468)