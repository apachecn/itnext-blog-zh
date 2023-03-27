# 反应上下文和状态管理

> 原文：<https://itnext.io/react-context-and-state-management-16ed76686803?source=collection_archive---------4----------------------->

状态管理快速介绍

![](img/1fd7ffa22f6eb35b50a8c3056e7d605c.png)

[金赛](https://unsplash.com/@finalhugh?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

您已经掌握了 React 的基础知识，并希望提升您的游戏来创建更具凝聚力的应用程序。学习 React Context 应该是你的下一个目标，因为它是对状态管理的一个很好的介绍。我们大多数人都熟悉通过树将道具传递给子组件，但是当这变得难以管理和阅读时会发生什么呢？React Context 允许我们在组件之间共享和更新数据，而不需要通过每个子组件发送 props。上下文的一些很好的用例是存储登录用户的信息和维护语言或颜色主题等偏好。

# 上下文. js

## 我们开始吧

React 内置了上下文，因此您可以立即开始编码！

```
import React, { Context } from 'react';
```

React 上下文返回一个具有两个属性的对象，一个提供者和一个使用者。首先创建一个新的上下文并将其导出，以便我们以后使用。

```
export const MyContext = React.createContext();
```

## 供应者

在 Context.js 中，我们还可以创建一个新的类组件来管理我们的状态。在返回函数中，我们创建了一个接受值属性的提供者。我们现在可以创建一个对象来保存我们想要访问的状态和其他函数。

```
export class MyProvider extends Component {
  state = {
    isAuthenticated: false
  };authenticate = () => {
    this.setState({
      isAuthenticated: true
    });
  };render() {
    return (
      <MyContext.Provider
        value={{
          state: this.state,
          authenticate: this.authenticate
        }}
      >
        {this.props.children}
      </MyContext.Provider>
    );
  }
}
```

## 消费者

消费者是我们访问存储在上下文中的数据的方式。消费者标签中包含的任何代码都可以访问我们创建的上下文。创建一个新组件，并尝试导入上下文和创建一个消费者。我们需要多少消费者就有多少消费者，这样我们就可以避免通过道具传递数据。

```
import { MyContext } from "./Context";const page = () {
  return(
    <MyContext.Consumer>
      {({ state, authenticate }) => (
        // component code goes here
        // you now have access to state and values from the provider
        <button onClick={authenticate} />
      )}
    </MyContext.Consumer>
  )
}
```

## 更新上下文

如果您查看上面的代码示例，您可以看到如何更新上下文。单击按钮时，我们调用消费者返回的身份验证方法。在提供程序中，我们创建了一个 authenticate 方法，该方法将值 isAuthenticated 更新为 true。现在，应用程序的其余部分可以访问更新后的值。

```
authenticate = () => {
    this.setState({
      isAuthenticated: true
    });
  };
```

## 为你的应用提供背景

为了给你的整个应用程序提供上下文，用上下文组件包装你的<app>。</app>

```
import { MyContext, MyProvider } from "./Context";ReactDOM.render(
  <MyProvider>
    <MyContext.Consumer>{state => <App data={state} /></MyContext.Consumer>
</MyProvider>,
rootElement
);
```

您不必像这样实现提供者。如果您只需要在一个组件中访问 provider 中的数据，那么您可以包装该组件。因为我们希望在整个应用程序中访问用户信息，所以用提供者组件包装整个应用程序是有意义的。

## 结论

React Context 很好地介绍了如何管理状态。在尝试像 Redux 这样更复杂的解决方案之前，熟悉这些基本概念是很好的。

我已经展示了一个全局状态的例子，但是我建议创建一个更具体的上下文，比如 UserContext 或 AdminContext。开始编码，找到适合你的！