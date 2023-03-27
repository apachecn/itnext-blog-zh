# 使用 webpack、react、redux、router、saga 和 postcss 的快速现代前端设置，第 3 部分

> 原文：<https://itnext.io/fast-and-modern-front-end-setup-with-webpack-react-redux-router-saga-and-postcss-part-3-27ac4bc3f969?source=collection_archive---------1----------------------->

![](img/b0edce9ca5bf04674727450842a80dbd.png)

希望意外三部曲《:D》的最后一部

让我们开始吧。我们现在有了 webpack 设置、样式、反应和还原。我们需要管理路由和 api 调用来完成我们的基本设置。

如果我们添加一个组件，这两个问题我们会更好地理解，所以让我们开始吧。在 src/components/ add Menu.js 中:

```
import React from 'react';
import PropTypes from 'prop-types';import AppBar from 'material-ui/AppBar';
import FlatButton from 'material-ui/FlatButton';const Menu = props => <AppBar
    title="Our project"
    onTitleClick={props.goHome}
    iconElementRight={<FlatButton label="Show stuff" />}
    onRightIconButtonClick={props.onClick}
/>Menu.propTypes = {
 goHome: PropTypes.func.isRequired
}Menu.defaultProps = {
 goHome: () => console.log('going home')
}export default Menu;
```

并将其包含在我们的 App.js 中:

```
import React from 'react';
import PropTypes from 'prop-types';import RaisedButton from 'material-ui/RaisedButton';import Menu from './Menu';const App = props => <div>
 <Menu />
 <RaisedButton label={props.buttonText} onClick={props.onClick} />
</div>App.propTypes = {
 buttonText: PropTypes.string.isRequired,
 onClick: PropTypes.func.isRequired
}App.defaultProps = {
 buttonText: 'defaultText',
 onClick: () => console.log('default click action')
}export default App;
```

我们可以看到我们的菜单。

我们想要的是，当用户点击 Show stuff 时，我们向后端发送一个请求，我们接收一个 json，我们将该 json 简化为一种状态，然后在另一个页面上显示该状态。为了处理 API 调用，redux 使用中间件，所以让我们设置 saga:

```
npm i -S redux-saga
```

中间件的工作方式很像 webpack 中加载器的工作方式:它们知道在把控制权交还给所有者之前，它们想要处理什么样的事情，它们知道如何处理它，你只需要设置它。那么让我们来设置中间件。我们将在 src/文件夹中创建一个名为 store.js 的新文件:

```
import { createStore, applyMiddleware, compose } from "redux";
import createSagaMiddleware from "redux-saga";
import { reducers } from "./reducers/index";
import { sagas } from "./sagas/index";let middlewares = [];const sagaMiddleware = createSagaMiddleware();
middlewares.push(sagaMiddleware);let middleware = applyMiddleware(...middlewares);const store = createStore(reducers, middleware);
sagaMiddleware.run(sagas);export { store };
```

因此，我们制作了一个中间件数组，然后使用中间件和 reducers 创建了商店。我们还使用 src/sagas/index 中的 sagas，我们将在将它导入 index.js 后使用它，所以删除

```
let store = createStore(reducers);
```

并补充:

```
import {store} from './store';
```

然后在 src/做一个新文件夹叫 sagas。这是我们将处理所有异步请求的地方，从而保持我们的 reducers 的纯净。制作索引. js:

```
import { takeLatest, put, call } from 'redux-saga/effects'
import actions from '../actions'
import api from '../api';function* getStuff() {
 try {
        const data = yield call(api.getStuff);
        yield put({ type: actions.GOT_STUFF, data });
    } catch (error) {
        console.log('saga fail: ', error);
        yield put({ type: actions.GOT_NO_STUFF, error })
    }
}export function* sagas() {
    yield takeLatest(actions.GET_STUFF, getStuff);
}
```

显然，我们需要将这些动作添加到我们的动作列表中:

```
export default {
    BASIC_ACTION: 'BASIC_ACTION',
    GET_STUFF: 'GET_STUFF',
    GOT_STUFF: 'GOT_STUFF',
    GOT_NO_STUFF: 'GOT_NO_STUFF'
}
```

函数声明中的*表示它是一个生成器函数。简单地说，这个函数不需要立刻运行，它可以产生一些结果，然后在以后继续运行。

要理解这一点，请检查主要的 sagas()函数。在这里，我们采取了最新 GET_STUFF 操作。由于我们在一个中间件中，saga 将检查所有传递类型。当有匹配时，它将处理它。TakeLatest 意味着如果有更多的并发请求，将只处理最新的请求，这很好。在他们的官方页面上查看更多选项。

我们取一个最新，然后通过另一个生成器，得到一个东西，有三个产量。首先，我们调用一个 api.getStuff 函数(我们马上就会用到它)。这个函数发送一个请求，我们的生成器传递回控制并等待。当我们得到数据时，我们把它放回 redux，传递一个将在 reducer 中使用的类型。如果不是跳来跳去，让我们更新 reducers/index.js:

