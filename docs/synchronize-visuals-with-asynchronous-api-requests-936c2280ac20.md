# 用异步 API 请求同步视觉效果

> 原文：<https://itnext.io/synchronize-visuals-with-asynchronous-api-requests-936c2280ac20?source=collection_archive---------2----------------------->

## 在最短时间内显示 API 加载指示器

![](img/1bd6c64beb6bcb88222b205d5f13bdfe.png)

照片由[乔纳森派](https://unsplash.com/@r3dmax?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

有时你的 api 请求完成的非常快，如果你有一个装载指示器，你会在屏幕上看到一个快速的信号。这是一个在最短时间内显示装载指示器的好方法。

下面是一个简单的 [axios](https://github.com/axios/axios) 请求:

普通 Ajax 请求

现在，让我们以上面的例子为例，如果 API 响应完成得比预期的要快，就把它改为最小延迟。注意`Promise.all`的使用，这将在返回 API 响应之前等待两个任务都完成。

延迟的 Ajax 请求

如果`axios.get`请求的完成时间少于 500 毫秒，那么额外的`delay(500)`函数将确保足够的延迟来加载动画以正确显示。使用这种策略，我们可以保证在 api 请求解决之前有一定的等待时间。

在这个例子中，我使用了 [axios](https://github.com/axios/axios) ，但是你也可以通过[获取 API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) 或任何其他技术来使用它。

查看我关于如何管理加载和错误状态的相关文章:

[](/react-redux-api-loading-errors-e783972c5424) [## 反应/减少 API 加载和错误

### 管理 API 请求状态并妥善处理错误

itnext.io](/react-redux-api-loading-errors-e783972c5424) 

如果你喜欢这篇文章，请分享它，关注我，阅读我的其他[文章](https://medium.com/@robertsavian)和/或使用下面我的推荐链接注册 Medium。谢谢！

[](https://medium.com/@robertsavian/membership) [## 通过我的推荐链接加入 Medium—Robert S(代码带)

### 作为一个媒体会员，你的会员费的一部分会给你阅读的作家，你可以完全接触到每一个故事…

medium.com](https://medium.com/@robertsavian/membership)