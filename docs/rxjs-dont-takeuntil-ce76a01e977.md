# RxJS:不要等到

> 原文：<https://itnext.io/rxjs-dont-takeuntil-ce76a01e977?source=collection_archive---------3----------------------->

![](img/d6f9810d723d6964ed92ce5cab05ad61.png)

这是本·莱什的优秀文章 [RxJS:不要退订](https://medium.com/@benlesh/rxjs-dont-unsubscribe-6753ed4fda87)的后续。我想在这里取一个能反映他的朗朗上口的标题。我认为你 ***应该*** *在你有充分理由这样做的时候使用* `*takeUntil*` *，订阅管理就是一个充分的理由。*就像本说的

> 嗯…好吧，只是不要退订太多。

我会说，确保你也正确地使用了`takeUntil`来管理你的可观测量。与此相关的是，你可能想看看 Nicholas Jamieson 的文章 [RxJS:避免 takeUntil 泄露](https://blog.angularindepth.com/rxjs-avoiding-takeuntil-leaks-fb5182d047ef)。

本文将详细介绍我在一个 Ionic (Angular)项目中天真地使用`takeUntil`时遇到的一些问题。希望这能帮助你们中的一些人预先避免这些陷阱！这个应用是 Ionic v3 应用，所以我们仍然使用一些旧的生命周期挂钩。Ionic v4 建议在某些情况下使用 Angular 的内置挂钩。参见:[https://ionicframework.com/docs/angular/lifecycle](https://ionicframework.com/docs/angular/lifecycle)

## 带 takeUntil 的订阅管理

假设你有一个不止一个可观测的组件。你应该先问自己几个问题:

1.  我可以使用`| async`管道来管理组件中的可观察订阅吗？
2.  有没有什么合理的方法可以合并观察对象来管理单个订阅？

即使你对第二个问题回答“是”，为了保持一致，你可能还是要使用`takeUntil`。例如:

```
@Component()
export class ObservingPage {
  users = this.store.selectObservable('users');
  userList = [];
  unsubscriber = new Subject(); ionViewDidLoad() {
    this.users.pipe(takeUntil(unsubscriber)).subscribe(users =>
      this.userList = users,
    );
  } ionViewWillUnload() {
    this.unsubscriber.next();
    this.unsubscriber.complete();
  }
}
```

在这种情况下，使用带有`users`的异步管道是有利的，但是您可能有一些队友在他们不熟悉异步管道或者不想使用它时编写了代码。也许有一个更好的例子，您必须创建一个手动订阅。无论哪种方式，这里的要点是当视图卸载时，`users`可观察将完成，因为我们正在发射到`unsubscriber`并使用`takeUntil`。

我和一个大型团队一起开发的应用程序有许多由不正确的取消订阅 Observables 引起的内存泄漏问题。在没有任何判断的情况下，我们遵循了团队的建议，在卸载组件时使用`takeUntil`来修复这些内存泄漏。在很大程度上，这非常有效，是一个可行的解决方案。

## 管理具有不同生命周期的订阅

这个解决方案很好，但是 Ionic 会将页面保存在内存中，即使你离开它们。这导致一些订阅回调在页面的后台被调用，这可能是不必要的或不可取的。没关系……我们可以使用 WillEnter / WillLeave 生命周期挂钩来代替。

```
@Component()
export class OnlyWhenEnteredPage {
  users = this.store.selectObservable('users');
  userList = [];
  unsubscriber = new Subject(); ionViewWillEnter() {
    this.users.pipe(takeUntil(unsubscriber)).subscribe(users =>
      this.userList = users,
    )
  } ionViewWillLeave() {
    this.unsubscriber.next();
    this.unsubscriber.complete();
  }
}
```

希望，对你们中的一些人来说这看起来足够无辜，就像我一样。也许其他人会对这个显而易见的(回顾起来)问题感到惊讶:`**unsubscriber**` **是在页面加载时创建的，但是我们在页面离开**时完成它。这意味着每次我们重新进入页面时，任何使用`takeUntil(unsubscriber)`的操作都会立即完成，因为我们已经向它发出了请求。

如果你被一个类似的问题弄得措手不及，开始质疑你对可观察的创造和完成的理解，我完全了解你的感受，因为我经历了那件事。这里的解决方案相对简单:也在 enter 上创建您的退订主题管理器。

```
@Component()
export class OnlyWhenEnteredPage {
  users = this.store.selectObservable('users');
  userList = [];
  unsubscriber: Subject<void>(); ionViewWillEnter() {
    this.unsubscriber = new Subject();
    this.users.pipe(takeUntil(unsubscriber)).subscribe(users =>
      this.userList = users,
    );
  } ionViewWillLeave() {
    this.unsubscriber.next();
    this.unsubscriber.complete();
  }
}
```

## 更强大的方法？

出于某种原因，像这样使用`takeUntil`总是让我不愉快。我认为这可能与同时调用`.next`和`.complete`有关，(从技术上讲，你不必做`.complete`，但如果我没弄错的话，不这样做可能会导致内存泄漏)。

Ward Bell 的简单的 subsink 工具似乎是一个不错的方法。这实际上只是在你取消订阅接收器时，对你正在创建的订阅列表进行手动取消订阅，但是你不必担心管理和完成一个单独的*主题*，这是另一个需要处理的可观察主题。

```
import { SubSink } from 'subsink';@Component()
export class WhenEnteredAndLoadedPage {
  users = this.store.selectObservable('users');
  invitations = this.store.selectObservable('invitations')
  userList = [];
  invitationList = [];
  pageEnterSubs = new SubSink();
  pageLoadSubs = new SubSink(); ionViewDidLoad() {
    this.pageLoadSubs.sink = this.invitations.subscribe(inv =>
      this.invitationList = inv,
    );
  } ionViewWillEnter() {
    this.pageEnterSubs.sink = this.users.subscribe(users =>
      this.userList = users,
    );
  } ionViewWillLeave() {
    this.pageEnterSubs.unsubscribe();
  } ionViewWillUnload() {
    this.pageLoadSubs.unsubscribe();
  }
}
```

上面的每个例子对于每个生命周期挂钩只使用一个订阅，但是您可以使用任意多个。`subsink`似乎不是特别受欢迎，但在这种情况下，这似乎是管理我的多个订阅的好方法。如果你不想使用它，你可以以类似的方式使用`takeUntil`和两个 subject，只是你必须在`ionViewWillEnter`中创建一个 subject 而不是构造函数。

当然，异步管道总是引人注目。

**一位读者还向我指出，**这个功能也内置在 RxJS 订阅中，所以如果你不想使用`subsink`，你可以简单地使用`new Subscription`和`.add`来获得类似的功能。参见:[https://rxjs.dev/api/index/class/Subscription#add](https://rxjs.dev/api/index/class/Subscription#add)和[https://blog . angularindepth . com/rxjs-composing-subscriptions-b 53 ab 22 f1 FD 5](https://blog.angularindepth.com/rxjs-composing-subscriptions-b53ab22f1fd5)。

## 经验教训

RxJS 非常强大，但为此很容易出错。无论何时使用 RxJS 操作符，都要考虑自己在做什么，并询问这将如何影响流，以及流是否会适当地完成，这一点很重要。很容易犯一个错误，那就是从不完成一个可观察的，并造成内存泄漏。正如我所发现的，过早地完成可观测也很容易，当你期望它发射时，它不会发射。

当使用生命周期挂钩时，记住在相应的挂钩中镜像操作也很重要。您可能会想“这个主题是在 WillLeave 中完成的，所以需要在 WillEnter 中创建它。”