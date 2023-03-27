# OnPush 变化检测用于更快的角度应用

> 原文：<https://itnext.io/onpush-change-detection-for-faster-angular-apps-f5d6dccea589?source=collection_archive---------3----------------------->

![](img/f15f14f792b167b435d2edb20754d29b.png)

虽然不是最快的，但默认情况下 Angular 是性能最好的框架之一。

即使大多数应用程序不需要进行任何高级优化就能正常运行，在旧的浏览器和较慢的设备上运行复杂的应用程序仍然是一项艰巨的任务。

# 变更检测策略🔥

我们可以做的第一个也可能是最重要的调整是改变 Angular 默认使用的检测策略，以最大限度地减少变化检测的运行次数，这将使你的应用程序运行得更流畅、更快。

默认情况下，你猜对了，Angular 使用策略`ChangeDetectionStrategy.Default`。这意味着组件将始终被检查。效率不高吧？

如果大多数组件不需要更新，为什么要这样做呢？输入`ChangeDetectionStrategy.OnPush`，它将指示变化检测跳过一个组件，除非发生以下任何情况:

*   元件的输入参考发生变化
*   组件中的 DOM 事件已被调度(例如点击)
*   通过异步管道订阅的可观察事件的发射
*   手动运行更改检测

这种实践对于大型复杂的应用程序更为重要，因为变更检测会跳过大量的组件。一个简单的方法可以看出这两种方法的区别，那就是使用 Chrome 的渲染开发工具。检查“油漆闪烁”选项，自己看看有多少次你的组件是不必要的重新渲染。

在下面的例子中，我们的组件将*而不是*更新视图:

```
@Component({
  ...,
  template: '{{ count }}',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ChangeDetectionComponent implements OnInit {
  count = 0; ngOnInit() {
    setInterval(() => ++this.count, 1000);
  }
}
```

# RxJS 来救援了

来自 Angular 1.x 的开发人员可能会发现这令人困惑并且难以使用:不可否认，在没有 RxJS 的情况下使用 OnPush 并不总是容易的。

也就是说，我认为使用 OnPush 提供了一种更好的编码实践方式。例如，通过推广 RxJS 和`async`管道的使用，我们得到了一个可预测的、声明性的代码库，它碰巧也是高性能的。

以下是使用`async`管道的一些优点:

*   自动订阅观察
*   当组件被破坏时自动退订
*   轻松与变革战略战略合作。OnPush
*   减少我们组件的锁定

简而言之，RxJS + OnPush =双赢。

让我们用一个`Observable`来重构前面的例子:

```
@Component({
  ...,
  template: '{{ count$ | async }}',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ChangeDetectionComponent implements OnInit {
  count$: Observable<number>; ngOnInit() {
    this.count$ = interval(1000)
        .pipe(
            map((count: number) => ++count)
        );
  }
}
```

我们现在有了一个优雅的、声明性的和执行性的解决方案！

# NGRX

当处理大型应用程序时，我建议使用状态管理库。不仅因为它有助于管理状态，还因为 Angular 状态管理库将可观察对象视为一等公民，就像框架一样。虽然有很多很棒的库，但我强烈推荐 NGRX。

NGRX 通过使用 RxJS 从存储中提取数据，轻松地处理纯角度组件，这意味着组件中保存的所有数据都是可观察的。

如果你还不知道 NGRX，那么你应该读一读。

为了阅读下面的例子，你只需要知道我们从商店(把它想象成我们的数据库)中检索数据作为一个可观察对象，并且我们通过`async`管道订阅在模板中显示它。

```
@Component({
    ...,
    changeDetection: ChangeDetectionStrategy.OnPush,
    template: `
        <div *ngFor="let todo of (todos$ | async)">
            {{ todo.name }}
        </div>
    `
})
export class TodosComponent {
    constructor(private store: Store<AppState>) {} ngOnInit() {
        this.todos$ = this.store.select((state) => state.todos);
    }
}
```

每当`todos$`发出一个新值时，框架就会呈现模板。

# 什么时候使用 OnPush 没有意义？

绝不！OnPush 是一种让你的应用程序更快的简单方法，个人认为没有理由不每次都使用它。

# 重构代码库以提高性能🚀

我使用的大多数遗留角度代码库都使用默认的变化检测，应用程序的性能受此影响很大。大多数开发人员也不热衷于使用它，只是因为它看起来令人生畏。但是，其实也不一定。

如果您计划通过使用 OnPush 变更检测来重构代码库，首先要知道的是，您永远不要从父组件开始。原因是，当 changeDetection 添加到父组件时，其所有组件树都会受到影响。

我的建议是从叶子开始，一路向上到父组件。哑组件，如果写得好，通常不应该受到影响，因为它们只是接收输入并呈现它，所以它们是你应该重构的第一个对象。一旦容器的所有树都被重构，就该是容器的时候了。

集装箱负责什么？

*   检索数据并将其传递给其他组件
*   将一个或多个组件的布局放在一起

管理数据可以说是前端开发人员今天面临的最困难的任务，这就是为什么设计良好的容器是项目整体架构的关键。

我推荐两种选择:

*   如果您不想使用第三方，可以使用自己的 RxJS 状态管理，方法是在服务中使用主题，并通过 Observables 公开数据
*   用 NGRX，NGXS，Akita 等。？

# 外卖食品

*   使用`OnPush`变化检测策略，你的应用会更快
*   使用`async`管，这将使`OnPush`更容易工作
*   使用状态管理库，或者在服务中利用 RxJS
*   重构很难:从你的叶子组件开始，一路向上，直到所有组件都使用`OnPush`

本文最初写于[https://frontend . consulting/on-push-change-detection-for-faster-angular-apps](https://frontend.consulting/on-push-change-detection-for-faster-angular-apps)