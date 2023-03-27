# 用 AbortController 为 abortable 异步函数开发 React 自定义钩子

> 原文：<https://itnext.io/developing-react-custom-hooks-for-abortable-async-functions-with-abortcontroller-1f2091b5a1c3?source=collection_archive---------1----------------------->

寻求反应钩的可能性。

![](img/d7705ecf0227d6f5e83195dd29e5ff09.png)

在我的上一篇文章中，我介绍了定制钩子`useAsyncTask`如何用 AbortController 处理异步函数，并演示了一个 typeahead 搜索示例。在本文中，我解释了关于`useAsyncTask`的实现。

## 概述

JavaScript promise 是不可放弃的。fetch API 是基于 promise 的，因此你不能在纯 JavaScript 中取消它。为了取消`fetch`，DOM 规范引入了`AbortController`。AbortController 是一个通用接口，不是`fetch`专用的。从技术上讲，我们可以用它来取消承诺，如果有一种简单的方法来处理可移植的异步函数就好了。在 React 世界中，我们期待着 Hooks API 的出现。我已经开始了一个项目，为可移植的异步函数实现定制钩子。

[](https://github.com/dai-shi/react-hooks-async) [## 戴式/反应式挂钩异步

### 带 React 钩子的异步函数库——Dai-Shi/React-Hooks-async

github.com](https://github.com/dai-shi/react-hooks-async) [](https://www.npmjs.com/package/react-hooks-async) [## 反应-钩子-异步

### 一个带 React 钩子的可移植异步函数库

www.npmjs.com](https://www.npmjs.com/package/react-hooks-async) 

## `useAsyncTask`的实施

`useAsyncTask`钩子是为了创建一个可以由`useAsyncRun`启动的异步任务，我们将在本文稍后描述。要创建一个异步任务，需要传递一个接收 AbortController 实例并返回 abortable 承诺的函数。作为一个具体的例子，我们稍后描述如何为`fetch`实现一个定制钩子。

代码的主要部分如下。

```
const useAsyncTask = (func, inputs) => {
  const forceUpdate = **useForceUpdate**();
  const task = **useRef**({});
  const newTask = **useMemo**(() => **createTask**(func, (t) => {
    if (task.current && task.current.taskId === t.taskId) {
      task.current = t;
      **forceUpdate**();
    }
  }), **inputs**);
  if (task.current && task.current.taskId !== newTask.taskId) {
    task.current = newTask;
  }
  **useEffect**(() => {
    const cleanup = () => { task.current = null; };
    return cleanup;
  }, **[]**);
  return task.current;
};
```

阅读这段代码需要注意三点:

*   `task`对象由`useRef`定义，重渲染由`forceUpdate`控制。
*   `createTask`是核心部分，在第二个参数中取一个回调函数，用`useMemo`调用，只有`inputs`被改变。
*   `useEffect`是清理`task`对象，以便卸载后不尝试渲染。

`useForceUpdate`挂钩是通用挂钩，通过以下实施。

```
const forcedReducer = state => !state;
const useForceUpdate = () => useReducer(forcedReducer, false)[1];
```

`createTask`的代码有点长。

```
const createTask = (func, notifyUpdate) => {
  const **taskId** = Symbol(`async_task_id_${idCounter += 1}`);
  let abortController = null;
  let task = {
    taskId,
    started: false,
    pending: true,
    error: null,
    result: null,
    **start**: async () => {
      if (task.started) return;
      abortController = new AbortController();
      task = { ...task, started: true };
      **notifyUpdate(task);**
      try {
        const result = **await func(abortController)**;
        task = { ...task, pending: false, result };
      } catch (e) {
        task = { ...task, pending: false, error: e };
      }
      **notifyUpdate(task);**
    },
    **abort**: () => {
      if (abortController) {
        abortController.abort();
      }
    },
  };
  return task;
};
```

*   对象具有`taskId`和四个属性:`started`、`pending`、`error`和`result`。
*   除此之外，`task`对象还包含控制执行的`start()`和`abort()`功能。
*   调用`start()`时，根据异步状态变化回呼`notifyChange()`。
*   第一个参数`func`是接收 AbortController 实例并返回承诺的函数。`func`负责正确处理 AbortController。

## useAsyncRun 的实现

`useAsyncTask`挂钩只是为了创建一个异步任务并使其准备好启动。`useAsyncRun`挂钩是实际启动异步任务的挂钩。我们将逻辑拆分为两个挂钩的原因是允许组合多个异步任务。`useAsyncCombineSeq` hook 是用于合并的，但我们在本文中不再赘述。

这个代码相当简单。

```
const useAsyncRun = (asyncTask) => {
  useEffect(() => {
    if (asyncTask) {
      asyncTask.start();
    }
    const cleanup = () => {
      if (asyncTask) {
        asyncTask.abort();
      }
    };
    return cleanup;
  }, **[asyncTask && asyncTask.taskId]**);
};
```

简称为`start()`和`abort()`。请注意，输入数组被传递给了`useEffect`的第二个参数。

## useAsyncTaskFetch 的实现

`useAsyncTaskFetch`挂钩是`useAsyncTask`的包装功能。基本实现如下。功能齐全的是 GitHub 存储库中的[(此处为](https://github.com/dai-shi/react-hooks-async/blob/master/src/use-async-task-fetch.js))。

```
const useAsyncTaskFetch = url => useAsyncTask(
  async (**abortController**) => {
    const response = await fetch(url, {
      **signal: abortController.signal,**
    });
    if (!response.ok) {
      throw new Error(response.statusText);
    }
    const body = await response.json();
    return body;
  },
  **[url]**,
);
```

注意，AbortController 信号被传递到`fetch`。这是因为 Fetch API 支持 AbortController。对其他人来说，你需要实现处理它。比如这里可以看看`useAsyncTaskAxios`是如何实现[的。](https://github.com/dai-shi/react-hooks-async/blob/master/src/use-async-task-axios.js)

## 摘要

本文展示了如何实现`useAsyncTask`和其他挂钩。我希望他们在使用 AbortController 的约定上是直截了当的。我正在寻找除`fetch`之外的其他使用案例，如果有任何反馈，我将非常乐意。