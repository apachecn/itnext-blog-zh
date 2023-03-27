# 你是怎么做的？

> 原文：<https://itnext.io/how-do-you-do-your-todos-cc6ece298bb2?source=collection_archive---------1----------------------->

## 带类型脚本的 React 16.3 中的状态管理选项

![](img/5ba8a7b1b18ff3cd1a4be7e90059a7ac.png)

[托尼·韦伯斯特](https://unsplash.com/@tonywebster?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的“现代几何霓虹灯桥建筑在夜晚的人行道上，高架步道桥”

随着 Javascript 应用程序从简单的脚本或简单的 DOM 操作发展到成熟的单页面应用程序，出现了几个规模问题。一个是代码组织的问题，经历了各种迭代(gulp、grunt、bower 等)，直到 [Webpack](https://webpack.js.org/) 和 [NPM](https://www.npmjs.com/) 成为捆绑器和依赖管理器的首选。最终，社区中一些最聪明的人将注意力转向了数据的可伸缩性，状态管理成了热门话题。源自脸书的 [Flux](http://www.flux.io/‎) 架构以不变性和单向数据流为核心，为各种状态管理解决方案提供了动力。

一些框架(例如 Vue 和 Vuex)有一个官方的状态管理包。React 有 redux，这无疑是最受欢迎的解决方案，但是还有其他选择，甚至在 redux 中，围绕如何对副作用建模也有问题需要解决(将远程数据访问视为一种常见的副作用)。

随着 React 16.3 的到来，React Context API 从架构中一个没有文档记录和不稳定的部分变成了另一个小型应用程序的潜在替代品。我决定使用 6 种不同的方法生成一个规范的 Todo 应用程序，以了解它们之间的不同之处。6 个状态管理选项是:

1.  [本地状态](https://github.com/dfcook/react-todos) —使用道具周围的设置状态和传递状态
2.  [Redux w/ redux-thunk](https://github.com/dfcook/react-todos/tree/master/src#redux-thunk)
3.  [Redux w/Redux-observable](https://github.com/dfcook/react-todos/tree/redux-observable)
4.  [Redux w/ redux-saga](https://github.com/dfcook/react-todos/tree/redux-saga)
5.  [Mobx](https://github.com/dfcook/react-todos/tree/mobx)
6.  [React Context](https://github.com/dfcook/react-todos/tree/react-context) —设置状态并使用上下文将数据传递给子组件

这里的代码是。主分支使用本地状态实现基线解决方案，其他每个状态管理选项都是其分支，如自述文件中所述。每个解决方案也使用 typescript 和 [antd](https://ant.design/docs/react/introduce) 组件库。

网上已经有一大堆例子，我认为值得再举一个的原因是我发现其中一些使用了旧版本的状态管理库或 typescript，不能像写的那样工作。我希望随着新版本的发布，代码保持最新。希望这将有助于人们了解这些选项。

## [本地状态](https://github.com/dfcook/react-todos)

React 组件，至少是非功能性组件，都有自己的状态管理 API setState。状态是组件的局部状态，用类级别的属性初始化:

```
interface AppState {
  filter: string;
  todosLoading: boolean;
  todos: Todo[];
}....state: AppState = {
  filter: 'ALL',
  todosLoading: false,
  todos: []
};
```

然而，对状态对象的更新不是通过直接改变该类级别的属性来进行的，而是应该调用 setState，传入一组将与当前状态合并的状态更改，或者传入一个函数，该函数将组件的当前状态和属性作为参数，并再次返回要合并到组件状态中的更改对象。

```
addTodo = async (title: string) => {
  const response = await api.add(title); this.setState({
    todos: [ ...this.state.todos, response.data ]
  });
}
```

由于状态对于单个组件是局部的，所以它通过 props 与子组件共享。Props 可以是来自状态或函数的数据，这些数据将被调用以通过 setState 传播状态更新。

这已经是一个复杂的状态管理选项，基于不变性和单向数据流的合理原则。的确，正如 Redux 的创造者自己所说，[你可能不需要它](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)。然而，随着应用程序的增长，您可能会发现道具传递变得单调乏味且容易出错，因为应用程序的不同部分试图共享相同的数据，状态在层次结构中被推得越来越高。此时，您可能会决定需要 Redux。

## [Redux w/ redux-thunk](https://github.com/dfcook/react-todos/tree/master/src#redux-thunk)

[Redux](https://github.com/reactjs/redux) 是一个“JavaScript app 的可预测状态容器”。这是一种获取本地状态并将其放入容器(称为存储)的方式，然后可以将它连接到应用程序中的组件，这些组件可以从存储中接收数据作为道具，并将更新作为操作发送到存储。对于来自面向对象背景的开发人员来说，redux 的术语(reducers、action creators)可能显得晦涩难懂，但是一旦点击，它们就变得相对简单了，事实上 redux 本身的核心功能可以在大约 20 行代码中实现！

缩减器是一个接受当前状态和动作对象并返回新状态的函数。

```
export default (state: AppState, action: TodoAction): AppState => {
  switch (action.type) {
    case ActionTypes.ADD_TODO_SUCCESS:
      return {
        ...state,
        todos: [ ...state.todos, (action as AddTodoAction).todo]
      };
  }
}
```

动作是一个普通的 javascript 对象，带有一个类型属性(字符串)和一个有效负载。

```
{
  type: 'ADD_TODO_SUCCESS',
  todo: {
    text: 'Walk the dog',
    completed: false
  }
}
```

Redux 经常与 React 联系在一起，因为它与 React 配合得非常好，但是它也可以用于其他 JavaScript 框架。为了将 React 组件链接到存储，我们构建了“容器”，这些容器使用 React-redux“connect”高阶函数将组件属性链接到状态属性和动作。然后，我们的组件可以很容易地测试功能组件，而不需要了解正在使用的状态管理系统。

[Redux](https://github.com/gaearon/redux-thunk) 通过使用中间件是可扩展的，其中一个是 redux-thunk，它通过允许动作创建者返回一个函数而不是动作对象来解决异步动作的问题。这允许副作用(比如调用外部 API)融入到我们的存储操作中。

```
export const addTodo = (title: string) => {
  return async (dispatch: Dispatch) => {
    const response = await api.add(title);
    dispatch(addTodoSuccess(response.data));
  };
};
```

## [Redux w/ redux-observable](https://github.com/dfcook/react-todos/tree/redux-observable)

Redux-observable 是 Redux 异步操作的另一个中间件，它使用[RxJS](http://reactivex.io/rxjs/)observable 从另一个方向解决问题。核心概念是“Epic”，一个接收动作流并返回动作流的函数。

```
const fetchTodoEpic = (action$: ActionsObservable<FetchTodosAction>) =>
  action$
    .ofType(ActionTypes.FETCH_TODOS)
    .mergeMap((action: FetchTodosAction) =>
      Observable.ajax({
        url: baseUrl,
        withCredentials: false,
        crossDomain: true
      })
      .map(req => fetchTodosSuccess(req.response)));
```

RxJS 附带了大量的“操作符”，用于操作流的函数，比如上面例子中的 mergeMap。例如，takeUntil 操作符允许调度一个取消异步操作的动作，这是 redux-observable 的一大卖点。

## [Redux w/ redux-saga](https://github.com/dfcook/react-todos/tree/redux-saga)

Redux-saga 是 Redux 的另一个中间件，旨在减轻副作用。它使用 ES6 生成器，这意味着异步流变得非常容易阅读(假设您理解[生成器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function%2A)的语法)。

```
export function* fetchTodos() {
  yield put(updateLoading(true));
  const response = yield call(api.fetch);
  yield put(loadTodosSuccess(response.data));
  yield put(updateLoading(false));
}
```

在 3 个 redux 中间件中，这是我最喜欢使用的一个，我认为故事和动作之间的分离使得最终用户代码非常可读，并且易于理解和测试。

## [Mobx](https://github.com/dfcook/react-todos/tree/mobx)

[Mobx](https://mobx.js.org/) 采用了与 Redux 截然不同的状态管理方法，这对于来自 Vue 和 Vuex 的开发人员来说是非常熟悉的。Mobx 存储是“反应式”的，您选择与您的状态一致的数据结构的形状，然后使用注释将其标记为可观察的。然后，您的组件使用一个观察者注释，使它们能够对存储状态的变化做出反应。

```
@observable todosLoading = false;
@observable todos: Todo[] = [];async refreshTodos() {
  this.todosLoading = true;

  const response = await api.fetch();
  this.todos = response.data; this.todosLoading = false;
}
```

来自 Redux 和 setState，这有点文化冲击，不变性在哪里？它当然非常不同，但是它允许一些很好的抽象，例如 computed 注释允许将计算属性逻辑放在存储中。

```
@computed get filteredTodos() {
  switch (this.filter) {
    case 'ACTIVE':
      return this.todos.filter(todo => !todo.completed); case 'COMPLETED':
      return this.todos.filter(todo => todo.completed); default:
      return this.todos;
  }
}
```

## [反应上下文](https://github.com/dfcook/react-todos/tree/react-context)

React 的上下文 API 长期以来一直被标记为实验性的，可能会发生变化，但是它的消费者/提供者模型是 Redux 的提供者的基础。在 16.3 版本中，API 已经稳定下来，现在是状态管理的另一个选择。

类似于本地状态选项状态和对其进行操作的功能位于层次结构顶端的组件中。然而，它不是通过 props 将状态传递给子组件，而是使用 createContext 函数将状态放在一个上下文中。

```
export interface AppState {
  filter: string;
  loading: boolean;
  todos: Todo[];
  updateFilter: (filter: string) => void;
  addTodo: (title: string) => void;
  toggleTodo: (todo: Todo) => void;
  deleteTodo: (todo: Todo) => void;
}export const TodoContext = React.createContext<AppState>({
  filter: 'ALL',
  loading: false,
  todos: [],
  updateFilter: (filter: string) => {},
  addTodo: (title: string) => {},
  toggleTodo: (todo: Todo) => {},
  deleteTodo: (todo: Todo) => {}
});
```

上面代码中的 TodoContext 是一个具有两个属性的对象，Provider 和 Consumer，然后可以在我们的组件 JSX 中使用它们将组件与上下文挂钩。值得注意的是，消费者使用另一种模式，即“render prop ”,以允许在呈现子组件时访问上下文。

我希望上面的简要描述，更重要的是，代码对每种状态管理方法都有一个简单的介绍。