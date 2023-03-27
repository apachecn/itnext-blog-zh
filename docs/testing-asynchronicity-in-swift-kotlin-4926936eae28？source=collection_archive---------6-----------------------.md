# 在 Swift & Kotlin 中测试异步性

> 原文：<https://itnext.io/testing-asynchronicity-in-swift-kotlin-4926936eae28?source=collection_archive---------6----------------------->

比较 Combine、ReactiveSwift 和 RxJava 进行测试

![](img/806a1c1fe0a46717a734bab640f0170a.png)

多线程很美但是很棘手！

在这个系列中，我们介绍了一个[干净的 MVVM 架构](https://medium.com/perry-street-software-engineering/kotlin-in-xcode-swift-in-android-studio-26a4ace6fc72)，用于在 Swift 和 Kotlin 中编写一致的代码。

![](img/3b2212386ee21964b4266e6027e5c41e.png)

我们干净的 MVVM 架构的核心是[反应流](http://reactivex.io/)，它隐含地将异步引入到我们的应用中。测试反应式流并不简单——当编写使用反应式流的测试时，我们必须找到一种等待或阻塞的方法，直到我们的流完成或数据发出。

![](img/70e98f685c4fbcea1e8351bbf53ec855.png)

反应式编程

# 将执行转移到主线程

在 Android 上，我们可以覆盖调用反应流的线程，这样通常异步调用就变成了同步调用。我们用一个`ExtendWith`块中的两个`beforeEach`方法来实现这一点。

```
@ExtendWith(InstantExecutorExtension::class,         
            RxTestSchedulersExtension::class
```

`InstantExecutorExtension`适用于 [LiveData](https://developer.android.com/topic/libraries/architecture/livedata) ，改编自[这篇博客](https://perrystreet.atlassian.net/wiki/spaces/SWPRJ/pages/33604)的帖子。`RxTestSchedulersExtension`适用于 [RxJava](https://github.com/ReactiveX/RxJava) 中的线程执行。这两种实现都可以在下面找到:

# 使用 TestObserver 评估流

反应流需要测试，为此， [Combine](https://developer.apple.com/documentation/combine) 、 [ReactiveSwift](https://github.com/ReactiveCocoa/ReactiveSwift) 和 [RxJava](https://github.com/ReactiveX/RxJava) 有一个 TestObserver 的概念。TestObserver 是一个给定了流的类，比如一个 [Publisher](https://developer.apple.com/documentation/combine/publisher) (Combine)、[signal producer](https://reactivecocoa.io/reactiveswift/docs/latest/SignalProducer.html#/s:13ReactiveSwift14SignalProducerV)(ReactiveSwift)或 [Observable](http://reactivex.io/RxJava/javadoc/io/reactivex/Observable.html) (RxJava)，它为这些流保存了发出值的历史记录。您也可以使用 TestObserver 来等待一个给定的事件被发出，然后测试才能继续。

注意，我们通常将 TestObserver 对象声明为每个内部类/上下文中的共享对象，我们在`beforeEach`块中分配它们，然后在每个 then 测试中询问它们。

## 结合

要在 Combine 中评估流，请使用 [CombineExpectations](https://github.com/groue/CombineExpectations) 包。这个包定义了一个`Recorder`对象，在功能上相当于 Android 上的一个 TestObserver。

```
let eventsRecorder = context.viewModel.events.record()
context.viewModel.onEditProfileClick()
let event = try eventsRecorder.next().get()
XCTAssertEqual(event, PSSAccountViewModel.Event.triggerProfileOfflineAlert)
```

## 反应 Swift

ReactiveSwift 本身没有 TestObserver，但是 Kickstarter 已经在他们的知识库中定义了这样一个类。下面是一个观察到发出`.flagProfile`事件的例子:

## RxJava

为了评估 RxJava 中的流，我们使用了被定义为`io.reactivex.observers` 包的一部分的`TestObserver`类。这是一个观察到发出了一个`FlagProfile`事件的例子。

## LiveData

我们对 LiveData 的使用通常是作为视图层的[行为主体](http://reactivex.io/RxJava/javadoc/io/reactivex/subjects/BehaviorSubject.html)的替代。因此，如果我们已经应用了上述的 InstantExecutor 扩展，我们通常可以在测试中直接从主线程中访问`.value`属性，而不需要任何特殊的处理或观察者。

# 本系列的更多内容

*   [在斯威夫特清洗 MVVM&科特林](https://medium.com/p/26a4ace6fc72)
*   [Swift 中的视图模型& Kotlin](https://medium.com/p/721bbc6f8c07)
*   [Swift 中的逻辑类& Kotlin](/logic-classes-in-swift-kotlin-f7ac1f295839#4bdb-3e583789601)
*   [Swift 中的存储库和域模型& Kotlin](https://medium.com/perry-street-software-engineering/repositories-and-models-in-swift-kotlin-df9ae2c84cb7)
*   [Swift 中的 API 类& Kotlin](https://medium.com/perry-street-software-engineering/api-classes-in-swift-kotlin-a2cb90d3906d)
*   [Swift 中的视图& Kotlin](https://medium.com/p/f5416dec42ea)
*   [在 Swift 中测试 MVVM&科特林](https://medium.com/better-programming/testing-mvvm-in-swift-and-kotlin-ccb55e462760)
*   在 Swift & Kotlin 中测试异步性← *你在这里*
*   [清洁 MVVM 总结](https://medium.com/perry-street-software-engineering/summary-clean-mvvm-in-swift-kotlin-7057230600aa)

# 你可能喜欢的其他系列

[**清理 API 架构(2021)**](/a-visual-history-of-web-api-architecture-c36044df2ac7)构建现代 API 端点时的类、执行模式、抽象。

[**Android 活动生命周期被认为有害**](https://proandroiddev.com/android-activity-lifecycle-considered-harmful-98a5b00d287)**【2021】** Android 进程死亡，不可解释的 NullPointerExceptions，以及你现在需要的 MVVM 生命周期

# 关于作者

*埃里克·西尔弗伯格(Eric Silverberg)是* [*佩里街软件*](https://www.perrystreet.com/) *的 CEO 和创始人，是 iOS 和 Android 上两个全球最大的 LGBTQ+交友应用的发行商。*