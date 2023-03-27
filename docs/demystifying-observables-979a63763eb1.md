# 揭开可观察事物的神秘面纱

> 原文：<https://itnext.io/demystifying-observables-979a63763eb1?source=collection_archive---------0----------------------->

## 实用的方法

![](img/94eeeec4ecb5bbb4af763b00c57a8fd2.png)

反应式编程是当今最热门的话题之一。随着像 RxJs 这样的库和 Angular 这样的大框架采用它，它不断地继续增长。

根据我的经验，很难向没有太多函数式编程经验的人解释可观测量。在大多数情况下，这需要一点思想转变。

可观测量伴随着许多操作符和概念。人们经常因为反应式编程而过度紧张，因为他们从来没有完全理解可观测量是如何工作的。

但是好消息是。可观测量的主要概念并不像乍看上去那么复杂。事实上，它们与纯 JavaScript 函数有很多共同之处。什么？在我的博客上找到更多信息:

[](https://medium.com/@kevinkreuzer/observables-just-powerful-functions-a033c355b22c) [## 可观察的，只是强大的功能？

### 反应式编程是当今最热门的话题之一。有了 RxJs 这样的库和 React 或…

medium.com](https://medium.com/@kevinkreuzer/observables-just-powerful-functions-a033c355b22c) 

如标题所示，这篇博客试图揭开可观察事物的神秘面纱。在我看来，只有一种方法可以做到这一点；通过编写我们自己的自定义可观察对象。😎

写出我们自己的可观测数据听起来很复杂，但这不是火箭科学。事实上，你会惊讶地发现实现自己的基本可观察对象是多么容易。

# 我的可观察、冷漠和懒惰

大多数，但不是所有的可观测量都是懒惰的！也许你已经听说过冷和热这两个术语。

为了简单起见，让我们把重点放在冷的主要部分——懒惰。他们不会做任何事，直到你告诉他们这样做。但是你如何告诉他们去做一些事情呢？

让我们先来看看 Observables 构造函数。我们要用这个函数来创建一个发出 1，2，3 然后完成的可观察对象。

```
const example$ = new Rx.Observable(function(observer){
  observer.next(1)
  observer.next(2)
  observer.next(3)
  observer.complete()
})example$.subscribe(next => console.log(next), 
                   err=>  console.error(err), 
                   () => console.log('Done'))// Output
// 1
// 2
// 3
// Done
```

我们调用构造函数并传入一个回调。猜猜这个回调叫什么？对，订阅！

> [*在 Twitter*](https://twitter.com/KevinKreuzer90) *或 medium 上关注我，了解最新的博客帖子和有趣的前端内容！🐤*

事实上，这就是我们之后要调用五行的同一个 subscribe。为了更清楚地理解，让我们按以下方式重构上面的代码片段:

```
const subscribe = function(observer){
  observer.next(1)
  observer.next(2)
  observer.next(3)
  observer.complete()
}const observer = {
  next: next => console.log(next),
  error: err => console.error(err),
  complete: () => console.log('Done')
}const example$ = new Rx.Observable(subscribe)
example$.subscribe(observer)
```

重要的是要理解，我们在开始时传递的 subscribe 函数与我们在订阅流时将要调用的函数是相同的。但是这是如何工作的呢？

如果你读过我的另一篇关于“可观测量，只是强大的功能”的博文，你可能已经有了主意。

让我们继续实现一个非常基本的可观察对象。同样，你会对它的代码之少感到震惊。

```
class MyObservable {

    constructor(subscribe) {
        this._subscribe **=** subscribe
    }

    subscribe(observer) {
        return this._subscribe(observer);
    }
}
```

MyObservable 的构造函数接受 subscribe 回调，并将其分配给 internal _subscribe 字段。

通过调用 public subscribe，我们可以用传入的 Observer 在内部调用 _subscribe。通过这个函数调用，我们开始我们的可观察对象，告诉他用 1，2，3 调用我们的观察者的下一个，然后完成。

太好了！！现在，我们可以轻松地从顶部获取我们的代码片段，并交换 Rx。用我们自己的习惯来观察。

但是等等。还记得我们如何传递三个函数而不是一个观察者对象吗？在我们目前的实现中，这还不可能。

# 接受回调和观察者对象🤝

为了接受回调和观察器，我们将实现一些小的转换逻辑。

如果用户传入三个函数，而不是一个回调函数，我们将把这些回调函数转换成一个 Observer 对象，然后传递给我们的内部 subscribe 函数。

```
subscribe(next, error, complete) {
    let observer;
    if (typeof next **===** 'function') {
        observer **=** {
            next,
            error**:** error **||** function () {},
            complete**:** complete **||** function () {}
        }
    } else {
        observer **=** next;
    }
    return this._subscribe(observer);
}
```

将回调转换成可观察的对象非常简单。

> 如果第一个传入的值是一个函数，我们将在内部创建自己的观察器。如果用户没有传入一个错误或完整的函数，我们就给我们的可观察对象分配一个空函数。
> 
> 如果消费者决定传入一个观察者对象，我们不做任何转换，直接使用观察者对象。

厉害！我们已经创建了一个非常基本的实现。🤠

可观察的事物需要思维的转变，但它们不是纯粹的魔法。

# 🧞‍ 🙏顺便说一下，点击👏🏻拍手声👏🏻按钮在左边(高达 50 倍)，如果你喜欢这篇文章。

# 掌声帮助其他人找到它，并鼓励我写更多的帖子

想知道如何测试你的可观测链？然后查看我的文章“关于 RxJs marble 测试和 TestScheduler 的发现”。

[](/findings-about-rxjs-marble-testing-and-the-testscheduler-b23c6bdf6b49) [## 关于 RxJS marble 测试和 TestScheduler 的发现

### 最近，我编写了一个定制的 Rx 操作符，用于重试失败的 http 请求。RxJS 允许我们处理这种异步的方式…

itnext.io](/findings-about-rxjs-marble-testing-and-the-testscheduler-b23c6bdf6b49)