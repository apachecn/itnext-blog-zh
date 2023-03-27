# 键入 React (4) Routing

> 原文：<https://itnext.io/typing-react-4-routing-8a67f97930f0?source=collection_archive---------7----------------------->

![](img/7526fbe1daf91a77249f71c371f415c0.png)

在上一篇文章中，我们演示了如何将 TypeScript 与 Redux 和相关库一起使用。在本帖中，我们将讨论路由，这是几乎所有 React 应用程序都必须处理的重要部分。

以前的文章:

*   [打字反应(1) —基本](/typing-react-1-basic-488f661149f6?source=your_stories_page---------------------------)
*   [键入 React (2) — Material-UI](/typing-react-2-material-ui-9e95a4aec6bc?source=your_stories_page---------------------------)
*   [输入 React (3) — Redux](/typing-react-3-redux-84e73e41db7f)

# 反应-路由器-dom

最常见的方法是使用`react-router-dom`。要使用它，我们还必须安装类型:

```
$ npm install --save react-router-dom @types/react-router-dom
```

我将跳过基本设置部分，因为它非常简单(参见[手册](https://reacttraining.com/react-router/web/guides/quick-start))。我想展示的唯一一件事是，当我们需要使用路由参数时，我们需要组件具有`match`属性，可以通过`RouteComponentProps`像这样键入:

```
// Definition of IProps
export interface IProps
  extends RouteComponentProps<{ id?: string }>,
  ReturnType<typeof mapStateToProps>,
  ReturnType<typeof mapDispatchToProps> {
  // other properties
}

// In component
const TodoList: React.FC<IProps> = props => {
  const { match } = props;
  if (match.params.id) {
    ...
  }
}
```

注意`RouteCompnentProps`的模板参数是`match.params`的类型。`RouteCommponentProps`的定义是:

```
export interface RouteComponentProps<Params extends { [K in keyof Params]?: string } = {}, ...> { ... }
```

所以我们提供的类型必须是一个带有可选字符串值的字典。

# 连接反应路由器

如果你选择使用`redux-observable`，那么很可能你将需要使用`connected-react-router`。原因是，大多数时候我们希望在异步动作完成后使用重定向，但是使用`redux-observable`，异步调用是在 epics 中完成的，组件没有办法知道调用何时完成。因此，只有史诗知道正确的时间，并相应地处理重定向。将`<Redirect>`包装成一个动作，这样我们可以在 epics 中处理重定向。

假设我们在 Todo 项目详细信息屏幕上，并希望在保存项目后导航回项目列表。动作流应该是这样的:

*   调度`TODO:SAVE:REQUEST`
*   epic 在接收到`TODO:SAVE:REQUEST`动作时发送异步调用
*   当异步调用返回时，将`TODO:SAVE:SUCCESS`和`REDIRECT`动作推送到动作流

诀窍在于，在最后一步中，我们需要通过两个动作将一个可观察对象(异步调用的响应)转换为另一个可观察对象。这部史诗可以这样写:

```
export const SaveTodoEpic: RootEpic = (actions$, $state, { todos }) =>
  actions$.pipe(
    filter(isOfType(getType(listTodo.request))),
    mergeMap(action =>
      todos.updateTodos$().pipe(mergeMap(res => of(listTodo.success(res), push('/'))),
    catchError(err => of(updateTodo.failure(err))),
  );
```

由于`push`成为动作流的一种可能类型，我们也需要改变`store/types.d.ts`(注意添加到`RootAction`的`CallHistoryMethodAction`类型:

```
declare module 'StoreTypes' {
  export type Store = StateType<typeof import('./index').default>;
  export type RootAction = ActionType<typeof import('./actions').default> | CallHistoryMethodAction;
  export type RootState = StateType<ReturnType>typeof import('./reducers').default>>;
  export type RootEpic = Epic<RootAction, RootAction, RootState, Services>;
}
```

这应该可以解决路由问题。