# 将几个 api 函数组合成一个

> 原文：<https://itnext.io/combining-several-api-functions-into-one-a5177d17eae9?source=collection_archive---------7----------------------->

![](img/12753c2991fd0b6554d588c90ac03db6.png)

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fcombining-several-api-functions-into-one-a5177d17eae9)

情况是这样的:

1.  你有一个特殊的文件，它只导出一堆函数。
2.  这些函数都是 api 调用，它们接受一些参数并给出一个承诺。

```
import axios from 'axios';export default {
    getSomeStuff: (url, param1, param2) => axios.post(url, {param1, param2}), 
    putSomeStuff: (url, info1, info2) => axios.post(url, {info1, info2}),
    ...
}
```

代码中经常发生的事情，特别是当你使用许多第三方库来处理应用程序的不同部分时，是每个部分分别处理所有事情。这听起来不错，因为这才是重点。

但是在上面这种情况下，完全解耦的 api.js 模块要求您在添加或更改内容(比如添加新的端点)时更新它。虽然应用程序中用户点击按钮的部分和数据实际到达服务器并返回的部分相隔几步，但这并不意味着每一步都需要在其中存储特定活动的规范。

用上面的例子来解释:api.js 包含 getSomeStuff 的规范(需要 url 和两个名为 param1 和 param2 的参数)。它还保存了它导出的所有其他函数的规范。但是所有这些函数都有一个共同点:只是来回传递数据。

让我们更改这个模块，这样无论何时发生变化，我们都不需要更新它:

```
import axios from 'axios';
import config from '../config.js';export default {
    makeRequest: params => axios.post(config.baseUrl, params),
    ...
}
```

让我们看看我们做了什么:我们现在只为这个服务器导出一个函数，它的 url 在某个配置文件中。`params`是一个对象(与您将从容器组件中分派的对象相同)。

```
params === {type: actions.SOME_TYPE, param1, param2}
```

因此，如果服务器获得的对象与您从容器中分派的对象完全相同，这意味着您可以在服务器上使用相同的 actions.js 来识别动作并做出相应的反应。这样，容器需要按照服务器期望的方式命名分派的参数，但是当您重新构建时，感觉服务器只是前端的一部分，因为它对相同的动作做出反应。

这里的一个额外好处是，您可以使用同一个 api 模块来处理几个不同的 API(服务器通信),而不会有混乱的东西和导出。

如果你喜欢这个:D·:D·:D，别忘了鼓掌