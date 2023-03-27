# Redux Toolkit 如何在下一个 React 应用中减少 Redux 的设置

> 原文：<https://itnext.io/how-redux-toolkit-can-reduce-your-setup-of-redux-in-your-react-app-d87baab59268?source=collection_archive---------2----------------------->

![](img/0a1eba016e621ad907aae016caf05b05.png)

在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上 [Max Duzij](https://unsplash.com/@max_duz?utm_source=medium&utm_medium=referral) 拍摄的照片

React 应用程序中状态管理的设置可能是一项复杂或重复的任务。考虑到这一点，Redux Toolkit(又名“RTK ”)应运而生。React toolkit 是官方的、固执己见的、包含电池的工具集，用于高效的 Redux 开发。如官网所说，toolkit 是本着`[create-react-app](https://create-react-app.dev/)`或`[apollo-bost](https://www.apollographql.com/docs/react/migrating/boost-migration/)`的精神打造的。从初学者到高级 Redux 用户，这个工具是为那些想简化 Redux 代码/设置的人提供的一套工具。

## 使用工具包我会得到什么好处？

## 用有用的函数减少样板文件

*   **configureStore() -** 组合 reducers，添加您提供的 redux 中间件，默认为开发阶段带来`redux-thunk`和有用的中间件，生成动作创建者和动作类型，然后启用 DevTools，无需任何复杂的配置。
*   **createSlice() -** 一个助手函数，接受一组缩减器函数、初始状态和片名，该函数执行[片缩减器模式](https://redux.js.org/recipes/structuring-reducers/splitting-reducer-logic/)。
*   **create selector()**—直接从 [reselect](https://github.com/reduxjs/reselect) 库中重新导出，以便于在 [useSelector](https://react-redux.js.org/api/hooks) 钩子内部创建逻辑。
*   这里可以看到更多功能[。](https://redux-toolkit.js.org/introduction/quick-start)

## 默认情况下包括流行的中间件

默认情况下，RTK 包括这三个中间件。

*   **redux-immutable-state-invariant:**只在**开发阶段使用**，如果你试图直接变异状态**，这个 lib 会抛出错误。**有一个特殊的情况，因为 RTK [默认情况下也在内部使用 immer.js】，这是自动为您完成的，因为该库提供的结构不允许您改变它们。](https://redux-toolkit.js.org/usage/usage-guide#considerations-for-using-createreducer)
*   **serializable-state-invariant-middleware:**检查你的状态树和动作是否有非序列化的数据，比如:函数、承诺和非普通的 JavaScript 数据值。如果他们有这些控制台。错误将被打印。
*   **redux-thunk:** 如果你想减少 redux 逻辑的副作用，推荐使用一个中间件。

默认情况下，这三个中间件是通过函数`getDefaultMiddleware()`添加的，除非您想要添加额外的中间件或删除它们。

## 设置 Redux 开发工具

使用 RTK，Redux DevTools 在默认情况下是受支持的，通过一个简单的标志 **true** 或 **false，**您可以启用或禁用 redux devtools，以简化应用程序中的调试状态。该配置在`configureStore`功能中启用。

## Github 知识库

您可以使用 react、Redux Toolkit、React Router & MaterialUI 在 React 应用程序中看到所有这些功能，React Router & material ui 由 [**乔丹·温斯洛**](https://twitter.com/JordanDWinslow) 创建，如果对您有帮助，请告诉我！所有的荣誉都属于乔丹，非常感谢你分享这个知识库。

[](https://github.com/JordanWinslow/flash-cards) [## 约旦文斯洛/抽认卡

### 创建新的抽认卡，点击底部导航中的+图标，填写卡片的正面和背面，然后点击…

github.com](https://github.com/JordanWinslow/flash-cards)