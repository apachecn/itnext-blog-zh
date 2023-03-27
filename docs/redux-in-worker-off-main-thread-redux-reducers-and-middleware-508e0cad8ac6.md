# 工作线程中的 Redux:脱离主线程的 Redux Reducers 和中间件

> 原文：<https://itnext.io/redux-in-worker-off-main-thread-redux-reducers-and-middleware-508e0cad8ac6?source=collection_archive---------3----------------------->

## 异步中间件示例

![](img/df76874279f01d468318ac8719f729f7.png)

# 介绍

Redux 是一个框架无关的全局状态库。它经常与 React 连用。

虽然我喜欢 Redux 的抽象，但 React 将在不久的将来引入并发模式。如果我们想得到`useTransition`的好处，状态必须在 React 内部以允许状态分支。这意味着我们无法从 Redux 中获益。

我一直在为允许状态分支的全局状态开发[反应跟踪](https://react-tracked.js.org/)。它在并发模式下运行良好。这给我留下了一个问题:只有 Redux 能做的用例是什么。

Redux 不允许状态分支的原因是状态在外部存储中。那么，拥有一个外部商店的好处是什么。 [Redux Toolkit](https://redux-toolkit.js.org/) 可以是一个答案。我有另一个答案，一个允许脱离主线程的外部存储。

React 是一个 UI 库，它打算在主 UI 线程中运行。Redux 通常是 UI 不可知的，所以我们可以在工作线程中运行它。

已经进行了几次实验来从主线程中卸载 Redux，并在 Web Workers 中运行 Redux 的部分或全部工作。我开发了一个库来卸载整个 Redux 商店。

# 裁员

这个库叫做 redux-in-worker。请查看 GitHub 库。

[](https://github.com/dai-shi/redux-in-worker) [## 戴氏/复工

### 受 React+Redux+Comlink = Off-main-thread 启发的 Web Worker 中的完整 Redux。这仍然是一个实验…

github.com](https://github.com/dai-shi/redux-in-worker) 

虽然这个库不依赖于 React，但它是为了与 React 一起使用而开发的。也就是说，它将确保保持对象引用相等，从而防止 React 中不必要的重新渲染。

请查看我写的关于此事的博文。

[脱离主线程的反应与性能的降低](https://blog.axlight.com/posts/off-main-thread-react-redux-with-performance/)

在接下来的部分中，我将展示一些使用 redux-in-worker 进行异步操作的代码。

# redux-api 中间件

redux-api-middleware 是早期就存在的库之一。它接收动作并运行动作中描述的 API 调用。action 对象是可序列化的，因此我们可以毫无问题地将其发送给 worker。

下面是示例代码:

```
import { createStore, applyMiddleware } from 'redux';
import { apiMiddleware } from 'redux-api-middleware';import { exposeStore } from 'redux-in-worker';export const initialState = {
  count: 0,
  person: {
    name: '',
    loading: false,
  },
};export type State = typeof initialState;export type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'setName'; name: string }
  | { type: 'REQUEST' }
  | { type: 'SUCCESS'; payload: { name: string } }
  | { type: 'FAILURE' };const reducer = (state = initialState, action: Action) => {
  console.log({ state, action });
  switch (action.type) {
    case 'increment': return {
      ...state,
      count: state.count + 1,
    };
    case 'decrement': return {
      ...state,
      count: state.count - 1,
    };
    case 'setName': return {
      ...state,
      person: {
        ...state.person,
        name: action.name,
      },
    };
    case 'REQUEST': return {
      ...state,
      person: {
        ...state.person,
        loading: true,
      },
    };
    case 'SUCCESS': return {
      ...state,
      person: {
        ...state.person,
        name: action.payload.name,
        loading: false,
      },
    };
    case 'FAILURE': return {
      ...state,
      person: {
        ...state.person,
        name: 'ERROR',
        loading: false,
      },
    };
    default: return state;
  }
};const store = createStore(reducer, applyMiddleware(apiMiddleware));exposeStore(store);
```

上面的代码运行在一个 worker 中。

主线程中运行的代码如下:

```
import { wrapStore } from 'redux-in-worker';
import { initialState } from './store.worker';const store = wrapStore(
  new Worker('./store.worker', { type: 'module' }),
  initialState,
  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__(),
);
```

请在存储库中找到完整的示例:

[https://github . com/Dai-Shi/redux-in-worker/tree/master/examples/04 _ API](https://github.com/dai-shi/redux-in-worker/tree/master/examples/04_api)

# 还原传奇

另一个可以和 redux-in-worker 一起使用的库是 [redux-saga](https://redux-saga.js.org/) 。对于任何带有生成器的异步函数来说，这都是一个强大的库。因为它的动作对象是可序列化的，所以它只是工作。

下面是示例代码:

```
import { createStore, applyMiddleware } from 'redux';
import createSagaMiddleware from 'redux-saga';
import {
  call,
  put,
  delay,
  takeLatest,
  takeEvery,
  all,
} from 'redux-saga/effects';import { exposeStore } from 'redux-in-worker';const sagaMiddleware = createSagaMiddleware();export const initialState = {
  count: 0,
  person: {
    name: '',
    loading: false,
  },
};export type State = typeof initialState;type ReducerAction =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET_NAME'; name: string }
  | { type: 'START_FETCH_USER' }
  | { type: 'SUCCESS_FETCH_USER'; name: string }
  | { type: 'ERROR_FETCH_USER' };type AsyncActionFetch = { type: 'FETCH_USER'; id: number }
type AsyncActionDecrement = { type: 'DELAYED_DECREMENT' };
type AsyncAction = AsyncActionFetch | AsyncActionDecrement;export type Action = ReducerAction | AsyncAction;function* userFetcher(action: AsyncActionFetch) {
  try {
    yield put<ReducerAction>({ type: 'START_FETCH_USER' });
    const response = yield call(() => fetch(`[https://jsonplaceholder.typicode.com/users/](https://jsonplaceholder.typicode.com/users/)${action.id}`));
    const data = yield call(() => response.json());
    yield delay(500);
    const { name } = data;
    if (typeof name !== 'string') throw new Error();
    yield put<ReducerAction>({ type: 'SUCCESS_FETCH_USER', name });
  } catch (e) {
    yield put<ReducerAction>({ type: 'ERROR_FETCH_USER' });
  }
}function* delayedDecrementer() {
  yield delay(500);
  yield put<ReducerAction>({ type: 'DECREMENT' });
}function* userFetchingSaga() {
  yield takeLatest<AsyncActionFetch>('FETCH_USER', userFetcher);
}function* delayedDecrementingSaga() {
  yield takeEvery<AsyncActionDecrement>('DELAYED_DECREMENT', delayedDecrementer);
}function* rootSaga() {
  yield all([
    userFetchingSaga(),
    delayedDecrementingSaga(),
  ]);
}const reducer = (state = initialState, action: ReducerAction) => {
  console.log({ state, action });
  switch (action.type) {
    case 'INCREMENT': return {
      ...state,
      count: state.count + 1,
    };
    case 'DECREMENT': return {
      ...state,
      count: state.count - 1,
    };
    case 'SET_NAME': return {
      ...state,
      person: {
        ...state.person,
        name: action.name,
      },
    };
    case 'START_FETCH_USER': return {
      ...state,
      person: {
        ...state.person,
        loading: true,
      },
    };
    case 'SUCCESS_FETCH_USER': return {
      ...state,
      person: {
        ...state.person,
        name: action.name,
        loading: false,
      },
    };
    case 'ERROR_FETCH_USER': return {
      ...state,
      person: {
        ...state.person,
        name: 'ERROR',
        loading: false,
      },
    };
    default: return state;
  }
};const store = createStore(reducer, applyMiddleware(sagaMiddleware));
sagaMiddleware.run(rootSaga);exposeStore(store);
```

上面的代码运行在一个 worker 中。

主线程中运行的代码如下:

```
import { wrapStore } from 'redux-in-worker';
import { initialState } from './store.worker';const store = wrapStore(
  new Worker('./store.worker', { type: 'module' }),
  initialState,
  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__(),
);
```

这和前面的例子一模一样。

请在存储库中找到完整的示例:

[https://github . com/Dai-Shi/redux-in-worker/tree/master/examples/05 _ saga](https://github.com/dai-shi/redux-in-worker/tree/master/examples/05_saga)

# 结束语

这种方法的最大障碍之一是[的重复。redux-thunk 采用不可序列化的函数操作。它是官方工具，也包含在 Redux Toolkit 中。这意味着这种方法不会成为主流。](https://github.com/reduxjs/redux-thunk)

但无论如何，我希望有人喜欢这种方法，并在一些真实的环境中进行评估。请随时在 [GitHub 问题](https://github.com/dai-shi/redux-in-worker/issues)中展开讨论。

顺便说一下，我为 React 开发了另一个库来使用 Web Workers。

[](https://github.com/dai-shi/react-hooks-worker) [## 戴氏/反应钩-工人

### React 为网络工作者定制钩子。Web Workers 是浏览器中主线程的另一个线程。我们可以负重奔跑…

github.com](https://github.com/dai-shi/react-hooks-worker) 

这个库可以让你脱离主线程运行任何函数。这是一个小图书馆，相当稳定。也来看看。

*原载于 2020 年 3 月 29 日 https://blog.axlight.com**的* [*。*](https://blog.axlight.com/posts/redux-in-worker-off-main-thread-redux-reducers-and-middleware/)