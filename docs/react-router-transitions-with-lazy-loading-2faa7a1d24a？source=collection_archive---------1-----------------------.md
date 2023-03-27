# 通过延迟加载对路由器转换做出反应

> 原文：<https://itnext.io/react-router-transitions-with-lazy-loading-2faa7a1d24a?source=collection_archive---------1----------------------->

![](img/7dad631239b85609a436bf91f815d7a0.png)

[https://tico times . net/2015/10/17/11-鲜为人知-树懒-事实](https://ticotimes.net/2015/10/17/11-little-known-sloth-facts)

本指南建立在我之前的指南之上，在那里我解释了如何用 react-router-dom 和 react-pose 获得路线转换/动画。

[](https://medium.com/capgemini-norway/the-holy-grail-of-react-router-transitions-on-the-web-a1293591e34f) [## React 路由器在网络上过渡的圣杯

### 去年我一直在用 react-router-dom 和 react-pose 完善路由转换。本周我能够…

medium.com](https://medium.com/capgemini-norway/the-holy-grail-of-react-router-transitions-on-the-web-a1293591e34f) 

这是一个相当简单的设置。当您开始扩展您的应用程序，并且您的代码库开始增长时，您可能希望向组件添加一些延迟加载。有了 Create React App，这真的很简单。您只需添加一个暂记包装器，并以异步方法导入组件:

```
*const Lazy*Component = React.lazy(() => *import*("./Component"));return (
  <React.Suspense *fallback*={<div />}>
    <LazyComponent />
  </React.Suspense>
)
```

但是，当您刷新页面时，这会破坏初始的路由动画，因为异步操作占用了 JavaScript 中的单线程运行时间。

那么我们能做些什么来解决这个问题呢？解决方案是在早期预加载组件。这实际上是我找到的处理这个问题的唯一解决方案，所以请耐心等待，因为这可能看起来像一个黑客。

# 在添加预加载之前

单击“Two”页面，看看它是如何跳过动画的，因为组件需要在实际动画之前预加载。

# 添加预加载后

我们仍然得到懒惰加载的组件，动画是完整的。

# 履行

本质上，您需要在渲染优先级的早期将所有惰性加载的组件加载到一个隐藏的 div 中。在这种情况下，明智的做法是将路由路径和组件放在一个数组的对象中。这样，您可以将它们导出到实际的 React 路由器<switch>组件和 PreloadLazyComponents.js 文件中。</switch>

这个实现的超时是为了在我们知道所有组件都已经实现时从 DOM 中删除它们。此外，我们将把*预载*道具传递给所有被预载的组件。这是为了控制它不在子组件(如页面 1)中执行任何 API 请求。

假设 Page1 有一个调用 API 的 useEffect。您不希望过早地这样做，所以 useEffect 必须重写为

```
!preload && useAsyncFetch("/api")
```

此外，您的组件可能依赖于 Redux 中尚未使用的数据。因此，您必须确保组件不会在没有数据的情况下崩溃。我建议您在渲染任何其他组件之前先渲染 PreloadLazyComponents。然而，这可能意味着您没有在预加载的组件中准备好数据。我通过检查变量是否存在来解决这个问题。

```
const Page1 = () => {
  const user = useSelector(state => state.user);if (!user) return null;
  else return <div> page 1 says hi to { user.name }! </div>
}
```

或者

```
const Page1 = () => {const [ data, setData ] = React.useState(null)
  const user = useSelector(state => state.user)React.useEffect( () => {
    setData( user ? fetchDataByUserId(user.id) : null )}return <div>{data}</div>}
```

基本上，您需要确保您的组件可以在没有数据的情况下呈现。我通过使用 react-testing-library 为我的所有组件添加测试来解决这个问题，在我的所有 reducers 中都有一个空状态。当然，您可以使用 Enzyme 或您喜欢的库来测试这一点。只需呈现没有任何预期状态的组件，并断言它不会崩溃。

```
test(`<Overview/> renders without data`, *async* () => {
    *// All reducer states are initial and empty
    const* store = mockStore(mockInitialReducer);     *const* { container } = render(
        <ReduxProvider *store*={store}>
            <Router *history*={history}>
                <React.Suspense *fallback*={<div />}>
                   <Overview />
                </React.Suspense>
            </Router>
        </ReduxProvider>
    );
    expect(container).toBeInTheDocument();
});
```

# 结论

我知道很多人认为这是过度设计了。不得不检查懒惰加载的组件上的所有变量只是混乱的代码。在应用程序的早期加载大量的组件，只是为了移除它们，这似乎是一件麻烦的事情。然而，如果你想延迟加载组件，并在每次导航时保持你的漂亮动画，这是我通过询问这些库的维护者找到的唯一解决方案。如果你有更好的解决方案，我很乐意看到或听到他们。