# 用 15 行代码编写 Redux

> 原文：<https://itnext.io/writing-redux-in-15-lines-of-code-6d317b521719?source=collection_archive---------5----------------------->

**2020 年更新:**你可能有兴趣学习如何从头开始构建一个商店，它使用 Redux Devtools，并使用 React 挂钩在任何地方访问它:

[](https://medium.com/@dominictobias/using-react-hooks-for-global-redux-stores-97c092afe210) [## 对全局 Redux 存储使用 React 挂钩

### 当 hooks 问世时，有各种各样的文章暗示 Redux 可能不再需要，而像这样的反驳…

medium.com](https://medium.com/@dominictobias/using-react-hooks-for-global-redux-stores-97c092afe210) 

我今天写这篇文章的灵感来自于一位新加入 React 的同事，他抱怨说在使用 Redux(和 Redux-Saga)时有太多的样板文件。所以我把这些放在一起有几个原因:

1.  我们错误地认为冗长对于单向数据流是必要的。
2.  我们使用 Redux 并不了解它是如何工作的。

对于我们的简单 Redux 商店，我们需要以下功能:

*   更新商店状态的函数(因为我们需要在状态改变时通知订户)——***更新(storeKey，updateFn)***
*   订阅更新功能— ***【订阅(fn)***
*   获取内部状态的函数(这不是必需的，但鼓励用户不要直接操作状态对象)——***getState()***

```
class Store {
  constructor(initialState) {
    this.state = initialState;
    this.subscriptions = [];
  }
  update(storeKey, updateFn) {
    const nextStoreState = updateFn(this.state[storeKey]);
    if (this.state[storeKey] !== nextStoreState) {
      this.state[storeKey] = nextStoreState;
      this.subscriptions.forEach(f => f(store));
    }
  }
  subscribe = fn => this.subscriptions.push(fn)
  getState = () => this.state
}
```

*注意没有换行符来保留我吸引人的标题*

很简单，对吧？现在，每次商店更新时，我们都希望调用 *ReactDOM.render* 来重新呈现我们的应用程序。稍后我们可以添加一个类似于 *redux-connect* 的特设组件来包装我们的根组件以处理更新，并在应用程序中的任何地方传递商店道具(第二部分有人知道吗？).

让我们通过一个简单的演示来看看这一点。

你可能已经注意到了下面的方法:

```
increaseUserAge = () => {
  this.props.store.update('user', state => ({
    ...state,
    age: state.age + 2,
  }));
}
```

# 听着妈妈，不要行动！

然而，数据仍然是单向流动的。没有什么能阻止我们将更新提取到可重用且易于测试的功能中:

```
// Somewhere else.
function fetchUserDetailsAction(store) {
  const { user } = store.getState(); if (!user.fetched) {
    $get('/user/123').then(user => store.update('user', user));
  }
}// In your Component
componentDidMount() {
  fetchUserDetailsAction(this.props.store);
}
```

# “但我喜欢减速器和行动🤬"

如上所述，可以提取复杂的状态更新，而无需采取行动。但是我也喜欢减速器。它们将每个存储节点的数据操作保存在一个地方。

没有什么能阻止我们拥有一个`reducers/user.js`文件:

```
const userReducer = {
  increaseUserAge: (state, ageIncrease) => ({
    ...state,
    age: state.age + ageIncrease,
  }),
  setUserEmail: (state, email) => ({
    ...state,
    email,
  }),
};
```

我们可以用这个来代替:

```
increaseUserAge = () => {
  this.props.store.update('user', state =>
    userReducer.increaseUserAge(state, 2));
}
```

动作也有超出逻辑封装的目的；**时间旅行**。我们可以记录和重放动作，这对于调试来说非常方便！我们可以通过命名我们的行为来达到类似的目的

我想在这里展示一些基本原则，并表明如果你愿意，你可以拥有一个非常精简的商店更新周期。

我们的下一步将是添加一个 ***connect()*** 函数，这样我们就可以将我们的应用程序包装在一个特设中，这样我们就可以像 *redux-connect* 那样将状态片段作为道具传递给组件。