# Typescript 函数(几乎)可变参数

> 原文：<https://itnext.io/typescript-function-almost-variadic-arguments-b4f4290cf69c?source=collection_archive---------6----------------------->

![](img/1555995cadf0139096c10de60e0c2ff5.png)

安德烈斯·梅迪纳在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Ftypescript-function-almost-variadic-arguments-b4f4290cf69c)

这是我最近遇到的一个小问题，如何为 following 函数编写 typescript 泛型？

```
**function** createAction(type,payloadCreator,metaCreator){
  **return** (...args) => {
    **return** ({
      type,
      ...(payloadCreator && {payload: payloadCreator(...args)}),
      ...(metaCreator && {meta: metaCreator(...args)}),
    })
  }
}
```

这个函数的灵感来自于一个类似的名为 redux-actions 的库。对于创建返回 redux 动作的动作创建器来说，这是一个非常有用的函数。可以按如下方式使用:

```
const incr = createAction('INCREMENT')
incr() // {type: "INCREMENT"}
incr(1) // {type: "INCREMENT"}const incr = createAction('INCREMENT', arg => arg)
incr(1) //{type: "INCREMENT", payload: 1}const incr = createAction('INCREMENT',arg=>arg,()=>({user:'admin'}))
incr(1) // {type: "INCREMENT", payload: 1, meta: {user:'admin'}}
```

在我的代码中，该函数将 enum 作为第一个类型参数，这就是为什么我希望允许 typescript 使用该类型创建 action，而不是像 redux-actions 那样使用 string。

首先你需要声明一些带多个参数的通用函数，你可以声明任意多个，这里我只有 4 个，从一个带 0 个参数的开始，到一个带 3 个参数的结束。

```
**type** ActionFunction0<R> = () => R
**type** ActionFunction1<T1, R> = (t1: T1) => R
**type** ActionFunction2<T1, T2, R> = (t1: T1, t2: T2) => R
**type** ActionFunction3<T1, T2, T3, R> = (t1: T1, t2: T2, t3: T3) => R
```

如果你需要 4 个或更多参数的函数呢？你可以继续声明更多的类型，但是最终你必须在某个地方停下来，也许 9 是个好数字。如果有人用 10 个或更多参数调用你的函数呢？在这种情况下，我们可以退回到这个版本:

```
**export type** ActionFunctionAny<R> = (...args: **any**[]) => R
```

这将无法正确处理类型，但将允许用户输入更多类型。

好了，接下来要做的是像这样添加多个重载函数声明:

```
**export function** createAction<T, P, M>(
  type: T,
  payloadCreator: ActionFunction0<P>,
  metaCreator: ActionFunction0<M>,
): ActionFunction0<Action<T,P>>

**export function** createAction<T, P, M, A1>(
  type: T,
  payloadCreator: ActionFunction1<A1,P>,
  metaCreator: ActionFunction1<A1,M>,
): ActionFunction1<A1, Action<T,P>>

**export function** createAction<T, P, M, A1, A2>(
  type: T,
  payloadCreator: ActionFunction2<A1,A2,P>,
  metaCreator: ActionFunction2<A1,A2,M>,
): ActionFunction2<A1,A2,Action<T,P>>

**export function** createAction<T, P, M, A1, A2, A3>(
  type: T,
  payloadCreator: ActionFunction3<A1,A2,A3,P>,
  metaCreator: ActionFunction3<A1,A2,A3,M>,
): ActionFunction3<A1,A2,A3,Action<T,P>>
```

最后，我们添加我们的实现。以下是参数最多的版本:

```
**export function** createAction<T, P, M, A1, A2, A3>(
  type: T,
  payloadCreator: ActionFunction3<A1,A2,A3,P>,
  metaCreator: ActionFunction3<A1,A2,A3,M>,
): ActionFunction3<A1,A2,A3, Action<T,P>> {
  **return** (a1:A1,a2:A2,a3:A3) => {
    **return** ({
      type,
      ...(payloadCreator && { payload: payloadCreator(a1,a2,a3) }),
      ...(metaCreator && { meta: metaCreator(a1, a2, a3) }),
    })
  }
}
```

现在，typescript 可以正确识别所有类型。