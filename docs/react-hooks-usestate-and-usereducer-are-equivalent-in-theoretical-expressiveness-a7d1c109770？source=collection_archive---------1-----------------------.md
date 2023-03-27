# React 钩子 useState 和 useReducer 在理论表达上是等价的

> 原文：<https://itnext.io/react-hooks-usestate-and-usereducer-are-equivalent-in-theoretical-expressiveness-a7d1c109770?source=collection_archive---------1----------------------->

## 你喜欢哪一个？

![](img/845682981f3b59719bd0e0b7c7591de9.png)

# 介绍

`useReducer`是一个强有力的钩子。已知`useState`是用`useReducer`实现的。

在 React hooks [docs](https://reactjs.org/docs/hooks-reference.html#usereducer) 中，这样注明:

> 当你有涉及多个子值的复杂状态逻辑或者下一个状态依赖于前一个状态时，useReducer 通常比 useState 更好。useReducer 还允许您优化触发深度更新的组件的性能，因为您可以向下传递分派而不是回调。

很长一段时间，我误解了 useReducer 比 useState 更强大，并且有一些优化是 useState 无法实现的。

事实证明，useState 在表现力上和 useReducder 一样强大。这是因为使用状态允许[功能更新](https://reactjs.org/docs/hooks-reference.html#functional-updates)。即使深度更新，您也可以传递自定义回调。

因此，您是使用 State 还是 useReducer 只是您的偏好。当我在 JSX 使用`dispatch`的时候，我会使用 Reducer。在 JSX 之外进行逻辑分析对我来说似乎是干净的。

# 例子

如果你创建了一个自定义钩子，无论你使用的是 useState 还是 useReducer 都只是一个内部实现的问题。让我们看一个例子。我们用两个钩子实现了一个简单的反例。对于这两种情况，钩子都返回动作回调，这对于在比较中隐藏实现细节很重要。

## **useReducer**

```
const initialState = { count1: 0, count2: 0 };const reducer = (state, action) => {
  switch (action.type) {
    case 'setCount1':
      if (state.count1 === action.value) return state; // to bail out
      return { ...state, count1: action.value };
    case 'setCount2':
      if (state.count2 === action.value) return state; // to bail out
      return { ...state, count2: action.value };
    default:
      throw new Error('unknown action type');
  }
};const useCounter = () => {
  const [state, dispatch] = useReducer(reducer, initialState);
  const setCount1 = useCallback(value => {
    dispatch({ type: 'setCount1', value });
  }, []);
  const setCount2 = useCallback(value => {
    dispatch({ type: 'setCount2', value });
  }, []);
  return { ...state, setCount1, setCount2 };
};
```

## **使用状态**

```
const initialState = { count1: 0, count2: 0 };const useCounter = () => {
  const [state, setState] = useState(initialState);
  const setCount1 = useCallback(value => {
    setState(prevState => {
      if (prevState.count1 === value) return prevState; // to bail out
      return { ...state, count1: value };
    });
  }, []);
  const setCount2 = useCallback(value => {
    setState(prevState => {
      if (prevState.count2 === value) return prevState; // to bail out
      return { ...state, count2: value };
    });
  }, []);
  return { ...state, setCount1, setCount2 };
};
```

你觉得哪个舒服？

# 奖金

如果 useState 和 useReducer 一样强大，那么 useReducer 应该可以和 userland 中的 useState 一起实现。

```
const useReducer = (reducer, initialArg, init) => {
  const [state, setState] = useState(
    init ? () => init(initialArg) : initialArg,
  );
  const dispatch = useCallback(
    action => setState(prev => reducer(prev, action)),
    [reducer],
  );
  return useMemo(() => [state, dispatch], [state, dispatch]);
};
```

# 最终注释

我的大多数库都是用 useReducer 编写的，但是我可能会改变我的想法，在更合适的时候考虑使用 useState。

我确实改变了我的实验图书馆。这里的差异是。

[](https://github.com/dai-shi/react-hooks-fetch) [## 戴式/反应钩取

### 用于获取 API 的 React 自定义挂钩。通过在…上创建一个帐户，为 dai-shi/react-hooks-fetch 开发做出贡献

github.com](https://github.com/dai-shi/react-hooks-fetch) 

最后一点，至于单元测试，我确信分离的`reducer`更容易测试。

*原载于 2019 年 6 月 30 日*[*https://blog.axlight.com*](https://blog.axlight.com/posts/react-hooks-usestate-and-usereducer-are-equivalent-in-theoretical-expressiveness/)*。*