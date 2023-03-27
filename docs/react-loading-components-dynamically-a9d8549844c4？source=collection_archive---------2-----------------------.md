# React —动态加载组件

> 原文：<https://itnext.io/react-loading-components-dynamically-a9d8549844c4?source=collection_archive---------2----------------------->

![](img/ae6552b8dd2e9d41649edddde58dcc5f.png)

当您编写 React 应用程序时，对于大多数项目来说，所有组件都是一次加载的，这是一件好事，因为一旦应用程序在客户端上完全加载，并且您处于离线状态，所有组件都在那里！

一些项目要求模块化架构，用户只加载应用程序的一部分——假设你想为每种类型的用户加载不同的应用程序。多亏了[*React Code-Splitting*](https://reactjs.org/docs/code-splitting.html)和 [*Webpack* 支持](https://webpack.js.org/guides/code-splitting/)这才能做到。

这里我要展示的例子非常简单，没有考虑复杂的应用架构或服务器端渲染的其他策略，但这是实现模块化架构的一个很好的起点。

除了 React 代码分割和 Webpack，参考 [react-loadable](https://www.npmjs.com/package/react-loadable) 也很重要，因为它非常容易处理代码分割。

这是用**react-loaded**定义一个组件的样子:

```
let MyComponent = Loadable({
  loader: () => import('./MyComponent'),
  loading: () => <div>Loading...</div>
});
```

在我写这篇文章的时候，我正在使用 CRA 1 . 1 . 4 和 react-loadable 5.4.0。

下面的例子是基于 CRA。我只是把标题和主体分成两个独立的部分(见下文)，让最初的应用程序在初始加载时更小更快。

```
import React, { Component } from 'react';
import Loadable from 'react-loadable';
import './App.css';// Modules configured. This can come from a database...
const available = ['Header', 'Body'];class App extends Component {
  state = { modules: [], active: [] }; // Toggle module
  toggleModule = (nome) => {
    const { active } = this.state, modules = []; // Add or remove from list
    let i = active.indexOf(nome);
    if (i > -1) active.splice(i, 1);
    else active.push(nome); // Create loadables. THIS IS THE MAGIC!
    active.map(m => {
      return modules.push(Loadable({
        loader: () => import('./'+m), // Here can be any component!
        loading: () => <div>Loading { m }...</div>,
      }));
    });
    this.setState({ ...this.state, modules, active });
  } render() {
    const { modules, active } = this.state;
    return (
      <div className="App"> { modules.map((item, i) => {
          let Module = modules[i]
          return <Module key={i} />
        }) } <ul>
          { available.map((name, i) => (
            <li key={i}>
              { name }
              <input type="checkbox"
                checked={active.indexOf(name) > -1}
                onClick={e => this.toggleModule(name)}
              />
            </li>
          ))}
        </ul>
      </div>
    );
  }
}export default App;
```

出于演示的目的，我将活动组件保持在本地*状态*，并且我还呈现了可用组件的列表，以让用户触发**模块化行为**。

为了完成这个例子，下面是 *Header.js* :

```
import React from 'react';
import logo from './logo.svg';const Header = () => (
  <header className="App-header">
    <img src={logo} className="App-logo" alt="logo" />
    <h1 className="App-title">Welcome to React</h1>
  </header>
)export default Header;
```

…还有 Body.js:

```
import React from 'react';const Body = () => (
  <p className="App-intro">
    To get started, edit <code>src/App.js</code> and save to reload.
  </p>
)export default Body;
```

运行应用程序，并在切换项目时查看您的网络通话。

玩得开心！:)