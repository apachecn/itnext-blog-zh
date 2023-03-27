# 如何使用 React 挂钩和上下文处理全局状态的异步操作

> 原文：<https://itnext.io/how-to-handle-async-actions-for-global-state-with-react-hooks-and-context-72d88c2e8323?source=collection_archive---------4----------------------->

## 用 React 跟踪

![](img/0be81e8f55ffa4487a4f07a8ba85fb3e.png)

# 介绍

我一直在开发 React Tracked，这是一个带有 React 钩子和上下文的全局状态库。

[https://react-tracked.js.org](https://react-tracked.js.org/)

这是一个小图书馆，只专注于一件事。它使用状态使用跟踪来优化重新渲染。从技术上来说，它使用代理来检测渲染中的使用情况，并且只在必要时触发重新渲染。

因此，React Tracked 的用法非常简单。这就像正常的使用环境。这里有一个例子。

```
const Counter = () => {
  const [state, setState] = useTracked();
  // The above line is almost like the following.
  // const [state, setState] = useContext(Context);
  const increment = () => {
    setState(prev => ({ ...prev, count: prev.count + 1 }));
  };
  return (
    <div>
      {state.count}
      <button onClick={increment}>+1</button>
    </div>
  );
};
```

具体例子请查看[文档](https://react-tracked.js.org/)中的“入门”。

现在，因为 React Tracked 是 React 挂钩和上下文的包装器，所以它本身不支持异步操作。这篇文章展示了一些如何处理异步动作的例子。它是为 React Tracked 编写的，但它可以在没有 React Tracked 的情况下使用。

我们使用的例子是一个简单的从服务器获取数据的例子。第一种模式没有任何库，使用定制的钩子。剩下的是用三个库，其中一个是我自己的。

# 不带库的自定义挂钩

让我们来看一个本地解决方案。我们首先定义一个商店。

```
import { createContainer } from 'react-tracked';const useValue = () => useState({ loading: false, data: null });
const { Provider, useTracked } = createContainer(useValue);
```

这是在 React Tracked 中创建存储(容器)的模式之一。请查看其他图案的[配方](https://react-tracked.js.org/docs/recipes#recipes-for-createcontainer)。

接下来，我们创建一个自定义挂钩。

```
const useData = () => {
  const [state, setState] = useTracked();
  const actions = {
    fetch: async (id) => {
      setState(prev => ({ ...prev, loading: true }));
      const response = await fetch(`[https://reqres.in/api/users/](https://reqres.in/api/users/)${id}?delay=1`);
      const data = await response.json();
      setState(prev => ({ ...prev, loading: false, data }));
    },
  };
  return [state, actions];
};
```

这是一个基于 useTracked 的新钩子，它返回状态和动作。您可以调用`action.fetch(1)`开始获取。

注意:如果需要稳定的异步函数，可以考虑用 useCallback 包装。

React Tracked 实际上接受一个自定义钩子，所以这个自定义钩子可以嵌入到容器中。

```
import { createContainer } from 'react-tracked';const useValue = () => {
  const [state, setState] = useState({ loading: false, data: null });
  const actions = {
    fetch: async (id) => {
      setState(prev => ({ ...prev, loading: true }));
      const response = await fetch(`[https://reqres.in/api/users/](https://reqres.in/api/users/)${id}?delay=1`);
      const data = await response.json();
      setState(prev => ({ ...prev, loading: false, data }));
    },
  };
  return [state, actions];
};const { Provider, useTracked } = createContainer(useValue);
```

尝试工作示例。

[https://codesandbox.io/s/hungry-nightingale-qjeis](https://codesandbox.io/s/hungry-nightingale-qjeis)

# 使用 useThunkReducer

[react-hooks-thunk-reducer](https://github.com/nathanbuchar/react-hook-thunk-reducer)提供自定义钩子`useThunkReducer`。这个钩子返回接受 thunk 函数的`dispatch`。

同样的例子可以这样实现。

```
import { createContainer } from 'react-tracked';
import useThunkReducer from 'react-hook-thunk-reducer';const initialState = { loading: false, data: null };
const reducer = (state, action) => {
  if (action.type === 'FETCH_STARTED') {
    return { ...state, loading: true };
  } else if (action.type === 'FETCH_FINISHED') {
    return { ...state, loading: false, data: action.data };
  } else {
    return state;
  }
};const useValue = () => useThunkReducer(reducer, initialState);
const { Provider, useTracked } = createContainer(useValue);
```

调用异步操作就像这样。

```
const fetchData = id => async (dispatch, getState) => {
  dispatch({ type: 'FETCH_STARTED' });
  const response = await fetch(`[https://reqres.in/api/users/](https://reqres.in/api/users/)${id}?delay=1`);
  const data = await response.json();
  dispatch({ type: 'FETCH_FINISHED', data });
};dispatch(fetchData(1));
```

对于 redux-thunk 用户来说应该很熟悉。

尝试工作示例。

[https://codesandbox.io/s/crimson-currying-og54c](https://codesandbox.io/s/crimson-currying-og54c)

# 使用减速器

[使用-saga-reducer](https://github.com/azmenak/use-saga-reducer) 提供自定义挂钩`useSagaReducer`。因为这个库使用了[外部 API](https://redux-saga.js.org/docs/api/index.html#external-api) ，所以可以不用 redux 就使用 redux-saga。

让我们用 Sagas 再次实现同样的例子。

```
import { createContainer } from 'react-tracked';
import { call, put, takeLatest } from 'redux-saga/effects';
import useSagaReducer from 'use-saga-reducer';const initialState = { loading: false, data: null };
const reducer = (state, action) => {
  if (action.type === 'FETCH_STARTED') {
    return { ...state, loading: true };
  } else if (action.type === 'FETCH_FINISHED') {
    return { ...state, loading: false, data: action.data };
  } else {
    return state;
  }
};function* fetcher(action) {
  yield put({ type: 'FETCH_STARTED' });
  const response = yield call(() => fetch(`[https://reqres.in/api/users/](https://reqres.in/api/users/)${action.id}?delay=1`));
  const data = yield call(() => response.json());
  yield put({ type: 'FETCH_FINISHED', data });
};function* fetchingSaga() {
  yield takeLatest('FETCH_DATA', fetcher);
}const useValue = () => useSagaReducer(fetchingSaga, reducer, initialState);
const { Provider, useTracked } = createContainer(useValue);
```

调用它很简单。

```
dispatch({ type: 'FETCH_DATA', id: 1 });
```

注意相似和不同之处。如果你不熟悉生成器函数，它可能看起来很奇怪。

无论如何，尝试工作的例子。

[https://codesandbox.io/s/fancy-silence-1pukj](https://codesandbox.io/s/fancy-silence-1pukj)

(不幸的是，截至发稿时，这个沙盒还不能在线使用。请“导出到 ZIP”并在本地运行。)

# useReducerAsync

[use-reducer-async](https://github.com/dai-shi/use-reducer-async) 提供自定义钩子`useReducerAsync`。这是我开发的库，灵感来自`useSagaReducer`。它不具备生成器函数所能做到的功能，但是它可以与任何异步函数一起工作。

下面是这个钩子的同一个例子。

```
import { createContainer } from 'react-tracked';
import { useReducerAsync } from 'use-reducer-async';const initialState = { loading: false, data: null };
const reducer = (state, action) => {
  if (action.type === 'FETCH_STARTED') {
    return { ...state, loading: true };
  } else if (action.type === 'FETCH_FINISHED') {
    return { ...state, loading: false, data: action.data };
  } else {
    return state;
  }
};const asyncActionHandlers = {
  FETCH_DATA: (dispatch, getState) => async (action) => {
    dispatch({ type: 'FETCH_STARTED' });
    const response = await fetch(`[https://reqres.in/api/users/](https://reqres.in/api/users/)${action.id}?delay=1`);
    const data = await response.json();
    dispatch({ type: 'FETCH_FINISHED', data });
  },
};const useValue = () => useReducerAsync(reducer, initialState, asyncActionHandlers);
const { Provider, useTracked } = createContainer(useValue);
```

您可以用同样的方式调用它。

```
dispatch({ type: 'FETCH_DATA', id: 1 });
```

模式类似于 useSagaReducer，但语法类似于 useThunkReducer 或本机解决方案。

尝试工作示例。

[https://codesandbox.io/s/bitter-frost-4lxck](https://codesandbox.io/s/bitter-frost-4lxck)

# 比较

虽然可能有失偏颇，但以下是我的建议。如果您喜欢没有库的解决方案，请使用本机解决方案。如果你是 saga 用户，毫无疑问使用 useSagaReducer。如果你喜欢 redux-thunk，使用 redux-thunk reducer 会很好。否则，请考虑 useReducerAsync 或本机解决方案。

对于 TypeScript 用户，我的建议是 useSagaReducer 和 useReducerAsync。原生解决方案应该也可以。请查看 React Tracked 中的完整类型示例。

*   [https://github . com/Dai-Shi/react-tracked/tree/master/examples/12 _ async](https://github.com/dai-shi/react-tracked/tree/master/examples/12_async)
*   [https://github . com/Dai-Shi/react-tracked/tree/master/examples/13 _ saga](https://github.com/dai-shi/react-tracked/tree/master/examples/13_saga)

# 结束语

老实说，我认为原生解决方案对于小应用来说很好。所以，我没有动力去创建一个图书馆。然而，在为 React Tracked 编写教程的过程中，我注意到用库来限制模式更容易解释。use-reducer-async 是一个很小的库，没有什么特别的。但是，它显示了一个模式。

关于异步操作的另一个注意事项是数据获取的暂停。目前在实验频道。新推荐的数据获取方式是取即渲染模式。这与这篇文章中描述的模式完全不同。我们将会看到它如何发展。最有可能的是，这种新模式需要一个库来方便开发人员遵循这种模式。有兴趣的话可以看看我的[实验项目](https://github.com/dai-shi/react-hooks-fetch)。

*原载于 2019 年 12 月 20 日*[*【https://blog.axlight.com*](https://blog.axlight.com/posts/how-to-handle-async-actions-for-global-state-with-react-hooks-and-context/)*。*