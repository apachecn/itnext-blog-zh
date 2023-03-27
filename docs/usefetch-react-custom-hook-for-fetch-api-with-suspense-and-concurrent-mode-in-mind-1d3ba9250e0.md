# useFetch:考虑到暂停和并发模式，对 Fetch API 的自定义挂钩做出反应

> 原文：<https://itnext.io/usefetch-react-custom-hook-for-fetch-api-with-suspense-and-concurrent-mode-in-mind-1d3ba9250e0?source=collection_archive---------1----------------------->

直到我们有了 react-cache 或用于不同的目的。

![](img/b358d9d190dc2a0db82ac67dcade9502.png)

## 介绍

虽然我不喜欢在没有钩子的情况下编写 React，但 react-cache 似乎仍然很遥远。当然，数据获取中的缓存很重要，然而我想寻求仅用 React 钩子实现的可能性。这些在未来可能会过时，但我今天想要一些东西，那就是“useFetch”。在不需要复杂缓存的情况下，如果没有 react-cache，这仍然是有用的。

我已经开始开发“useFetch”作为构建自定义钩子的教程。

[](/react-hooks-tutorial-on-developing-a-custom-hook-for-data-fetching-8ad5840db7ae) [## React Hooks 开发数据获取定制钩子教程

### 钩子进入反应 16.7。

itnext.io](/react-hooks-tutorial-on-developing-a-custom-hook-for-data-fetching-8ad5840db7ae) 

此后，添加了对 AbortController 的支持，如果您像 typeahead 建议的那样获取一次性数据，这是令人难以置信的。

现在，为了让这个“useFetch”挂钩更有用，本文描述了对悬念和并发反应的支持。

## 反应悬念

[](https://reactjs.org/docs/code-splitting.html#suspense) [## 代码分解-反应

### 用于构建用户界面的 JavaScript 库

reactjs.org](https://reactjs.org/docs/code-splitting.html#suspense) 

悬念目前在代码分割部分描述。然而，它的一般用途是捕捉组件树中的承诺并呈现回退元素。当承诺被解决时，它将呈现一棵正常的树。悬念的好处是你可以在树上的任何地方放置悬念，你甚至可以只在树的顶端放置一个悬念。

一个不确定的点是，当数据提取的暂停被释放时，该功能是否会相同。没关系。我们应该能够跟上即将到来的变化。

## 并发反应

[](https://reactjs.org/blog/2018/11/27/react-16-roadmap.html#react-16x-q2-2019-the-one-with-concurrent-mode) [## React 16.x 路线图- React 博客

### 你可能在以前的博文中听说过类似“钩子”、“悬念”和“并发渲染”的特性…

reactjs.org](https://reactjs.org/blog/2018/11/27/react-16-roadmap.html#react-16x-q2-2019-the-one-with-concurrent-mode) 

目前还没有关于并发模式的文档。据我理解，我们需要关心的是 a)渲染函数可以被多次调用，b)渲染阶段和提交阶段不同步。至于 a)，render 函数应该是纯的或者至少调用两次也不应该改变世界。对于 b)，我们不应该假设多个组件的调用顺序。不过，我可能还是有些误解。我希望将来能有一些关于 React 钩子的指导方针。

## 图书馆

