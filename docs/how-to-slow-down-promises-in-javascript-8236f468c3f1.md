# 如何减缓 javascript 中的承诺

> 原文：<https://itnext.io/how-to-slow-down-promises-in-javascript-8236f468c3f1?source=collection_archive---------0----------------------->

![](img/f47a568c2d10e3a9e4be945bc135263e.png)

如果您必须遍历数组并对数组中的每个对象发出 web 请求，您可以这样做:

问题是，如果您的阵列有许多成员，所有这些请求将同时到达服务器。所以，我们需要找到一些方法，不要同时运行所有的东西。

另外一个问题是，承诺从产生的那一刻起就开始执行。

是时候开始寻找解决方案了:

首先出现是 [es6-promise-pool](https://www.npmjs.com/package/es6-promise-pool) ，但是这个库看起来很落后，而且在示例中太复杂了(我需要编写 promise producer)

他们还推荐了几个其他的库:

*   [承诺池](https://github.com/vilic/promise-pool) —老得要命，自 2015 年以来没有更新
*   Bluebird's Promise.map() —我使用本地承诺已经有一段时间了
*   [contra 的 concurrent()函数](https://github.com/bevacqua/contra#%CE%BBconcurrenttasks-cap-done) —两年前的最后一次提交，不支持承诺，而是使用回调。
*   [Q](https://github.com/kriskowal/q)with[Q limit](https://github.com/suprememoocow/qlimit)—Q limit 实际上是 Q 的插件，现在我需要两个可互操作的库，而且插件 4 年没更新了，那就很多了。
*   async with[queue](https://github.com/caolan/async#queueworker-concurrency)()function——这个维护良好的库，但是 queue function 的例子是用回调完成的，这已经过时了。

# 救援的极限

[p-limit](https://github.com/sindresorhus/p-limit) 是一个非常简单的库，使用起来非常简单:

现在，我们的请求按照我们想要的速度执行，因为我们可以将并发性设置为我们想要的任何数量。我增加了进度条作为奖励。

作者不仅创建了 p-limit，还创建了一整套[有用的 promise helper 函数](https://github.com/sindresorhus/promise-fun)。