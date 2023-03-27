# 反应本机性能优化和分析

> 原文：<https://itnext.io/react-native-performance-optimization-and-profiling-5b586e9018f8?source=collection_archive---------0----------------------->

![](img/9c1c8d68b0d0cf9fc61e871511744f7f.png)

React Native 是一项受工程师欢迎的技术，这些工程师正在寻求以一种高效的方式构建移动应用程序，而不牺牲本机性能。

如今的设备都配备了多核 CPU 和大量内存。这对于处理大型数据集、计算密集型任务、丰富的交互式 UI 来说非常棒，我们的应用程序甚至会原谅我们没有考虑性能优化。这并不意味着在开发移动应用程序时不应该考虑性能。

我将分享我在 React Native 中应该注意的性能问题方面的经验。这些都是基于在[国际](https://www.internations.org/)使用 React Native 构建大型高性能移动应用程序的经验。我将强调常见的陷阱，并对可能出错的地方以及如何解决问题给出一个大致的理解。🖥 +🕵️‍♀️ +🔧=🚀。

*在构建一个小型 React 原生应用的情况下，遇到性能问题的可能性当然相当低，我不建议过早优化。*

## 反应本地性能的状态

每次父组件收到新的道具或更新其状态时，React per default 都会重新渲染组件，这是 React 构建 UI 的范例更优越的众多原因之一。这也是造成重复渲染浪费的原因。

在 React Native 中，我们可以每秒更新 60 次帧。这为用户提供了流畅的本地体验。随着应用程序复杂性的增加，计算量也在增加。为了达到恒定的 60 FPS，我们需要确保我们的 UI 永远不会被阻塞。不幸的是，JavaScript 运行在一个*单线程*环境中，相比之下 *iOS* 的 *Swift/Objective-C* 或者*Java*/*Kotlin*Android 都支持*多线程*。幸运的是，许多标准的 React 本地组件在一个独立的线程上执行，这个线程被称为*主线程*。这真的很好，因为当 JavaScript 线程繁忙时，UI 仍然会有响应。

一个昂贵的用户交互可能是导航。假设用户正在多个屏幕之间切换。每个屏幕都在执行网络请求、计算数据、屏幕转换和导航动画。所有这些都发生在 JavaScript 线程上，那么应用程序的帧速率会显著下降(取决于复杂性)。我强烈建议将导航完全外包给主线程，这将极大地提高整体性能。寻找原生导航[库](https://github.com/wix/react-native-navigation)解决方案。

在 React Native 中有一个*桥的概念，*它是连接本地世界和 JavaScript 端的东西。他们两个通过以 *JSON* 格式发送和接收数据进行通信。通过网桥来回发送数据可能非常昂贵，因为每次都需要对数据进行*序列化*和*反序列化*。

## 测试性能问题

可以使用*性能监视器*来监视两个线程的当前帧速率，该监视器可以位于[开发者菜单](https://facebook.github.io/react-native/docs/debugging.html#accessing-the-in-app-developer-menu)下。

测量组件渲染的次数是绝对关键的。一个简单的方法就是在你的 *render* 方法中添加一个 *console.count* 调用。

```
render() {
  console.count('component')
  return <Component />
}
```

如果您运行的是 RN 版本 0.57 或更高版本，您可以使用 React Profiler。
参见[文档](https://github.com/facebook/react-devtools/blob/master/packages/react-devtools/README.md)了解如何设置 react-devtools，并阅读这篇[帖子](https://reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html)了解如何配置组件渲染。

最新的 *Android Studio* 和 *Xcode* 版本都提供了评测工具*。尝试在运行时诊断您的内存消耗和 CPU 负载。在应用程序中导航或运行某些活动，以验证内存是否被分配给堆，并在不再使用时再次移除(希望如此)。还要留意 CPU 峰值，以识别计算繁重的任务。请参阅官方文档，了解关于平台特定评测工具 [Android](https://developer.android.com/studio/profile/memory-profiler.html) 或 [iOS](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/MonitoringMemoryUsage.html) 的更多深入解释。*

> 构建软件时不注意内存泄漏会导致软件出现内存泄漏

对于运行内存不足的 android 应用程序，您可能希望通过在 Android 清单文件中添加 *largeHeap* 属性来请求更大的堆大小。点击查看更多详情[。](https://developer.android.com/guide/topics/manifest/application-element.html#largeHeap)

如果您正在将工作从 JavaScript 卸载到本机端，那么通过监控通过网络发送和接收消息的次数或者有效负载的大小来防止网桥过载。 [*消息队列*](https://github.com/facebook/react-native/blob/master/Libraries/BatchedBridge/MessageQueue.js) 让你调用一个间谍方法来观察*桥*上的交通。一个好的起点是识别在桥上发送的包含很少到零大小的有效负载的消息，看看是否可以删除或替换消息的来源。例如，用于包装多个元素的空视图可以替换为 [React 片段](https://reactjs.org/docs/fragments.html)。

```
import MessageQueue from 'react-native/Libraries/BatchedBridge/MessageQueue'MessageQueue.spy(message => console.log('Message', message))
```

## 解决性能问题

用 [*React 保留不必要的重新渲染。PureComponent*](https://reactjs.org/docs/react-api.html#reactpurecomponent) *或*[*memo*](https://reactjs.org/docs/react-api.html#reactmemo)*。*纯组件(不要与无状态函数组件混淆)将对其属性进行浅层比较，以确定是否需要重新渲染。对于更细粒度的控制，你应该使用[*shouldcomponentwupdate*](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)*方法。*

*有时我们希望更频繁地重新呈现，例如，假设您有一个呈现项目列表的组件，该列表从不同的用户输入接收新的*道具*，如分页或下拉列表以刷新其数据。避免*渲染*方法中的内联函数，它们将在每次渲染时被重新创建，并导致组件因属性改变而重新渲染，为了降低成本，命名该函数并将其存储在*渲染*之外。*

```
*class Component extends React.Component {
  renderSectionHeader = () => <View style={...} />render() {
    return (
      <FlatList 
        renderSectionHeader={this.renderSectionHeader}
        {...this.props}
      />
    )
  }
}*
```

*在这篇文章中，了解更多关于如何避免无意义的重新渲染的不同技术和概念。*

*如果您将状态提升到全局级别，那么状态变化的副作用可能会更大。对于我们这些使用 [Redux](https://redux.js.org/) 和 [React Connect](https://redux.js.org/basics/usage-with-react) 作为状态管理工具的人来说，应该注意每次状态改变时*选择器*的重新运行。保护*选择器*免于昂贵计算的一个好方法是缓存它们，这样如果它对状态的引用没有改变，它们就返回状态改变的前一个结果。这可以通过一个名为 [Reselect](https://github.com/reactjs/reselect) 的便利库轻松实现。*

*如果您仍然考虑在 JavaScript 端处理导航，或者您正在与动画中的丢帧现象作斗争。考虑给 [InteractionManager](https://facebook.github.io/react-native/docs/interactionmanager.html) API 一个机会，例如推迟组件的渲染，直到所有用户交互或动画结束。*

```
*class AfterInteractions extends React.Component {state = { interactions: true }

  componentDidMount() {
    InteractionManager.runAfterInteractions(() => {
      this.setState({ interactions: false })
  }render() {
    if (interactions) {
      return null
    }
    return this.props.children
  }
}*
```

*React Native 中的动画看起来很好，也很容易创建。使用*动画*库可以启用*本地驱动程序，*这将在动画开始前通过桥将动画发送到本地端。你的动画将在主线程上独立于被阻塞的 JavaScript 线程执行，这将导致更少的丢帧和更流畅的体验。将 [useNativeDriver](https://facebook.github.io/react-native/blog/2017/02/14/using-native-driver-for-animated.html) 设置为动画配置。*

```
*Animated.timing(
  this.state.fadeAnim, {
    toValue: 1,
    useNativeDriver: true,
  }
).start()*
```

*最后但同样重要的是，图像。减小图像文件大小对保存用户数据计划、加快网络请求和减少内存占用是有意义的。Webp 是一种值得考虑使用的压缩格式。将静态图像转移到本机将减少 JavaScript 包的大小，这是一件好事👍(但是，您将无法再随时为客户提供您的图像)。关于这个话题的更多细节可以在这里找到。*

*在 React Native 中开发可以让您受益于跨目标平台开发。同时为 Android 和 iOS 构建。根据我们的经验，两个平台之间的代码重用是惊人的。iOS 上的内存布局与 Android 不同(自动引用计数而不是垃圾收集),这有助于 iOS 应用程序占用更少的内存。我们已经注意到，iOS 的运行速度比 Android 快很多倍，而且消耗的内存更少。我建议，如果你的目标是在两个平台上发布，那么优先考虑 Android。我希望这不是一件事，但根据我们的经验，如果 Android 运行流畅，那么 iOS 也会。*

***不管是什么平台，在开发期间都应该在真实设备上不断进行测试。**在使用模拟器时，确保在 Android 上激活[硬件加速器](https://developer.android.com/studio/run/emulator-acceleration.html#avd-gpu) it 会大幅提升你模拟器的性能。*

*到目前为止，React Native 在性能和优化工具方面的总体体验是积极的。你边走边学，你的应用程序总会有需要改进的地方。我期待着在 React Native 中继续开发，并看到社区如何继续改进这项伟大的技术。我希望在 React Native 中很快看到的一个改进是支持*多线程*而不需要桥接数据。我可以想象我们在网络上使用的类似于 [*workers*](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) 的东西。*

***我希望你现在已经学到了一些新的东西，这将有助于加速你的原生应用**！*

*我很乐意回答问题或讨论这里提到的任何话题。在下面留言或在 twitter 上找到我。*