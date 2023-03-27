# 为什么在 React SPA 中使用基于散列的 URL 会比你想象的节省更多时间？

> 原文：<https://itnext.io/why-using-hash-based-urls-in-your-react-spa-will-save-you-more-time-than-you-think-a21e2c560879?source=collection_archive---------0----------------------->

![](img/f94c3b3bbd2e982ee96be485a6e26980.png)

卡斯帕·卡米尔·鲁宾在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

当你开发你的单页应用程序时，有时你也需要设计 url 结构。

通常开发者使用两种类型的 url…

# 正式网址

这是我们习惯看到的 url，用正常的路径，像这样:

```
https://myspa.com/app/login
https://myspa.com/app/dashboard
```

这种 url 具有 SEO 的优势，但是您必须创建与应用程序路由一样多的后端端点，以避免在重新加载 url 或将 url 传递给其他人时出现 404 错误代码消息。

# 基于哈希的 Url

这个 url 有一个唯一的应用程序端点`/app/`,由一个单一的后端动作和多个定义 SPA 路由的 Javascript 管理的 hashtags 控制。

```
https://myspa.com/app/**#/login**
https://myspa.com/app/**#/dashboard**
```

在这种情况下，我们只需要为应用程序创建一个控制器。这样做前端团队只会使用后端 API，分离前端和后端逻辑，保持 ***清晰整体结构*** ！另一方面，SEO 更难管理(但这是相对的，当我们谈论 SPA 时)。

# 在 React 中实现基于散列的 Url

为了在我们的 SPA 中用 React 实现基于*散列的 url* ，我选择使用[连接的 React 路由器](https://github.com/supasate/connected-react-router)库。

该库可以:

*   通过单向流将路由器状态与 redux 存储同步
*   支持 React 路由器 v4 和 v5
*   支持功能组件热重装，同时保留状态
*   历史方法调度(`push`、`replace`、`go`、`goBack`、`goForward`)对 [redux-thunk](https://github.com/gaearon/redux-thunk) 和 [redux-saga](https://github.com/yelouafi/redux-saga) 都有效
*   还有更多！

所以首先我们需要安装它

```
yarn add connected-react-router
```

之后，我们需要用特定的键`**router**`创建一个减速器

```
// reducers.js
import { combineReducers } from 'redux'
import { connectRouter } from 'connected-react-router'

export default (history) => combineReducers({
  router: connectRouter(history),
  ... // rest of your reducers
})
```

现在我们添加相关的中间件，这样我们就可以调度历史动作

```
// configureStore.js
...
import { createHashHistory } from 'history'
import { applyMiddleware, compose, createStore } from 'redux'
import { routerMiddleware } from 'connected-react-router'
import createRootReducer from './reducers'
...
**export const history = createHashHistory({
    hashType: 'slash',
    getUserConfirmation: (message, callback) => callback(window.confirm(message))
});**

export default function configureStore(preloadedState) {
  const store = createStore(
    createRootReducer(history), // root reducer with router state
    preloadedState,
    compose(
      applyMiddleware(
        routerMiddleware(history), // for dispatching history actions
        // ... other middlewares ...
      ),
    ),
  )

  return store
}
```

`history`包提供了 3 种不同的方法来创建一个`history`对象，这取决于你的环境。

*   `createBrowserHistory`用于支持 [HTML5 历史 API](http://diveintohtml5.info/history.html) 的现代网络浏览器中(参见[跨浏览器兼容性](http://caniuse.com/#feat=history))
*   `createMemoryHistory`用作参考实现，也可以用于非 DOM 环境，比如 [React Native](https://facebook.github.io/react-native/) 或 tests
*   `createHashHistory`用于传统的网络浏览器

我们使用 *createHashHistory* 而不是 *createBrowserHistory* ，以便使用基于散列的 URL*。*

最后检索 ConnectedRouter，并将创建的历史传递给它。

```
// index.js
...
import { Provider } from 'react-redux'
import { Route, Switch } from 'react-router' // react-router v4/v5
import { ConnectedRouter } from 'connected-react-router'
import configureStore, { history } from './configureStore'
...
const store = configureStore(/* provide initial state if any */)

ReactDOM.render(
  <Provider store={store}>
    <**ConnectedRouter history={history}**> { /* place ConnectedRouter under Provider */ }
      <> { /* your usual react-router v4/v5 routing */ }
        <Switch>
          <Route exact path="/" render={() => (<div>Match</div>)} />
          <Route render={() => (<div>Miss</div>)} />
        </Switch>
      </>
    </ConnectedRouter>
  </Provider>,
  document.getElementById('react-root')
)
```

尽情享受吧！😉

感谢阅读，如果你喜欢这篇文章，请留下评论，不要忘记鼓掌！😃