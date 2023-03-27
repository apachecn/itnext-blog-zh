# React Hooks 开发数据获取定制钩子教程

> 原文：<https://itnext.io/react-hooks-tutorial-on-developing-a-custom-hook-for-data-fetching-8ad5840db7ae?source=collection_archive---------0----------------------->

钩子进入反应 16.7。

![](img/41d2e600d9c3ef40fbfcc7a87812479c.png)

## 介绍

React 钩子是 React 16.7 中的一个新特性。它允许我们编写有状态的功能组件，这在没有类组件的情况下是不可能的。官方文件是必读的，所以如果你还没有看的话就去看看吧。

[](https://reactjs.org/docs/hooks-intro.html) [## 介绍钩子-反应

### 钩子是一个新的特性提议，它让你不用写类就可以使用状态和其他 React 特性。他们是…

reactjs.org](https://reactjs.org/docs/hooks-intro.html) 

除了有状态功能组件之外，这个新特性允许我们构建一个定制的钩子来共享组件之间的逻辑。这对于高阶组件(hoc)来说是可能的，尽管从技术上来说钩子能达到的效果没有区别，但是钩子简化了很多，减少了所谓的“包装地狱”。这种简单性鼓励构建定制挂钩，这种趋势让我想起了 npm 的早期。

本文解释了如何编写一个定制的钩子，并展示了它如何共享逻辑，即使只有几行代码。我们以一个简单的数据获取(Fetch API)为例。

## 目标

假设读者已经学习了 React 钩子的基础，我们将从我们开发的定制钩子如何使用的目标例子开始。

```
const MyComponent = () => {
  const { error, loading, data } = useFetch('http://...');
  if (error) return <Err error={error} />;
  if (loading) return <Loading />;
  return (
    <DataView data={data} />
  );
};
```

这里，`useFetch`是自定义挂钩。不是很直观吗？`MyComponent`是一个“有状态”的功能组件，而`DataView`可以是一个无状态的功能组件，应该更具可重用性。

## 编写自定义挂钩

让我们看看如何编写一个自定义钩子来实现上述目标。自定义钩子只是一个使用其他钩子的函数。我们首先定义一个函数。

```
const useFetch = (url) => {
  // ...
};
```

这个钩子返回三个值，所以我们定义了三个状态值。请注意，我们不需要将它们组合在一个状态对象中。

```
const useFetch = (url) => {
  **const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);
  const [data, setData] = useState(null);**
  // ...
  **return { error, loading, data };**
};
```

现在，主要部分被定义在一个`useEffect`钩子中。注意，我们在输入数组中传递了`url`。

```
const useFetch = (url) => {
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);
  const [data, setData] = useState(null);
  **useEffect(() => {
    (async () => {
      setLoading(true);
      const response = await fetch(url);
      const data = await response.json();
      setData(data);
      setLoading(false);
    })();
  }, [url]);**
  return { error, loading, data };
};
```

我们还没有实现错误处理。补充一下吧。

```
const useFetch = (url) => {
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);
  const [data, setData] = useState(null);
  useEffect(() => {
    (async () => {
      setLoading(true);
      **try {**
        const response = await fetch(url);
        **if (response.ok) {**
          const data = await response.json();
          setData(data);
        **} else {
          setError(new Error(response.statusText));
        }
      } catch (e) {
        setError(e);
      }**
      setLoading(false);
    })();
  }, [url]);
  return { error, loading, data };
};
```

这是实现我们目标的基本定制钩子。它可以在几个组件中重用。您可以查看这个 codesandbox 中的工作示例。

[https://code sandbox . io/s/github/Dai-Shi/react-hooks-fetch/tree/master/examples/01 _ minimal](https://codesandbox.io/s/github/dai-shi/react-hooks-fetch/tree/master/examples/01_minimal)

## 扩展自定义挂钩

我们的钩子仍然是基本的，我们可能希望支持 POST 方法或非 JSON 数据。可以有各种方式来延长这个钩子。让我们尝试一个简单的方法，尽可能地保持获取 API。

```
const defaultOpts = {};
const useFetch = (input, **opts = defaultOpts**) => {
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);
  const [data, setData] = useState(null);
  **const {
    readBody = body => body.json(),
    ...init
  } = opts;**
  useEffect(() => {
    (async () => {
      setLoading(true);
      try {
        const response = await fetch(input, **init**);
        if (response.ok) {
          const body = await **readBody**(response);
          setData(body);
        } else {
          setError(new Error(response.statusText));
        }
      } catch (e) {
        setError(e);
      }
      setLoading(false);
    })();
  }, [input, **opts**]);
  return { error, loading, data };
};
```

这个扩展的自定义钩子的难点之一是在`useEffect`的输入数组中传递`opts`。除非用户仔细了解它的工作原理，否则可能会导致无限调用`fetch` es。

下面是一个如何使用这个定制钩子的例子。

```
const PostRemoteData = () => {
  const opts = useMemo(() => ({
    method: 'POST',
    body: JSON.stringify({
      title: 'foo',
      body: 'bar',
      userId: 1,
    }),
    readBody: body => body.text(),
  }), []);
  const { error, loading, data } = useFetch('http://...', opts);
  if (error) return <Err error={error} />;
  if (loading) return <Loading />;
  return (
    <span>Result: {data}</span>
  );
};
```

## 图书馆

虽然这只是一个很小的定制钩子，但是我把它作为一个库并在 npmjs.com 发布了它。

[](https://github.com/dai-shi/react-hooks-fetch) [## 戴式/反应钩取

### 用于获取 API 的 React 自定义挂钩。通过在…上创建一个帐户，为 dai-shi/react-hooks-fetch 开发做出贡献

github.com](https://github.com/dai-shi/react-hooks-fetch) 

## 最终注释

实际上，我不太确定 Fetch API 是否是解释构建自定义钩子的合适主题。人们可能更喜欢与视图层分离的数据提取层。然而，我希望这篇文章解释了如何开发一个定制的钩子，并有助于注意到它的可重用性。