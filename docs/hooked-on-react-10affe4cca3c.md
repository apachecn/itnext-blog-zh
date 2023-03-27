# 迷上了 React

> 原文：<https://itnext.io/hooked-on-react-10affe4cca3c?source=collection_archive---------4----------------------->

![](img/4cf08cc9d89f3a38fe05fd0f9406011b.png)

印度阿姆利则的窗户清洁工(图片由作者提供)

## React 的新钩子 API 如何帮助你赢得阶级战争

**本文假设您已经相当熟悉 React。**

# 什么是钩子？

[钩子](https://reactjs.org/docs/hooks-intro.html)是 JavaScript 函数，让你从普通的函数组件中与 React 组件的状态和生命周期特性进行交互，而不必写一个类，或者将组件包装在更高阶的组件中。

钩子是在 React 16.8 中引入的，现在是一个稳定的特性。钩子在 React Native 的现代版本中也可以工作。

观看 Dan Abramov 在 React Conf 2018 上对 React Hooks 的介绍。

React 提供了许多内置的钩子，并为您编写自己的钩子提供了一套简单的规则。

# useState 挂钩

这个钩子可能是你最常用的钩子。它为您提供了一种简单明了的方式来访问组件的内部状态，而无需将组件声明为`React.Component`的子类。

## 一个例子

老路:

*注意* [*复数*](https://www.npmjs.com/package/pluralise) *的使用是可选的，但我讨厌 UI 显示“1 次”。*

```
import React from 'react'class Counter extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      count: 0
    }
  } increment = () => {
    this.setState({ count: this.state.count + 1 })
  } render() {
    const { count } = this.state
    return (
      <div>
        <p>You clicked {pluralise(count, 'time')}</p>
        <button onClick={this.increment}>Increment</button>
      </div>
    )
  }
}
```

现在使用钩子:

```
import React, { useState } from 'react'const Counter = () => {
  const [count, setCount] = useState(0)

  const increment = () => setCount(count + 1)

  return (
    <div>
      <p>You clicked {pluralise(count, 'time')}</p>
      <button onClick={increment}>Increment</button>
    </div>
  )
}
```

## 这是怎么回事？

```
const [count, setCount] = useState(0)
```

`useState`钩子返回两个东西，一个是你关心的状态，另一个是设置状态值的函数。在上面的例子中，我们关心一个叫做`count`的状态，我们给了它一个默认值`0`，我们希望能够设置那个`count`的值。值和函数在一个两元素的数组中返回，所以我们使用*数组析构*作为分配这些值的简写。

# useEffect 挂钩

`[useEffect](https://reactjs.org/docs/hooks-effect.html)`钩子允许你在函数组件中执行副作用。你可以把`useEffect`钩子想象成`componentDidMount`、`componentDidUpdate`和`componentWillUnmount`的组合。

## 例子

假设我们想在`count`增加时更新页面标题。

老方法是:

```
import React, { Component, Fragment } from 'react'class Counter extends Component {
  constructor(props) {
    super(props)
    this.state = {
      count: 0
    }
  } setTitle = count => {
    document.title = `You clicked ${pluralise(count, 'time')}`
  } increment = () => {
    this.setState({ count: this.state.count + 1 })
  } componentDidMount() {
    this.setTitle(this.state.count)
  } componentDidUpdate() {
    this.setTitle(this.state.count)
  } render() {
    const { count } = this.state
    return (
      <Fragment>
        <p>You clicked {pluralise(count, 'time')}</p>
        <button onClick={this.increment}>Increment</button>
      </Fragment>
    )
  }
}
```

但是使用钩子:

```
import React, { Fragment, useState, useEffect } from 'react'const Counter = () => {
  const [count, setCount] = useState(0)

  const increment = () => setCount(count + 1)

  useEffect(() => {
    document.title = `You clicked ${pluralise(count, 'time')}`
  }) return (
    <Fragment>
      <p>You clicked {pluralise(count, 'time')}</p>
      <button onClick={increment}>Increment</button>
    </Fragment>
  )
}
```

使用`useEffect`钩子告诉 React 组件在渲染之后需要做一些事情*。React 将记住您传递给`useEffect`的函数，并在执行 DOM 更新后调用它。*

注意:传递给`useEffect`的函数会在每次渲染时重新创建。这确保了`count`永远不会过时。每次重新渲染组件时，都会安排不同的效果，替换之前的效果。

## 如何防止效果在没有变化时触发

`useEffect`钩子接受第二个参数，用来和它的当前状态进行比较。即使`count`没有改变，上面的例子也会重写`document.title`。为了解决这个问题，我们调用了`useEffect`，但是也传入了`count`。

```
useEffect(
  () => {
    document.title = `You clicked ${pluralise(count, 'time')}`
  },
  [count]
)
```

现在，如果`count`不变，则不会调用效果功能。

**提示**:你可以通过传入`[]`作为第二个参数来强制一个效果只运行一次，从而模拟`componentDidMount`的生命周期。

## 非阻塞效应与阻塞效应

大多数效果不需要同步发生，与`componentDidMount`或`componentDidUpdate`不同，使用`useEffect`安排的效果不会阻止浏览器更新屏幕。在极少数情况下，您需要确保在组件完成渲染后应用效果，有一个单独的`useLayoutEffect`钩子，它的 API 与`useEffect`相同。

## 效果后的清理。

提供给`useEffect`的效果函数可以返回一个可选的清理函数，该函数在组件卸载时执行。React 还会在下一次渲染中运行效果之前清除上一次渲染的效果。

# React 提供的其他标准挂钩

## 使用上下文

`[useContext](https://reactjs.org/docs/hooks-reference.html#usecontext)`钩子接受一个`context`对象(从`React.createContext`返回的值)并返回那个`context`的当前值。当前上下文值由树中调用组件上方最近的`<MyContext.Provider>`的`value`属性决定。

当组件上方最近的`<MyContext.Provider>`更新时，这个钩子将触发一次重新呈现，将最新的上下文值传递给那个`MyContext`提供者。

## 用户教育

`useState`的替代品。`useReducer`吊钩接受一个 Redux 类型的减速器

```
(state, action) => newState
```

并用一个`dispatch`方法返回当前状态。可以把它想象成一个特定于组件的迷你 Redux。

## 使用回调

传递一个回调和一个依赖数组。`useCallback`将返回一个*内存化的*版本的回调，只有当其中一个依赖关系改变时才会改变。这在将回调传递给依赖引用相等的优化子组件以防止不必要的呈现时非常有用。

参见:[如何记忆 React 中的组件](https://medium.com/@rossbulat/how-to-memoize-in-react-3d20cbcd2b6e)。

## 使用备忘录

传递一个`create`函数和一个依赖数组。`useMemo`仅当其中一个依赖关系改变时，才重新计算 memoised 值，避免每次渲染时重新计算。

## useRef

返回一个*可变* `ref`对象，其`.current`属性被初始化为传递的参数`(initialValue)`。返回的对象将在组件的整个生存期内保持不变。

如果您将一个`ref`对象传递给 React with `<div ref={myRef} />`，React 将在 DOM 节点发生变化时将`ref.current`属性设置为相应的 DOM 节点。

在[React 文档](https://reactjs.org/docs/refs-and-the-dom.html)中阅读更多关于`ref`属性的用法。

**提示**:你可以使用，`useRef`作为超过`ref`的属性。这有助于保留任何可变值，类似于在类中使用实例字段。

## useLayoutEffect

与`useEffect`相同，但它在所有 DOM 突变后同步触发*。使用它从 DOM 中读取布局并同步重新渲染。在浏览器有机会绘制之前，安排在`useLayoutEffect`中的更新将被同步刷新。*

***提示**:尽可能使用标准的`useEffect`，以避免阻碍视觉更新。*

## *useDebugValue*

*用于在 React DevTools 中显示自定义挂钩的标签。*

# *钩子代替 Redux 吗？*

*不会。Redux 仍然是管理共享或应用程序范围状态的优秀框架。React Redux [从 v7.1.0](https://react-redux.js.org/api/hooks) 开始就支持钩子 API，并公开诸如`useDispatch`和`useSelector`之类的钩子，使您能够`subscribe`到 Redux 存储和`dispatch`动作，而不必将组件包装在`connect`中。*

*`useSelector`钩子大致相当于`connect`的`mapStateToProps`参数。将以整个 Redux 存储状态作为唯一参数来调用选择器。每当组件呈现时，它都会运行。`useSelector`还将订阅 Redux store，并在调度动作时运行您的选择器。*

## *例子*

*我们希望许多计数器组件共享相同的计数。定义一个简单的重复计数`state`、`reducer`和`store`，如下所示:*

```
*import { createStore, combineReducers } from 'redux'// count starts at 0
const INITIAL_COUNT = 0// the count state reducer
const count = (state = INITIAL_COUNT, action = {}) => {
  switch(action.type) {
    case 'increment-count': return state + 1
    default: return state
  }
}// export the redux store
export default createStore(combineReducers({ count }))*
```

*一个`Count`组件只是从使用`useSelector`的状态中获取`count`并显示在一个`span`中:*

```
*import React from 'react'
import { useSelector } from 'react-redux'const Count = () => {
  const count = useSelector(({ count }) => count)
  return <span>{count}</span>
}*
```

*`useDispatch`钩子从 Redux 存储中返回对`dispatch`函数的引用。您可以根据需要使用它来调度操作。*

## *例子*

*从上方使用`Count`组件。*

```
*import React, { Fragment } from 'react'
import { useDispatch } from 'react-redux'
import Count from './Count'const Counter = () => {
  const dispatch = useDispatch()
  const increment = () => dispatch({ type: 'increment-count' }) return (
    <Fragment>
      <Count />
      <button onClick={increment}>Increment</button>
    </Fragment>
  )
}*
```

*当使用`dispatch`将回调传递给子组件时，建议使用`useCallback`进行调用，否则子组件可能会由于引用的改变而出现不必要的渲染。*

```
*import React, { Fragment, memo, useCallback } from 'react'
import { useDispatch } from 'react-redux'
import Count from './Count'const Incrementer = memo(({ onClick }) => (
  <button onClick={onClick}>Increment</button>
))const Counter = ({ count }) => {
  const dispatch = useDispatch()
  const increment = useCallback(
    () => dispatch({ type: 'increment-count' }),
    [dispatch]
  ) return (
    <Fragment>
      <Count />
      <Incrementer onClick={increment} />
    </Fragment>
  )
}*
```

# *React 路由器怎么样？*

*React 路由器[从 v5.1 开始就支持钩子 API](https://reacttraining.com/react-router/web/api/Hooks) 。*

## *使用历史*

```
*import { useHistory } from 'react-router-dom'export const HomeButton = () => {
  const history = useHistory() const goHome = () => history.push('/home') return (
    <button type="button" onClick={goHome}>Home</button>
  )
}*
```

## *React 路由器暴露的其他挂钩:*

*   *`useLocation`*
*   *`useParams`*
*   *`useRouteMatch`*

# *测试呢？*

*为你的定制钩子编写标准的单元测试；钩子只是函数。*

*从 React 的角度来看，使用钩子的组件是常规组件。您可以像往常一样测试它，如果您的钩子做了一些复杂的事情，您可以独立地测试钩子，并从组件的测试中模拟它，就像您对任何其他依赖项所做的那样。*

*更多信息见[reactjs.org/docs/testing-recipes.html](https://reactjs.org/docs/testing-recipes.html)。*

# *“钩子的规则”*

*为了让钩子工作[，它们需要遵循三个简单的规则](https://reactjs.org/docs/hooks-rules.html)。*

***首先**:你的钩子必须以`use`这个字开头。*

***其次**:只从 React 函数调用钩子，或者从其他钩子调用。*

*不要从常规的 JavaScript 函数中调用钩子。*

***注意**:你不能在一个类组件中使用钩子，但是当你从旧风格`React.Component`类中移植你的应用时，你*可以在一个单独的树中用钩子混合类和函数组件。从长远来看，React 团队期望带挂钩的简单函数成为人们编写 React 组件的主要方式。**

*第三个:只在你的函数的顶层调用钩子。*

*不要从内部循环、条件或嵌套函数中调用钩子，否则每次组件渲染时，钩子的调用顺序都不一样。*

*有一个名为`[eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks)`的`ESLint`插件执行这些规则。*

***注意**:如果你正在使用`Create React App`，那么你会看到`[eslint-plugin-react-hook](https://www.npmjs.com/package/eslint-plugin-react-hooks)s`已经包含在`react-scripts`的最新版本中。*

# *定制钩子——编写自己的钩子*

*通过[将逻辑提取到一个钩子](https://reactjs.org/docs/hooks-reference.html)，在两个 JavaScript 函数之间共享逻辑；一个名字以`use`开头的 JavaScript 函数，可以调用其他钩子。*

*您的自定义钩子不需要有特定的签名。您可以决定它接受什么作为参数，以及它返回什么(如果有的话)。这就像一个普通的函数。它的名字*必须*以`use`开头，这样你一眼就能看出挂钩的*规则适用于它。**

## *例子*

*让我们假设任何一个自重的计数器都需要显示一个当前的`count`，能够`increment`那个`count`，并在当前的`count`改变时用当前的`count`更新页面标题。(这当然假设每页只有一个`count`)。我们可以这样写一个`useCount`钩子:*

```
*export const useCount = () => {
  const [count, setCount] = useState(0)
  const increment = () => setCount(count + 1) useEffect(() => {
    document.title = `You clicked ${pluralise(count, 'time')}`
  }, [count]) return [count, increment]
}*
```

*那么`Counter`组件可以写成:*

```
*const Counter = () => {
  const [count, increment] = useCount() return (
    <Fragment>
      <p>You clicked ${pluralise(count, 'time')}</p>
      <button onClick={increment}>Increment</button>
    </Fragment>
  )
}*
```

*每次你使用一个自定义钩子，它里面的所有状态和效果都被完全隔离。在这个例子中，每个使用`useCount`钩子的组件都有自己的`count`状态，从`0`开始。如果您希望在组件之间共享状态，您可以使用 Redux，或者使用许多全局状态管理钩子中的一个(例如`[useGlobalHook](https://medium.com/javascript-in-plain-english/state-management-with-react-hooks-no-redux-or-context-api-8b3035ceecf8)`)。)*

*定制钩子以以前在 React 组件中不可能的方式实现了逻辑的封装和共享。它们可以覆盖广泛的用例，比如[表单处理](https://react-hook-form.com)、[动画](https://www.react-spring.io/docs/hooks/basics)、 [API 调用和订阅](https://codeburst.io/how-to-fetch-data-from-an-api-with-react-hooks-9e7202b8afcd)、[定时器](https://github.com/amrlabib/react-timer-hook)等等。*

*在野外已经有数以千计有趣的 React 钩子可供你使用。*

# *赢得阶级斗争*

*有经验的 JavaScript 开发人员知道使用类最多是有问题的，并把关键字`this`的存在视为代码气味。JavaScript 并不是真正的面向对象语言，不必要的尝试使用它可能会导致难以发现的细微错误。*

*钩子允许你通过简化你的组件来避开这些危险，而不是强迫组件接受`React.Component`(或者`React.PureComponent`)的秘密，或者在由高阶组件组成的深树中纠缠不清。*

# *最后*

*使用钩子。未来你会爱你的。*

*未来的我恨我(官方音乐视频)*

# *链接*

*链接按照它们在文章中出现的顺序显示。*

*   *[https://reactjs.org/docs/hooks-intro.html](https://reactjs.org/docs/hooks-intro.html)*
*   *[https://reactjs.org/docs/hooks-state.html](https://reactjs.org/docs/hooks-state.html)*
*   *[https://www.npmjs.com/package/pluralise](https://www.npmjs.com/package/pluralise)*
*   *[https://reactjs.org/docs/hooks-effect.html](https://reactjs.org/docs/hooks-effect.html)*
*   *【https://reactjs.org/docs/hooks-reference.html#usecontext】*
*   *[https://medium . com/@ Ross bulat/how-to-memo ize-in-react-3d 20 cbcd 2 b 6 e](https://medium.com/@rossbulat/how-to-memoize-in-react-3d20cbcd2b6e)*
*   *[https://reactjs.org/docs/refs-and-the-dom.html](https://reactjs.org/docs/refs-and-the-dom.html)*
*   *[https://react-redux.js.org/api/hooks](https://react-redux.js.org/api/hooks)*
*   *https://reacttraining.com/react-router/web/api/Hooks*
*   *【https://reactjs.org/docs/testing-recipes.html *
*   *[https://www.npmjs.com/package/eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks)*
*   *[https://reactjs.org/docs/hooks-rules.html](https://reactjs.org/docs/hooks-rules.html)*
*   *[https://reactjs.org/docs/hooks-custom.html](https://reactjs.org/docs/hooks-custom.html)*
*   *[https://reactjs.org/docs/hooks-reference.html](https://reactjs.org/docs/hooks-reference.html)*
*   *[https://medium . com/better-programming/here-are-6-awesome-react-hooks-2ff 0 c0b 35218](https://medium.com/better-programming/here-are-6-awesome-react-hooks-2ff0c0b35218)*
*   *[https://medium . com/JavaScript-in-plain-English/state-management-with-react-hooks-no-redux-or-context-API-8b 3035 CEE cf 8](https://medium.com/javascript-in-plain-english/state-management-with-react-hooks-no-redux-or-context-api-8b3035ceecf8)*
*   *[https://react-hook-form.com](https://react-hook-form.com)*
*   *[https://www.react-spring.io/docs/hooks/basics](https://www.react-spring.io/docs/hooks/basics)*
*   *[https://github.com/amrlabib/react-timer-hook](https://github.com/amrlabib/react-timer-hook)*
*   *[https://code burst . io/how-to-fetch-data-from-an-API-with-react-hooks-9e 7202 b 8 afcd](https://codeburst.io/how-to-fetch-data-from-an-api-with-react-hooks-9e7202b8afcd)*
*   *[https://nikgraf.github.io/react-hooks](https://nikgraf.github.io/react-hooks/)*

*—*

*像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。*