```
import { combineReducers } from "redux";
import basicReducer from './basicReducer';
import stuffReducer from './stuffReducer';export const reducers = combineReducers({
  text: basicReducer,
  stuff: stuffReducer
});
```

并生成 stuffReducer.js:

```
import actions from '../actions';const stuffReducer = (state = [], action) => {
 switch(action.type) {
  case actions.GOT_STUFF: return action.data
  default: return state
 }
}export default stuffReducer;
```

请注意，我们不处理 GET_STUFF，我们等待 saga 向我们发送数据，只有到那时我们才减少它。

现在只需在 src/ folder 中添加 api/ folder，并在那里进行 api 调用:

```
import axios from 'axios';
import actions from '../actions';const makeRequest = (urlExtension, data = {}) => axios.post(config.baseUrl + urlExtension, data, {withCredentials: true});export default {
 getStuff: () => makeRequest('getStuff.php')
}
```

我用 axios 做这些事情，所以你要么必须用`npm i -S axios`要么用你自己的。无论如何，这将我们的状态管理逻辑与我们的异步逻辑解耦，并将我们的异步逻辑与我们的外部通信逻辑解耦。

很好，现在我们已经处理好了，我们为菜单组件准备了一个容器。正如我们已经知道的，这个容器将充当中间人(也是:D 中间的东西)并定义实际的 onClick 函数。制作 src/containers/Menu.js:

```
import { connect } from 'react-redux';
import actions from '../actions/';import TheComponent from '../components/Menu';const mapStateToProps = (state, ownProps) => {
    return {
    }
}const mapDispatchToProps = (dispatch, ownProps) => {
    return {
        onClick: () => {
          dispatch({type: actions.GET_STUFF})
        }
    }
}const Menu = connect(
    mapStateToProps,
    mapDispatchToProps
)(TheComponent)export default Menu;
```

并将该文件导入 components/App.js

```
import Menu from '../containers/Menu';
```

启动应用程序并单击按钮。您将在控制台中看到一个 saga 失败，但这是因为我们没有后端端点。本教程不包括这一点，所以让我们只是回顾一下我们到目前为止:

1.  在 actions/ folder 中，我们保存了一个所有可能发生的事情的列表
2.  在 containers/ folder 中，我们定义了一些函数来调度这些动作中的一个，以防组件中发生了一些事情
3.  被分派的对象现在通过中间件。如果没有匹配，它就去减速器。如果有匹配，saga 通常通过调用外部 api 来处理它。
4.  在 api/ folder 中，我们可以制作几个文件来处理不同的端点，我们只是导出函数。
5.  saga 使用这些函数中的一个从服务器获取一些数据，然后用这些数据调度另一个动作
6.  Redux 收回控制权，现在寻找一个 reducer 来处理新的动作并更新状态树。

太好了，希望这足够简单，:D

但我们希望用户能够点击后退按钮，返回到某个先前的屏幕，在东西被显示在某个地方之前。为此，我们需要路由，我们将导入我们的 react-router:

```
npm i -S react-router@3.0.2 react-router-redux
```

我们选择 react-router 的较低版本是因为^4.0.0 与其他库之间存在一些问题，但是您可以自行研究它。

我们将让路由器处理路由、历史等，但是 react-router-redux 将使数据在状态树中可用。我们来更新一下 store.js:

```
import { createStore, applyMiddleware, compose } from "redux";
import createSagaMiddleware from "redux-saga";
import { reducers } from "./reducers/index";
import { sagas } from "./sagas/index";import { browserHistory } from "react-router";
import { syncHistoryWithStore, routerMiddleware } from "react-router-redux";let middlewares = [];middlewares.push(routerMiddleware(browserHistory));const sagaMiddleware = createSagaMiddleware();
middlewares.push(sagaMiddleware);let middleware = applyMiddleware(...middlewares);const store = createStore(reducers, middleware);
const history = syncHistoryWithStore(browserHistory, store);
sagaMiddleware.run(sagas);export { store, history };
```

我们传递另一个中间件，这个中间件处理 browserHistory，并导出这个历史。我们将能够像这样使用这个历史:

```
<Router history={history}>...
```

当然是导入之后。我们还需要为此添加一个 reducer，请访问 reducers/index.js:

```
import { combineReducers } from "redux";
import basicReducer from './basicReducer';
import stuffReducer from './stuffReducer';
import { routerReducer } from "react-router-redux";export const reducers = combineReducers({
  routing: routerReducer,
  text: basicReducer,
  stuff: stuffReducer
});
```

您可以查看路由对象，看看它能提供什么。

为了不让这篇已经很长的教程变得更长，我将在这里停下来，避免解释 react-router。

我希望这很容易理解。如果没有，请对容易混淆的部分发表评论。