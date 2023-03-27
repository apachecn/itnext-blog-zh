# Redux Saga 通用 api 函数

> 原文：<https://itnext.io/redux-saga-generic-api-function-c99ee5647e29?source=collection_archive---------0----------------------->

![](img/4e66691aa668401a60ef7d108953cb0c.png)

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fredux-saga-generic-api-function-c99ee5647e29)

想象一下这个 saga.js 主文件，它定义了两个函数，但是还有更多的函数:

```
import { takeLatest, put, call } from 'redux-saga/effects';
import actions from '../actions';
import api from '../api';function* getFromServer(params) {
 try {
        const data = yield call(api.getFromServer, ...params);
        yield put({ type: action.GET_FROM_SERVER_SUCCESS, data });
    } catch (error) {
        yield put({ type: actions.GET_FROM_SERVER_ERROR, error })
    }
}function* getOtherStuff(params) {
 try {
        const data = yield call(api.getOtherStuff, ...params);
        yield put({ type: action.GET_OTHER_STUFF_SUCCESS, data });
    } catch (error) {
        yield put({ type: actions.GET_OTHER_STUFF_ERROR, error })
    }
}export function* sagas() {
    yield takeLatest(actions.GET_FROM_SERVER, getFromServer);
    yield takeLatest(actions.GET_OTHER_STUFF, getOtherStuff);
}
```

在这里，saga 采用最新的 GET_FROM_SERVER 和 GET_OTHER_STUFF 操作，并从 api/index.js 调用函数，其中实际的服务器通信是解耦的。在这个结构中，我们需要在 actions/index.js 中为每个端点定义三个不同的操作，一个 sagas/index.js 中的新函数或者一个新文件和一个导入。与其让它变大，不如让它变得更通用，就像这样:

```
function* generic(...data) {
    let func = data[0];
    let params = data[1];
    let type = params.type;
 try {
        const data = yield call(api[func], params);
        yield put({ type: actions[type] + actions.SUCCESS, data });
    } catch (error) {
        console.log('saga fail: ', error);
        yield put({ type: actions[type] + actions.ERROR, error })
    }
}export function* sagas() {
    yield takeLatest(actions.GET_FROM_SERVER, generic, 'getFromServer');
    yield takeLatest(actions.GET_OTHER_STUFF, generic, 'getOtherStuff');
}
```

在这里，我们有一个通用的 api 调用者，可以用于主 sagas()函数中的任何产出。我们只需要在 sagas()函数中添加一行代码就完成了。还有，在 actions/index.js 中，我们只需要定义 _SUCCESS 和 _ERROR 两个后缀，它们在 saga 和 redux 通信中使用，这样就减少了动作类型的数量。

您甚至可以更进一步，在分派中传递函数名，这样甚至不再需要接触 saga，而是包含容器、api 调用者和 reducer 之间的通信。

如果你喜欢这篇文章，给我一些掌声，我会发布一个通用的 api 调用太:D