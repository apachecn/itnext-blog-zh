# 使用 Redux 可观察测试进行史诗级测试

> 原文：<https://itnext.io/going-epic-with-redux-observable-tests-dd42b80ee4f8?source=collection_archive---------0----------------------->

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fgoing-epic-with-redux-observable-tests-dd42b80ee4f8)

我开发的最后几个 React 项目是在 **Redux Observable** 库的大量帮助下构建的。这是一个很好的将业务逻辑从组件中分离出来的库，但是正确的测试方法仍然是他们需要找到的。在这篇文章中，我将分享我对这个话题的想法。

# 那么这个 Redux 可观测是什么呢？

对于那些不了解这个库的人，我推荐你去看看 [RxJS + Redux + React = Amazing！杰伊·菲尔普斯的谈话。这是一个非常鼓舞人心的演讲，讲述了网飞如何使用一些常见的 JS 模式结合](https://www.youtube.com/watch?v=AslncyG8whg) [RxJS](https://www.robinvdvleuten.nl/blog/going-epic-with-redux-observable-tests/(https://github.com/Reactive-Extensions/RxJS) 的功能来管理 React 应用程序中的业务逻辑。他们从网飞提取了核心，并在 Github 上作为开源库[共享。](https://github.com/redux-observable/redux-observable/)

![](img/c69df2bb168993c1078ec29e48d1fed1.png)

他们的[文档](https://redux-observable.js.org/)非常优秀，包含了许多小的运行示例来帮助你开始。整个图书馆本身就值得写一篇文章，但是有一个重要的方面仍然没有得到充分的展示。事实上，他们自己还在为最佳途径而挣扎；

> 测试会产生副作用的异步代码并不容易。我们仍在学习测试史诗的最佳方法。如果你找到了完美的方法，请分享！”
> — [*Redux 可观察文档*](https://redux-observable.js.org/docs/recipes/WritingTests.html)

在与几个项目上的可观察测试进行斗争之后，我想在本文中就这个主题发表我的两点意见。

# 我们要测试什么史诗？

为了更好地展示如何测试异步业务逻辑，我想到了以下方法:

```
export const authenticateUserEpic = (action$, store, { client }) => {
  // Only act on actions of a given type,
  // in this case "USER_AUTHENTICATE_REQUEST".
  return action$.ofType('USER_AUTHENTICATE_REQUEST')
    // Map the received action to a new action "observable".
    .switchMap(action => {
      // Authenticate with the dispatched credentials in the action,
      // using the injected client instance.
      return client.authenticate(action.username, action.password)
        .then(response => {
          if (!response.isSuccessful) {
            // Map the response to a "failed" action with the error.
            return {
              type: 'USER_AUTHENTICATE_FAILURE',
              error: 'Something went wrong while authenticating',
            };
          } return {
            // Map the response to a "successful" action with a JWT token.
            type: 'USER_AUTHENTICATE_SUCCESS',
            idToken: response.idToken,
          };
        });
    });
}
```

您可能已经注意到，这是一部关于使用发送的凭证对用户进行身份验证的史诗。我可以想象我会派遣这样的行动；

```
export const authenticate = (username, password) {
  return { type: 'USER_AUTHENTICATE_REQUEST', username, password };
}dispatch(authenticate('johndoe', 'mysupersecretpassword'));
```

你可能也注意到了，我已经将客户端依赖关系注入到我的 epic 中。您可以通过 *require* 或 *import* 语句获得一个客户端实例。但是通过使用*依赖注入*，它使得客户端方式更容易被模仿，而你的 epic 方式更容易被测试。

# 用 Jest 创建测试

大多数 React 项目似乎都在使用 [Jest](https://facebook.github.io/jest/) ，所以我只在示例测试中使用它。

我测试上述 epic 的方法是，当 epic 接收到*分派的*动作时，得到*预期的*动作。因此，对 epic 的快速浏览告诉我们，我们需要两个测试；一个是我们期望的带有 JWT 令牌的“用户验证成功”,另一个是我们期望的带有错误的“用户验证失败”。要将它们定义为 Jest 测试，可以这样定义它们:

```
describe('authenticateUserEpic', () => {
  it('should dispatch a JWT token when authenticating is successful', () => {
    // ...
  }) it('should dispatch an error when authenticating has failed', () => {
    // ...
  })
});
```

所以现在让我们把注意力集中在第一个测试上。我们需要向 epic 传递*调度*动作，并在 RxJS 观察完成时获得结果动作。有许多方法可以编写这样的代码，但是下面的方法最适合我；

```
import { ActionsObservable } from 'redux-observable';
import authenticateUserEpic from './epics';// ...it('should dispatch a JWT token when authenticating is successful', async () => {
  // The response object we expect to receive from the server.
  const response = {
    isSuccessful: true,
    idToken: 'a-random-generated-jwt',
  }; // Create a fake client instance which will return
  const client = { authenticate: jest.fn() };
  client.authenticate.mockReturnValue(Promise.resolve(response)); // Create an Observable stream of the dispatching action.
  const action$ = ActonsObservable.of({
    type: 'USER_AUTHENTICATE_REQUEST',
    username: 'johndoe',
    password: 'mysupersecretpassword',
  }); // Pass the Observable action to our action and inject the
  // mocked client instance.
  const epic$ = authenticateUserEpic(action$, store, { client }); // Get the resulting actions by using async/await.
  const result = await epic$.toArray().toPromise(); // Test if we've received the expected action as result.
  expect(result).toEqual([
    { type: 'USER_AUTHENTICATE_SUCCESS', idToken: 'a-random-generated-jwt' }
  ])
});
```

没那么难吧。你首先需要了解 RxJS。但是在这之后，您将在 React 应用程序中很好地分离关注点。为了使示例完整，下面的测试将处理失败的响应；

```
it('should dispatch an error when authenticating has failed', async () => {
  // The response object we expect to receive from the server.
  const response = {
    isSuccessful: false,
  }; // Create a fake client instance which will return
  const client = { authenticate: jest.fn() };
  client.authenticate.mockReturnValue(Promise.resolve(response)); // Create an Observable stream of the dispatching action.
  const action$ = ActonsObservable.of({
    type: 'USER_AUTHENTICATE_REQUEST',
    username: 'johndoe',
    password: 'mysupersecretpassword',
  }); // Pass the Observable action to our action and inject the
  // mocked client instance.
  const epic$ = authenticateUserEpic(action$, store, { client }); // Get the resulting actions by using async/await.
  const result = await epic$.toArray().toPromise(); // Test if we've received the expected action as result.
  expect(result).toEqual([
    { type: 'USER_AUTHENTICATE_FAILURE', error: 'Something went wrong while authenticating' }
  ])
});
```

一路上我有没有头疼？在对 RxJS 有一个基本的了解之前我肯定是得了[一些](https://stackoverflow.com/questions/42276419/invoking-epics-from-within-other-epics) [题](https://stackoverflow.com/questions/47037119/testing-observable-epic-which-invokes-other-epic)！但幸运的是，Redux Observable 非常有用。现在我有了一个非常有价值的新工具来构建我的 React 应用程序👌

```
Originally published at [www.robinvdvleuten.nl](https://www.robinvdvleuten.nl/blog/going-epic-with-redux-observable-tests/?utm_source=medium&utm_medium=article)
```

*希望它帮助你用 React 和 RxJS 构建了一些令人惊奇的东西。请在下方按* ***推荐*** *展示一些欣赏或在评论中分享你的想法。*

如果你正在做类似的事情，并且有任何你认为我可以帮忙的问题，我是推特上的[*@ robinvdvleuten*](https://twitter.com/robinvdvleuten)*！*