# 一个非常简单的带有钩子应用编程接口的 Redux 反应绑定库

> 原文：<https://itnext.io/a-deadly-simple-react-bindings-library-for-redux-with-hooks-api-822295857282?source=collection_archive---------3----------------------->

Love Redux 和 Hooks API。

![](img/3bf9b61a554e2597bc68cbd2d01602f6.png)

> 有一篇后续文章描述了这个库的实现细节。请在此访问[。](https://medium.com/p/483a90de0c71)

Redux 是 reaction 的流行库之一。虽然 Redux 本身不与 React 绑定，但它经常与允许将 Redux 状态连接到任何 React 组件的官方绑定`react-redux`一起使用。

在本文中，我们提出了另一个绑定库来组合 React 和 Redux。这个库试图对初学者来说简单易行。截至本文撰写之时，这仍是一个实验项目，欢迎反馈。

## 典型的反应还原用法

假设我们有一个简单的全球状态。

```
const initialState = {
  counter: 0,
  person: {
    firstName: 'Harry',
    lastName: 'Potter',
    age: 11,
  },
};
```

显示人名的组件如下所示。

```
const PersonName = ({ firstName, lastName }) => (
  <div>
    <span>{firstName}</span>
    <span>{lastName}</span>
  </div>
);const mapStateToProps = state => ({
  firstName: state.person.firstName,
  lastName: state.person.lastName,
});export default connect(mapStateToProps)(PersonName);
```

随着 React Hooks API 的提出，react-redux 寻求支持 API [ [1](https://github.com/reduxjs/react-redux/issues/1063) ]的方法。一种可能的方法如下。

```
const PersonName = () => {
  const mapStateToProps = state => ({
    firstName: state.person.firstName,
    lastName: state.person.lastName,
  });
  const [{ firstName, lastName }] = useRedux(mapStateToProps);
  return (
    <div>
      <span>{firstName}</span>
      <span>{lastName}</span>
    </div>
  );
};export default PersonName;
```

在任一情况下，`mapStateToProps`都是检测该部分状态是否改变的关键。(技术上，在上面的钩子示例代码中，即使`useRedux`没有检测到任何变化，渲染过程也不会停止。我们需要适当的记忆或纾困渲染。)

## 反应-钩子-容易-还原

特别是对于初学者来说，经常写`mapStateToProps`不是件小事，而且它可能偶然包含了昂贵的计算。我们希望通过完全消除此问题来消除它。

reactor-hooks-easy-redux 是我们正在开发的一个库。它仍处于试验阶段，但您可以尝试一下。注意:这是基于未发布的 Hooks API，还没有投入生产。

[](https://github.com/dai-shi/react-hooks-easy-redux) [## dai-shi/react-hooks-easy-redux

### 通过钩子 API 进行反应的简单 Redux 绑定。通过创建一个…

github.com](https://github.com/dai-shi/react-hooks-easy-redux) 

对于这个库，上面的示例代码如下所示。

```
import { useReduxState } from 'react-hooks-easy-redux';const PersonName = () => {
  const state = useReduxState();
  return (
    <div>
      <span>{state.person.firstName}</span>
      <span>{state.person.lastName}</span>
    </div>
  );
};export default PersonName;
```

这通过代理神奇地工作，即使状态中的计数器属性或年龄属性改变，渲染也不会发生。

你可以尝试一些例子，也可以写你自己的例子。

```
git clone [https://github.com/dai-shi/react-hooks-easy-redux.git](https://github.com/dai-shi/react-hooks-easy-redux.git)
cd react-hooks-easy-redux
npm install
npm run examples:minimal
```

您可以将“极小”更改为“typescript”和“deep”等其他示例。

您也可以在线尝试。

[](https://codesandbox.io/s/github/dai-shi/react-hooks-easy-redux/tree/master/examples/01_minimal) [## react-hooks-easy-redux-example-CodeSandbox

### 为网络应用定制的在线代码编辑器

codesandbox.io](https://codesandbox.io/s/github/dai-shi/react-hooks-easy-redux/tree/master/examples/01_minimal) 

## 最终说明

有些读者可能已经注意到，本文中没有关于行动或`mapDispatchToProps`的讨论。这是因为我们的库只支持暴露`dispatch`，我们想集中讨论。即使有了`react-redux`，我们也建议初学者使用“派遣作为学习的道具”，这样会更直观。

## 变更日志

*   *【2018–11–17】:首次发布。*
*   *【2018–12–14】:删除 bailOutHack，新增* [*一篇后续文章*](https://medium.com/p/483a90de0c71) *。*