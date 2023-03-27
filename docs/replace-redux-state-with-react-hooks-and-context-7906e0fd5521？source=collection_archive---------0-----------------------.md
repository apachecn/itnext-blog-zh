# 用 React 钩子和上下文替换 Redux 状态

> 原文：<https://itnext.io/replace-redux-state-with-react-hooks-and-context-7906e0fd5521?source=collection_archive---------0----------------------->

![](img/f24a92bedbfe821d163d3177d5f30b54.png)

[来自 spdload 的图像](https://spdload.com/jobs-vacancies/front-end/front-end-developer-react-js/)

> 免责声明。我知道 Redux 提供的不仅仅是全局状态。我在这里提供的是一个优雅的全局状态，没有依赖性，但是 React 16.8+

R eact Hooks 于 2018 年底公布。然后在 2019 年 3 月发布。最近我看了 React Conf 视频，其中[丹·阿布拉莫夫](https://twitter.com/dan_abramov/)和[瑞安·弗洛伦斯](https://twitter.com/ryanflorence?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor)宣布并展示了钩子的力量。Ryan 实现了一个带钩子的 Reducer，同时慢慢地拆开了旧的 React 类组件。这是辉煌的，你应该看到这里的两个发言:(你可以跳到丹的一部分)

陈述以一些零散的细节结束。在监听状态变化的同时，reducer 如何变得全局可用？在视频结束时，他实际上低声说道:

> 然后你加上[上下文……](https://reactjs.org/docs/context.html)

突然，一个新世界打开了。

*   创建 React 应用程序。
*   从 React 导入 useReducer 和 useContext。
*   创建一个减速器开关盒。
*   创建一个初始状态。
*   将状态传递给任何一个只有一行代码的组件(！idspnonenote)。！)的代码。

你得到了全局状态，本地和反应！Medium 上出现了很多关于 useReducer+useContext 设置的文章，给你一个带有 Reducer 的全局状态，没有 npm install redux redux-react。

我开始慢慢重构我的技术雷达协作应用，就像丹和医生建议的那样。然后我做得过火了，把我的应用程序撕成了碎片。我有 4 个不同的 StoreContext.js 文件试图完成同样的事情，在遵循指南的同时测试不同的实现。我最终得到的是许多解决方案的组合。

这是我最终得出的解决方案

# 用钩子和上下文减少反应状态

在您的项目中，创建/context 目录。添加 3 个文件:actions.js、reducers.js、StoreContext.js。有了这 3 个文件和一个 React 组件，您就可以做很多事情了！

我在代码沙箱中创建了一个演示应用程序。这显示了使用钩子只需几行代码就可以获得一个全局状态存储。如果你喜欢互动，看看这个演示。进一步的阅读包括这个项目文件的副本。

这是怎么回事？

## actions.js

使用带有前缀“use”的挂钩约定定义 useActions(state，dispatch)。useAction() { … }中定义的所有函数都将在 useContext()中的操作中可用。动作用于定义比纯粹的分派(类型、有效负载)功能更高级的逻辑。

```
import { types } from "./reducers";export const useActions = (state, dispatch) => {
  function addTechIfNotInList(newTech) {
    const techIndex = state.techList.indexOf(newTech);
    if (techIndex !== -1) {
      alert("Tech is defined in list");
    } else {
      dispatch({ type: types.ADD_TO_TECH_LIST, payload: newTech });
    }
  }return {
    addTechIfNotInList
  };
};
```

> 有了 Redux，逻辑应该何去何从就有了一个很好的平衡；还原者或行动创造者。找到适合你的模式！

## reducers.js

在这个文件中，我定义了初始状态、类型和缩减器本身。当我们从应用程序中的任何地方调度一个类型时，它将匹配开关条件，并对状态执行正确的操作。如果你熟悉 Redux，这应该对你来说非常熟悉！

```
const initialState = {
  techList: ["TypeScript", "React Hooks"]
};const types = {
  SET_TECH_LIST: "SET_TECH_LIST",
  ADD_TO_TECH_LIST: "ADD_TO_TECH_LIST",
  REMOVE_FROM_TECH_LIST: "REMOVE_FROM_TECH_LIST"
};const reducer = (state = initialState, action) => {
  switch (action.type) {
    case types.SET_TECH_LIST:
      return {
        ...state,
        techList: action.payload
      };
    case types.ADD_TO_TECH_LIST:
      return {
        ...state,
        techList: [...state.techList, action.payload]
      };
    case types.REMOVE_FROM_TECH_LIST:
      return {
        ...state,
        techList: state.techList.filter(
          tech => tech !== action.payload)
      };
    default:
      throw new Error("Unexpected action");
  }
};
export { initialState, types, reducer };
```

您可以看到，我已经选择在这个缩减器中包含一些逻辑。移除 tech 时，调度(REMOVE_FROM_TECH，techObj)就够了。然而，当添加技术时，它总是将它添加到状态中。这是我在动作创建器中创建的逻辑。

> 更高级的逻辑应该放在动作创建器中，比如 API 调用和其他[副作用，让你的函数变得“不纯粹”。](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)reducer 中还可以添加基本的数组、对象和字符串操作。

## StoreContext.js

这就是我们使用上下文使我们的存储全局可检索的地方。这相当于 Redux' createStore()，只是…不同。

该文件中的代码是

```
const StoreContext = createContext(initialState);const StoreProvider = ({ children }) => {
 **// Get state and dispatch from Reacts new API useReducer.** 
  const [state, dispatch] = useReducer(reducer, initialState);
 **// Get actions from useActions and pass it to Context**
  const actions = useActions(state, dispatch);**// Log new state
**  useEffect(() => console.log({ newState: state })},[state]);**// Render state, dispatch and special case actions
**  return (
    <StoreContext.Provider value={{ state, dispatch, actions }}>
      {children}
    </StoreContext.Provider>
  );
};export { StoreContext, StoreProvider };
```

现在，您可以在导出的 StoreProvider 的所有子组件中使用和**上下文**。只需用 useContext(StoreContext)初始化它，并获得 state、dispatch、actions 对象。

## App.js

有了这些设置，你可以清理你的组件了！只需要一行代码就可以获得状态和所有操作。

```
const { state, dispatch, actions } = useContext(StoreContext);
```

只要确保 App.js 是从 StoreContext.js 导出的 StoreProvider 的子元素即可

```
**//index.js**
<StoreProvider><App/></StoreProvider>
```

现在，我已经花了很多时间试图为我的用例解决这个问题！我现在已经在一个中等规模的应用程序中实现了这一点，开发者体验非常棒！如果你有任何批评，问题或意见，并想分享，请这样做！我一直在寻求改进🌟