[](https://github.com/dai-shi/react-hooks-fetch) [## 戴式/反应钩取

### 用于获取 API 的 React 自定义挂钩。通过在…上创建一个帐户，为 dai-shi/react-hooks-fetch 开发做出贡献

github.com](https://github.com/dai-shi/react-hooks-fetch) 

让我们看看“useFetch”是如何实现的。如果你想知道它是如何使用的，请跳到下一节。

首先，有一些效用函数，有两个值得注意的:

```
const **useForceUpdate** = () => useReducer(state => !state, false)[1];
```

这是一个自定义挂钩，用于强制重新渲染。

```
const **createPromiseResolver** = () => {
  let resolve;
  const promise = new Promise((r) => { resolve = r; });
  return { resolve, promise };
};
```

这是一种创造可以从外部解决的承诺的天真方式。

接下来是钩子的主要部分:

```
export const useFetch = (input, opts = defaultOpts) => {
  const forceUpdate = useForceUpdate();
  const error = **useRef**(null);
  const loading = **useRef**(false);
  const data = **useRef**(null);
  const promiseResolver = **useMemo**(createPromiseResolver, [input, opts]);
```

我们用`useRef`定义一些变量，用`useMemo`定义 promiseResolver。

```
 // ...continued
  useEffect(() => {
    let finished = false;
    const abortController = new AbortController();
    (async () => {
      if (!input) return;
      // start fetching
      loading.current = true;
      **forceUpdate()**;
      const onFinish = (e, d) => {
        if (!finished) {
          finished = true;
          error.current = e;
          data.current = d;
          loading.current = false;
        }
      };
      try {
        const { readBody = defaultReadBody, ...init } = opts;
        const response = await fetch(input, {
          ...init,
          signal: abortController.signal,
        });
        // check response
        if (response.ok) {
          const body = await readBody(response);
          onFinish(null, body);
        } else {
          onFinish(createFetchError(response), null);
        }
      } catch (e) {
        onFinish(e, null);
      }
      // finish fetching
      **promiseResolver.resolve()**;
    })();
    const cleanup = () => {
      if (!finished) {
        finished = true;
        **abortController.abort()**;
      }
      error.current = null;
      loading.current = false;
      data.current = null;
    };
    return cleanup;
  }, **[input, opts]**);
```

嗯，太久了。一些注意事项:a) `forceUpdate()`在变量更新时被调用。b)异步函数结束时，调用`primiseResolver.resolve()`。c)`cleanup`用于中止正在运行的获取并清除变量。d)`useEffect`的输入数组是`[input, opts]`。

```
 // ...continued
  if (loading.current) **throw** promiseResolver.promise;
  **return** {
    error: error.current,
    data: data.current,
  };
};
```

最后，它返回一些变量，但在此之前，如果它仍然处于加载状态，它抛出一个承诺，以便悬念捕捉它并显示一个后备加载组件。

## 如何使用它

让我展示一个最小的例子。

```
import React, { **Suspense** } from 'react';
import ReactDOM from 'react-dom';

import { **useFetch** } from 'react-hooks-fetch';

const Err = ({ error }) => <span>Error:{error.message}</span>;

const DisplayRemoteData = () => {
  const url = 'https://jsonplaceholder.typicode.com/posts/1';
  const { error, data } = **useFetch**(url);
  if (error) return <Err error={error} />;
  if (!data) return null;
  return <span>RemoteData:{data.title}</span>;
};

const App = () => (
  <**Suspense** fallback={<span>Loading...</span>}>
    <DisplayRemoteData />
  </**Suspense**>
);

ReactDOM.render(<App />, document.getElementById('app'));
```

使用`useFetch`的组件不处理加载状态，组件外部的`Suspense`处理并显示加载状态。一个重要的注意事项是第`if (!data) return null;`行，它是必需的，因为它最初是空的。目前这不是很好，我们想知道是否有解决方法。

你可以在 codesandbox 中试试这个例子。

[](https://codesandbox.io/s/github/dai-shi/react-hooks-fetch/tree/master/examples/01_minimal) [## react-hooks-fetch-example-code sandbox

### 为 web 应用程序定制的在线代码编辑器

codesandbox.io](https://codesandbox.io/s/github/dai-shi/react-hooks-fetch/tree/master/examples/01_minimal) 

在带有 codesandbox 链接的 GitHub repo 中，您还可以找到其他一些例子。

## 最终注释

起初，我希望使用悬念和错误边界来显示加载状态和错误状态，这样带有`useFetch`的主组件只关心成功的情况。事实证明，这或多或少涉及到黑客攻击，我最终通过一个钩子返回了正常的错误变量。在[文档](https://reactjs.org/docs/react-component.html#error-boundaries)中还有一个注释:

> 仅使用错误边界从意外异常中恢复；**不要试图用它们来控制流程。**

悬念也很难与 AbortController 合作。在写作的时候，它很像一个黑客。有兴趣的，这里的代码是[这里是](https://github.com/dai-shi/react-hooks-fetch/blob/0fbd03e6581a3e8e9ef13b4ce72ae3c2101ebc5a/examples/04_abort/src/App.tsx)。我真的想找到一个更好的方法来解决这个问题。我很好奇 react-cache 句柄是如何中止的。在此之前，我们会继续改进它。

## 变更日志

*   *【2019–02–04】:首次发布。*
*   *【2019–02–05】:提高代码简单性。*