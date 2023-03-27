# 使用 React.lazy 和 IntersectionObserver 进行无限组件滚动

> 原文：<https://itnext.io/infinite-component-scrolling-with-react-lazy-and-intersectionobserver-7774c03b08f2?source=collection_archive---------0----------------------->

![](img/198a6581167d94facbcf6949e0b050da.png)

在之前的[帖子](/infinite-scrolling-dynamic-module-loading-the-intersection-observer-api-1f17186c7a8d)中，我简要地写了如何使用 IntersectionObserver API 进行分析和动态模块加载，但是我没有提供任何具体的例子。在这篇文章中，我想实际展示如何用这个相当新的 API 实现上述目标。

**免责声明:**这里我使用 React 与 [Webpack](https://webpack.js.org/guides/code-splitting/) 一起使用，但同样可以在 Vue 以及 Angular 和其他捆绑器如 [parcel.js](https://parceljs.org/code_splitting.html) 中实现。

最近，React 发布了版本 [16.6.3](https://github.com/facebook/react/blob/master/CHANGELOG.md#1663-november-12-2018) ，该版本为我们提供了开箱即用的代码拆分支持，尽管对服务器端渲染的支持不可用，并且有其他实现，但出于我们的目的，我们将使用 React.lazy 和 React . junction。关于 API 细节的更多细节可以参考[此处](https://reactjs.org/docs/code-splitting.html#reactlazy)。

对于那些不熟悉代码分割或在运行时动态加载模块的人来说，理解这样做的主要目的之一是允许开发人员将他们的代码库分割成更小的块，通常由关注点、路径或功能来分隔。按路线划分应用程序是向用户发送较小的包的良好开端，反过来，只根据单个用户的需求向用户发送代码。如果他们从来没有访问过你的应用程序的一部分，无论是出于选择还是出于某种约束，比如权限限制，那么他们不会因为交互时间变慢或数据传输受限而受到惩罚。

React 文档对优势的解释如下…

> 对你的应用进行代码拆分可以帮助你“延迟加载”用户当前需要的东西，这可以极大地提高你的应用的性能。虽然你没有减少应用程序中的总代码量，但你避免了加载用户可能永远不需要的代码，并减少了初始加载期间所需的代码量。

那么，非基于路径或页面的代码分割的实际用例如何呢？当用户在应用程序中导航单个视图或路径时，我们如何动态加载代码呢？

在我目前的公司 [FanAI](http://fanai.io/) ，我们遇到了一个使用积极模块加载的用例，它帮助我们加快了应用程序的性能，并允许我们推迟大量的前期计算。我们发现，随着我们的应用程序增长，并继续变得更加复杂，我们的长滚动视图，其中包含许多不同的数据可视化，开始感到非常缓慢。大多数可视化效果都位于文件夹之下，并不总是与用户交互。此外，许多可视化调用各种 api 端点来获取数据，并且当这些 api 响应被解析并且可视化被呈现时，它们都在竞争动画和布局改变。

随着 React 的加入。我们现在能够按需加载这些不同的可视化，但大多数例子只是使用基于路径的加载或基于用户的动作来触发动态导入。我们需要一种在用户浏览单个视图时拉入代码的方法，更确切地说，是一种在用户滚动时加载代码的方法。这就是我们能够实现和利用 IntersectionObserver api 的地方。

使用 React.lazy 和 React 实现动态代码拆分。悬念是相当直截了当的，[文档](https://reactjs.org/docs/code-splitting.html#reactlazy)更深入地阐述了这个主题，但一个非常简单的实现可以实现如下:

```
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

上面的代码使用 React.lazy，它将一个函数作为唯一的输入，该函数调用一个动态导入，然后该动态导入返回一个解析为包含 React 组件的模块的承诺。

好了，现在让我们进入我们建造的细节。在下面的代码片段中，我们只是在进行设置。我们将默认的相交状态设置为 false，创建一个 ref，它稍后将被用作[目标](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API#Creating_an_intersection_observer)元素，在其中观察与根元素相关的相交。如果没有指定或者如果`null`，根元素又默认为浏览器视窗。

```
state = {
  hasIntersected: false
};targetContainerRef = React.createRef();options = {
  root: this.props.root || null,
  rootMargin: this.props.margin || "0px",
  threshold: this.props.threshold || 0
};observer;
```

接下来我们设置我们的订阅，当交集发生时，它将调用我们的回调`this.load`:

```
componentDidMount() {
  this.observer = new IntersectionObserver(this.load, this.options);
  this.observer.observe(this.targetContainerRef.current);
}componentWillUnmount() {this.observer.unobserve(this.targetContainerRef.current);
}
```

最后，当观察到交叉事件时，我们告诉观察者实际要做什么。我们的第一个条件告诉观察者，一旦出现交叉路口，就停止倾听。这就是我们想要的动态模块加载。

当您希望在每次触发交叉点事件时触发回调时，第二个条件非常有用。我们用它来无限加载图像和搜索结果，因为这允许我们对后端服务进行额外的调用，例如每次用户滚动到列表底部时。对于第二个场景，唯一需要记住的是，您可能需要跟踪滚动的方向，因为您可能只想在相交发生在特定方向时触发回调。

```
load = (entries) => { const { onIntersection, continueObserving } = this.props;if (!continueObserving && !this.state.hasIntersected) {
    const entry = 
      entries.find(
        entry => entry.target === this.targetContainerRef.current
      ); if (entry && entry.isIntersecting) {
      this.setState({ hasIntersected: true });
      onIntersection && onIntersection(entries);this.observer.unobserve(this.targetContainerRef.current);
    }} else if (continueObserving && onIntersection) {
      onIntersection(entries);
  }
};
```

接下来，我们需要将 IntersectionObserver 组件与 React.Suspense 结合使用。

```
<IntersectionObserver>
  <DynamicModule
    placeholder={<Placeholder />}
    component={() => import("./path/to/asyncComponent")}
  />
</IntersectionObserver>
```

上面的代码包装了一个组件，该组件在挂载时调用组件函数 prop。当交集发生时，我们的`<IntersectionObserver />`组件只在内部呈现其子组件。当这种情况发生时，
`<DynamicModule />`组件在挂载时会在内部执行以下操作:

```
componentDidMount() {
  this.setState({
    Component: lazy(this.props.component),
    initializing: true
  });
}
```

上面的代码利用了 React.lazy api，然后它在运行时拉入我们的组件，这只有在用户将该组件滚动到视图中时才会发生！要体验这些组件并了解更多细节，请查看下面的 codesandbox。

除了上面的例子，如果你想看看无限滚动如何与`<IntersectionObserver />`组件一起工作，你可以简单地把它附加到你的可滚动容器的底部，并传递给它一个 onIntersection()回调和一个设置为 true 的 continueObserving 属性，如果你想在每次`<IntersectionObserver />`与你的根元素相交时触发一个回调。

通过实现上述组件，我们能够推迟工作，这反过来又加快了我们的初始页面加载时间，因为我们交付的包要小得多。此外，它提高了应用程序的感知性能，因为我们只在实际需要时加载单个组件模块，而且因为这些模块依赖于外部数据服务，所以模块块的加载和来自 API 的异步响应配合得非常好。对于用户来说，这似乎是一回事，他们无法察觉模块的加载和 api 解析的加载状态之间的差异。这对我们来说是双赢，更重要的是对我们的用户来说！

我希望以上有意义，为了我和整个社区的利益，请随时提供任何反馈。编码快乐！🙌 🙌 🙌