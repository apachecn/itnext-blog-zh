# 将上下文挂钩作为状态管理解决方案

> 原文：<https://itnext.io/react-hooks-with-context-as-a-state-management-solution-526d1c13a07d?source=collection_archive---------1----------------------->

![](img/0a8cfecea216a961920204fe27865ec9.png)

图片由 [MichaelGaida](https://pixabay.com/users/michaelgaida-652234/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=3994109) 来自 [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=3994109)

我们都知道在大多数客户端应用程序中用来管理应用程序状态的大玩家 [**Redux**](https://redux.js.org/) 和 [**Mobx**](https://mobx.js.org/README.html) 库。但有时它们并不符合我们的需求，这就是你从 react 库中取出的 [**上下文**](https://reactjs.org/docs/context.html) 可以用来管理我们的应用程序状态的地方？

> 为什么要看这篇文章？因为我将提供一个简单而详细的例子来说明如何在您的应用程序中实现这个解决方案。

在我们的应用程序中使用这个解决方案一年多之后，我可以有把握地说，是的，你可以使用 [react context](https://reactjs.org/docs/context.html) ，在这篇文章中，我将尝试以一种简化的方式展示我是如何使用它的。

我将要展示的代码使用了 [react 钩子](https://reactjs.org/docs/hooks-intro.html)，这是目前推荐的编写 react 组件的方式，与编写类相反。

**为什么我没有用 Redux？** 我从事的项目是 angular 应用程序中 react 的混合，它已经管理了整个应用程序的状态，我想添加一个轻量级的状态库，可以减少组件之间传递的状态参数。由于 Redux 非常健壮，并且针对单个商店，所以它不太适合这个项目。

**我为什么不用 Mobx？我以前在一个不同的项目中使用过 Mobx，感觉就像使用上下文一样，因为我在寻找一个轻量级的解决方案，这次 Mobx 似乎不是一个合适的选择。**

为什么我要使用上下文？React 引入了 Context API，这是状态管理包的一种替代方案。它提供了一种通过组件树传递数据的方法，而不必在每一层手动向下传递属性。

上下文 API 只在需要从 3+级嵌套组件中访问数据时使用。

Context API 很棒，因为它使开发人员摆脱了必须理解、使用外部状态管理并将其集成到 react 应用程序中的麻烦。

如何用钩子实现上下文？
*context/index.js //这是我们配置上下文提供者的地方
这是最重要的文件，我们在这里创建“本地”上下文提供者，它将使我们能够创建单独的上下文(如商店)。我们还使用了*[*useReducer*](https://reactjs.org/docs/hooks-reference.html#usereducer)*钩子，使我们能够使用和更新本地上下文，就像使用 redux 一样。*

*操作以这种方式编写，因此我们可以有一个该上下文可用操作的文档。我们还可以在每个函数的上方添加文档来说明不同的参数。*

```
import React, { useReducer } from 'react';const initialState = {
 todoList: ['buy dinner', 'run 5 km', 'finish work']
};const actions = {
 add: (list, newItem) => {
  return newItem === '' ? list : list.concat(newItem);
 }, remove: (list, itemToRemove) => {
  return itemToRemove === '' ? list : list.filter(item => item !==   itemToRemove);
 }};const PageContext = React.createContext(initialState);
const pageReducer = (state, action) => {
 switch (action.type) {
  case 'add':
   state.todoList = actions.add(state.todoList, action.payload);
   return { ...state };
  default:
   throw new Error();
  }
};const PageContextProvider = props => {
const [contextState, updateContext] = useReducer(pageReducer, initialState);return (
 <PageContext.Provider value={{ contextState, updateContext }}> 
  {props.children}
 </PageContext.Provider>
);};export { PageContext, PageContextProvider, actions };
```

*your-page/index.js//这是容器页面使用上下文的地方*

*在该文件中，我们导入内容提供者，因此我们将本地上下文的访问权授予上下文下的所有子节点，在本例中为 ChildA、ChildB、ChildC。只有当我们在子组件*中使用 useContext 时，它们中的每一个才会在上下文改变时呈现

```
import React, { useEffect } from 'react';
import { PageContextProvider } from './context';
import ChildA from './ChildA';
import ChildB from './ChildB';
import ChildC from './ChildC';import './style.less';export default function TemplatePage() {return (<div className="template-page-wrapper">
 <PageContextProvider>
  <h2>** This is a template for page component **</h2>
  <ChildB />
  <ChildA />
  <ChildC />
 </PageContextProvider>
</div>
);
```

*ChildA/index.js//使用上下文的页面中的一个子节点*

*在这个子组件中，我们可以看到如何导入 pageContext，并使用 useContext 钩子从 PageContext 中提取状态和使用 useReducer 钩子公开的更新函数。*

```
import React, { useContext } from 'react';
import { PageContext } from '../context';import './style.less';export default function ChildA() {
 const { contextState, updateContext } = useContext(PageContext);
 return (
  <div className="child-a-wrapper">
   <strong>List of todos:</strong>
   {contextState.todoList.map((item, index) => (
     <div key={index} onClick={i => clickHandler(i)}>{item}</div>
   ))}
  </div>
 );function clickHandler(item) {
   updateContext({
     type: 'add',
     payload: item  
   });
 }
}
```

下一步是什么？

1.改进减速器
当你开始使用这种方法时，你很快就会看到减速器的开关盒变得越来越大，需要放在单独的文件中，这很难，我的意思是你如何组合几个开关盒？我不知道..因此，我想到了一种替代方案，可以给出同样的结果。这是代码:

```
// the new pageReducer varaible
const pageReducer = (state, action) => { const combinedReducer = {
  ...reducerA,
  ...reducerB,
  ...reducerC
 }; return combinedReducer[action.type](state, action);};
```

在单独的文件中，我放置了单独的减速器，这是它们的样子

```
const reducerA = { functionA: (state, action) => {
    ....
  },
  functionB: (state, action) => {
    ....
  }}export default reducerA;
```

2.了解局限性并为其制定解决方案。我想展示一个有趣的限制和解决方案。在大多数应用程序中，我们有一个容器，它是页面组件，在页面组件内部，我们有更小的组件。如果没有状态管理库，大多数应用程序将在页面中提取数据并作为道具传递给子组件，而在有状态管理库的应用程序中，提取数据的操作也将更新存储，每个“连接”的子组件都可以访问存储数据。在上下文中，这是有限制的。用提供者包装子组件的页面不能访问本地上下文，因此它不能将获取的数据传递给存储以供其他子组件使用。这是一个限制，因此有两种方法可以解决这个问题。你可以为那个页面组件制作另一个包装组件，我不认为这很好。或者…你可以使用特殊的子组件，我们称之为“APIConnect”。子组件可以访问本地上下文，因此他可以进行所有的获取并更新所有其他子组件，这使得父页面更干净，并为这一限制提供了一个很好的解决方案。

让我们看看这个特殊孩子的代码:

```
import React, { useEffect, useContext } from 'react';
import { PageContext } from '../context';
import { /* all the services */} from '../services';export default function APiConnect() {
 const { updateContext } = useContext(PageContext);
 useEffect(() => {
  async function getData() {
  Promise.all([
   promisify(blaService),
   promisify(bliService),
   promisify(bloService),
  ]).then(data => {
  const [bla, bli, blo] = data; updateContext({
    type: 'initialiseData',
    payload: { bla, bli, blo }
   });
  }); }
 getData();
}, []);return <></>;}
```

promisify 是 promise 的一个很好的包装函数，它采用了一个常规的回调函数和一个有效载荷(POST 所需要的),并将回调函数转化为 Promise，这样我们就可以使用 [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)

我希望这篇文章能激励你，关于提高性能的后续文章可以在[这里](https://talwaserman.medium.com/react-context-and-react-tracked-296a6b86f890)找到