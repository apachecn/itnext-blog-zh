# React 函数范式的顶峰——第 2 部分:React 最糟糕的部分

> 原文：<https://itnext.io/the-zenith-of-reacts-functional-paradigms-part-2-the-worst-parts-of-react-6011b85ba6a1?source=collection_archive---------6----------------------->

![](img/3a04624fe5369480c38bd774ecdb30ee.png)

本杰明·罗宾·叶斯柏森在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

功能性 React 代码库的两个主要构建块——无状态功能组件和高阶组件——都有严重的缺陷。

# 无状态功能组件的限制

我之前已经[发表过我对 SFCs](https://medium.com/@adamterlson/functional-react-series-part-1-get-your-app-outta-my-component-92656ae13e25) 的热爱。然而，有几个原因使我不能像我希望的那样充分利用它们。

在 React Native 中，性能优化的需求不是“是否”的问题，而是“何时”的问题——消除不必要的重新渲染是“React 优化 101”。

基准测试不会说谎:[sfc 是](https://moduscreate.com/blog/react_component_rendering_performance/) [而不是](/react-component-class-vs-stateless-component-e3797c7d23ab) [最快的选项](https://medium.com/groww-engineering/stateless-component-vs-pure-component-d2af88a1200b)。特别是 React 本地应用的用户可能会遭受*真正可察觉的*性能差异，仅仅因为工程师选择利用函数而不是类。

这个世界不应该是这样的！

工程师们别无选择:使用 PureComponent(或者用一个 HOC 包装每个组件，它本身又会使用 PureComponent)。

# 特设组件组成烂透了

hoc 是为 React 组件添加功能的可靠选择。需要访问状态、转换道具、访问上下文、调度一个动作，或者任何你能想象到的事情，hoc 可以是这些行为的描述方式(参见 [recompose](https://github.com/acdlite/recompose) 之类的库的流行程度)。

然而，走得太远了，你最后会留下一点混乱:

```
const FinalComponent = compose(
    connect(...)
    mapProps(...),
    dispatchOnMount(...),
    defaultProps(...),
    ...20 HOCs later...,
    pure,
)(IncompleteComponent)
```

除了简单地作为一个跟踪/调试的 PITA 之外，通过组件组合来实现行为定义有一些严重的缺陷，这些缺陷并不那么明显。

## 问题 1:属性和渲染封送处理

因为道具必须从一个组件流到下一个组件，所以每个 HOC 都有责任确保它们不被修改和中断地通过。

```
compose(
  withState,
  myCustomHOC, // Must pass all state changes through, uninterrupted
)(MyComponent)
```

除了简单地成为一类没人希望出现的新错误之外，这还意味着实际上不可能在特设中应用 shouldComponentUpdate 毕竟，阻止特设重新呈现将阻止下游组件从上游接收适当的更改。

## 问题 2:道具上的全有或全无型安全

在一个完美的世界中，当用 JSX 编写一个呈现方法时，类型系统将确保你正在使用的组件的所有适当的需求将被满足(并且希望有工作的智能感知)。

```
const MyComponent = compose(
    someHOCA, // Requires prop "propA"
    someHOCB, // Requires prop "propB"
)((props: { propC: string }) => (...)) <MyComponent propA={10} propB={false} /> // Error - PropC is missing
```

但是，为了用 HOC 组件组合实现这一点，类型系统必须能够完全理解每个 HOC 以及它对 props 有什么影响(如果有的话)。

```
const MyComponent = compose(
    someHOCA, // Requires prop "propA"
    someHOCB, // Requires prop "propB"
    propMap(props => ({ ...props, propC: "Yes!" }), // Add propC
)((props: { propC: string }) => (...)) <MyComponent propA={10} propB={false} /> // No error (hopefully)!
```

当试图创建一个具有良好类型覆盖和良好风格的 React 项目时，这种复杂性(参见:开销)会产生很多问题。

说到类型，要么全有，要么全无。生或死。杀人或被杀——你明白了。

如果您错过了，请前往[查看第 1 部分](https://medium.com/@adamterlson/react-hooks-memo-the-zenith-of-reacts-functional-paradigms-8067dc258afb)了解更多关于 React 函数范式的历史。

# 第三部分:很快

我将讨论这些问题是如何解决的，以及我所看到的函数式 react 代码库的惯用前进方式。