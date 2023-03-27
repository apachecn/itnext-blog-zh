# 通过使用 TestAppDelegate 改进测试用例执行时间

> 原文：<https://itnext.io/improve-test-case-execution-time-by-using-testappdelegate-ad0bebb7e17?source=collection_archive---------1----------------------->

![](img/1f4a5697f12e41fa7063f2d005303f68.png)

信用:[https://www.pexels.com](https://www.pexels.com)

单元测试应该尽可能的快，任何减慢测试速度的东西都应该被移除。

我们的应用程序中有超过 1K 个测试用例。随着测试用例的增加，运行测试的速度变慢了。我和同事们就此进行了讨论，并从讨论中总结出两个词，即`Set none to host application`或使用`TestAppDelegate`。我对`host application`进行了研究，并了解到它是必须进行应用测试的。我们在其他目标中有一个测试，即 cocoa touch 框架(库)，它不需要主应用程序作为主机来运行测试，它在那里工作得很好。到现在`host application`点已经明确。

测试顺序是这样的:

1.  启动模拟器
2.  在模拟器中，启动应用程序
3.  将测试包注入正在运行的应用程序
4.  运行测试

每当测试用例运行时，它调用 main.m，main . m 调用 appDelegate 重代码。测试运行时使用`TestAppDelegate`很好。它应该在我们开始新项目时设置，以避免任何副作用，因为应用程序代理将随着时间的推移而增长。可惜我们现在要用`TestAppDelegate`了。

# 为什么要用测试`AppDelegate`？

当 iOS 启动一个应用程序时，它需要以下内容之一:

*   `@UIApplicationMain`符号:它应用于一个类，表明它是应用程序委托。
*   `UIApplicationMain()`函数:在主入口点(通常是`main.swift`)调用它来创建应用程序对象、应用程序委托并设置事件周期。

默认情况下，一个 iOS 项目有一个符号为`@UIApplicationMain`的类`AppDelegate`。意味着这个类是你 app 的入口。

然后，iOS 必须找到 UI 的入口点。我们可以使用两种不同的方式来加载主 UI 组件:或者是`Using Storyboard`或者是`Load Programmatically`。

## 以编程方式加载:

## 创建新的应用入口点(main.m)

首先，我们需要一个新的入口点来加载普通或测试`AppDelegate`，这取决于应用程序是否由单元测试启动。

应用程序的入口点可以是带有符号`@UIApplicationMain`的`AppDelegate`或`UIApplicationMain()`函数。

第一步是从 AppDelegate.swift 中删除`@UIApplicationMain`注释，创建一个新文件`main.swift`。在这个文件中，我们必须检查应用程序是否通过单元测试启动:

```
**let** isRunningTests = NSClassFromString("XCTestCase") != nil
```

现在，我们必须决定加载哪个应用程序委托类:

```
return isRunningTests ? NSStringFromClass(TestAppDelegate.self) : NSStringFromClass(AppDelegate.self)
```

代替`TestAppDelegate` nil 也可以是 return。

```
return isRunningTests ? nil : NSStringFromClass(AppDelegate.self)
```

最终`main.swift`看起来是这样的:

## TestAppDelegate

我们知道`TestAppDelegate`只在单元测试之前被调用一次。这意味着您可以在运行单元测试集之前，在您的`TestAppDelegate`中运行一次测试逻辑。

在这里，我们不需要将视图控制器分配给窗口。

如果您正在使用任何 ui 测试或快照框架，需要由带有关键窗口的应用程序托管测试，那么您需要使用下面的测试应用程序委托。

在这个改变之后，所有的测试都在更短的时间内通过了。

如果你在 app delegate 中使用了核心数据，并且想要初始化，有一篇好文章可以跟随:[https://marcosantadev . com/fake-app delegate-unit-testing-swift/](https://marcosantadev.com/fake-appdelegate-unit-testing-swift/)

# 结论

我们已经将测试的应用启动步骤减少到最低限度。在设置项目时尽早采取这一步骤是有好处的，因为您真正的 iOS 应用程序代表最终会增长。

感谢阅读文章。

你可以在以下地址找到我:

**Linkedin:** [阿艾娜·贾恩](https://www.linkedin.com/in/aaina-jain/)

**推特:** [__aainajain](https://twitter.com/__aainajain)