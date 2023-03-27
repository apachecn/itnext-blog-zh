# 在 React 应用中提高 Redux 性能的 3 个小技巧

> 原文：<https://itnext.io/3-small-tips-for-better-redux-performance-in-a-react-app-9cde549df6af?source=collection_archive---------0----------------------->

![](img/8ea1c1336429e73ed105a6baac995965.png)

在我的[上一篇文章](https://medium.com/@aggelosarvanitakis/6-tips-for-better-react-performance-4329d12c126b)中，我介绍了 React 应用中性能优化的一些核心概念。本文将通过关注实践来扩展这些概念，这些实践将确保在您的项目中引入 Redux 不会在您的应用程序增长时导致任何性能瓶颈。

正如安德鲁[有趣地指出的那样](https://twitter.com/acdlite/status/1024852895814930432?lang=en)，Redux 本身只是一个事件发射器。因此，所有可能的优化都被自动委托给反应本身。在 React 中，我们将优化转化为整体渲染的**渲染时间减少**和**。**前者与组件渲染阶段的工作量有关(昂贵的计算、多次函数调用、自定义 DOM 突变等)。)，而后者与组件渲染的次数有关。React 组件内部的工作量与业务逻辑紧密相关，并且很可能与 redux 无关，但是呈现的数量很容易与 redux 相关，并且如果不小心的话会逐渐增加。

所以，我们开始吧:

## 1.在你的选择器中利用记忆

选择器在 redux 中并不是什么新东西，它们只是从读取它的组件中抽象出实际状态实现的一种方式；组件和实际 redux 状态之间的代理。编写选择器可以像这样简单:

“items”键的一个简单的冗余选择器

在上面的例子中，我们假设数据存储在一个列表中。Redux docs 虽然，[推荐状态被规范化](https://redux.js.org/recipes/structuring-reducers/normalizing-state-shape)。这意味着我们应该将项目存储在一个对象中，其中键是项目的 id，值是项目本身。为了获得与上面例子相似的结果，我们会天真地重写一个选择器(假设条目的顺序无关紧要):

```
// remember, items is an object in this case
export const getItems = state => Object.keys(state.items);
```

这个实现实际上服务于我们的目的，但是有一个大的缺陷。任何时候**Redux 状态**的任何部分被更新，我们的组件将重新呈现，不管存储更新是否与我们的组件相关。你看，当你使用 react-redux 的`connect()` HOC 连接一个组件时，redux 重新计算已经提供给`connect()`的`mapStateToProps`的输出，并执行一个浅层的相等检查。如果存储更新前后的属性在引用上是相同的，则组件不会重新呈现。如果它们不同，那么组件将重新呈现。在我们的例子中，`Object.keys`每次被调用时都返回不同的实例。因此，由于这个选择器在我们的`mapStateToProps`中，它将在每次执行存储更新时被重新评估。仅仅因为这个数组的引用将不同于它的前一个，react-redux 将错误地认为组件的属性是不同的，因此它将“允许”react 重新呈现。

像`.map`和`.filter`这样的操作符每次被调用时都返回不同的引用值。因此，我们必须确保当实际输出相同时，基准电压保持不变；当实际输出不同时，基准电压也应该不同。为此，我们采用了一种叫做记忆化的技术。这种技术保证了“只要函数的输入是相同的，那么输出将是相同的”。一个流行的库选项是[重新选择](https://github.com/reduxjs/reselect)。有了它，我们可以像这样重写我们的选择器:

```
import { createSelector } from 'reselect';export const getItems = 
  createSelector(state => state.items, items => Object.keys(items))
```

`createSelector`接受任意数量的函数参数，其中前 N-1 个函数是选择器的“输入”，第 N 个是“输出”。它保证只要输入具有相同的值，输出就会被“缓存”和重用。如果任何输入的值与上次选择器调用时的值不同，那么将重新计算第 n 个函数的“输出”。在我们的例子中，只要项目保持不变，那么无论调用多少次,`getItems`选择器都将返回一个具有相同引用的值。如果添加了新的条目，那么`items`状态将不再具有相同的值，因此`getItems`将返回一个新的引用，该引用将被缓存和重用。

作为一个规则，每当您需要在每次选择器调用时执行返回新引用的操作时，请确保您记住了结果(提示:在处理复杂的选择器逻辑时，您可以在另一个记忆选择器中使用一个记忆选择器)。

## 2.使用参照一致的动作创建者

有时动作创建器是简单的函数，比如:

```
const fireAction = () => ({ type: 'ACTION', payload: {} });
```

在上面的例子中，`fireAction`有一个固定的引用，但不幸的是并不总是这样。有时候，您需要将组件的`props`传递给动作创建者，以便区分传递给缩减器的数据。例如，让我们来看下面这个案例:

使用组件道具的动作创建器

上面的实现有点幼稚。我们希望将`fireAction`动作创建者“绑定”到`MyComponent`作为道具的特定`item`。为了实现这一点，我们从`mapDispatchToProps`内部“读取”道具，并使用它们来创建`fireActionWithItem`。有些人不知道的是，每当你从你的`mapDispatchToProps`中使用组件的道具时(每当你使用这个函数的第二个参数时)，react-redux 会在每次组件的道具改变时重新计算`mapDispatchToProps`的输出。因此，如果来自`MyComponent`的任何道具发生变化，那么`mapDispatchToProps`将再次运行。由于匿名函数的性质，作为道具传递给`MyComponent`的`fireActionWithItem`每次都会有不同的引用。这对于正在渲染的组件来说不是问题(因为不管在我们的`mapDispatchToProps`中发生了什么，它都会重新渲染)，但是对于在`MyComponent`中声明的将`fireActionWithItem`作为道具的其他组件来说，这可能会产生问题。例如，让我们看看下面的场景:

昂贵的组件无缘无故地重新渲染…

我们有两个组件:`MyComponent`和`ExpensiveComponent`。如你所见，`ExpensiveComponent`被包裹在`memo()`中，所以只有当它的道具改变时才会重新渲染。每次`MyComponent`中的`randomProp`改变，那么`mapDispatchToProps`将被重新计算，`fireActionWithItem`将获得新的参考，`MyComponent`将被重新渲染，`ExpensiveComponent`也将被强制重新渲染，因为它的`onClick`道具将会不同。最终，`ExpensiveComponent`根本不应该被重新渲染，因为没有任何影响它的东西实际上已经改变了。

为了应对这种情况，您可以将带有正确参数的动作触发委托给组件本身(通过将`item`和`fireAction`作为道具传递给组件)，允许它使用固定引用“创建”`fireActionWithItem`(通过使用`.bind` & `useCallback`)，或者您可以在动作创建器中使用记忆化技术。对于后者，您可以使用与选择器中相同的概念，并确保引用总是相同的。两种选择都可以在下面的要点中看到:

参照一致动作创建者的两种方法

## 3.减少渲染次数的批处理操作

每当你不得不一个接一个地链接多个动作时(你**有意**在你的动作中分开关注点)，你最好批处理它们并用一个`dispatch`发射它们。对这些动作进行分组可以保证将只有 1 次而不是 X 次可能的重新渲染。不过有一个警告:为了使操作“可批处理”，它们需要相互独立。这意味着 N 动作必须独立于 N-1 动作所创建的状态。换句话说，您应该能够在不影响结束状态的情况下更改这些操作的触发顺序。如果是这样的话，`react-redux` 7.x.x 提供了一个新的`batch`功能，可以帮助你只通过一个调度来更新你的状态，为你节省额外的——可能是昂贵的——渲染。

```
import { batch } from 'react-redux'
import { action1, action2 } from './actions';// dispatches both actions at the same time
batch(() => {
  dispatch(action1());
  dispatch(action2());
});
```

为了使用它，需要访问调度功能。有很多方法可以实现这一点，这取决于从哪里触发操作。例如，您可以从您的`mapDispatchToProps`函数、从`redux-thunk` thunk 或者从手动导入`store`并使用`store.dispatch`来访问`dispatch`。这主要归结于你的应用程序是如何构建的。不得不说，这种技术很少使用，但在主线程的自由度至关重要的情况下，它可以提高性能。例如，如果您在启动这些动作后正在执行繁重的动画，那么单个调度可能会为您节省宝贵的帧。

# 结论

我故意没有触及正确组织你的状态的话题，这比上面的建议更重要。本文的目的是揭示 React 应用程序中动作创建者和选择器的非最优集成可能导致的性能损失。我绝不相信所有项目都应该使用这些概念——因为对大多数项目来说它们都是多余的——但是知道这些概念是有好处的。

谢谢:)

*👋 ***嗨，我是***[***Aggelos***](https://aggelosarvanitakis.me)***！如果你喜欢这个，考虑一下*** [***在 twitter 上关注我***](https://twitter.com/AggArvanitakis) ***并与你的开发者朋友分享这个故事*😀***