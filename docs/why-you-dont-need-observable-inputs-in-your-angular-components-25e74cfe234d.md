# 为什么在角度分量中不需要可观测的输入

> 原文：<https://itnext.io/why-you-dont-need-observable-inputs-in-your-angular-components-25e74cfe234d?source=collection_archive---------1----------------------->

![](img/b7d6957110b783c678bb4055212f2d87.png)

“一对塞特”，亚历山大·波普，1913 年

我曾经试验并构建了几个组件，所有的组件都可以被观察到。为此，我必须使用 setters 并订阅收到的 observables。如果 Angular 支持“可观察的输入”,我就不用这么做了，但是我发现了为什么你不应该把可观察的输入传递给`@Inputs`,即使 Angular 支持它。

# 避免可观察输入的原因

## ⊖降低了可重用性

具有可观察输入的组件或指令的每个“消费者”都应该以可观察值的形式提供值——但通常情况并非如此，值只是原语或对象，因此您必须专门为该组件创建可观察值。有些情况下，您将组件嵌入到您无法修改的组件中。

## ⊖不可预测通知

当您的组件订阅 Observable 时，它可能会有一些缓冲值，但这并不能保证——而且它可能会因冻结的 UI 部件或永无休止的旋转而产生令人讨厌的错误。

如果您需要读取/合并来自多个输入的值，这种情况尤其容易遇到——只要一个没有缓冲值的可观察值就可能冻结整个链。

在框架层面上，Angular 可以将`startWith(undefined)`添加到操作中，但是要做到这一点，您的代码不应该有类似`if (value==null) return;`的东西。此外，您的代码应该准备好处理多个缓冲值。

## ⊖多重订阅可能会产生问题

很简单的例子:

```
getUsers() {
  return this.http.get('/users');
}

const users$ = getUsers();

getCoordinates(element) {
  return someHeavyDOMCalculations(element);
}

const coordinates$ = getCoordinates(this.element$);
```

但是如果您使用`users$`或`coordinates$`作为可观察输入的值，那么每次使用都会创建一个新的订阅(并且在每个组件中创建多个订阅非常容易)。如果应用了`ngFor`或`ngIf`,事情会迅速增加，性能下降会变得很明显。

在`users$`的情况下，很容易发现问题，像`getUsers()`这样的大多数方法会将`shareReplay()`添加到观察链中。但这只是一个简化的例子——在许多现实生活中，很容易找到一个例子，其中 observable 没有针对多个订阅者进行优化。

这不仅仅是性能问题——一些可观察到的问题可能会产生副作用，并修改数据——在这种情况下，损坏的数据比 XHR 请求或布局回流的瀑布更危险。

## ⊖可观测量会产生误差

可观察输入期待一个理想的场景——第一个值被缓冲，只有一个缓冲值，可观察输入准备好被多播，不会意外发送“完成”通知，并且永远不会发出错误。

您必须为每个可观察到的输入编写一些样板代码来处理所有的边缘情况，其中一些是框架无法为您处理的。

## ⊖输入可以被替换

与任何其他输入一样，可观察输入可以在运行时被替换——可能会创建一个新的可观察输入，并且您的代码必须处理重新订阅和实现正确更新的所有影响。如果你使用类似于`combineLatest()`的东西来处理多个不同的输入，事情可能会变得复杂。

# 解决办法

有一个健壮的、经过战斗考验的简单解决方案: **setters + store** 。

在一些简单的情况下，您可以只使用 [BehaviorSubject](https://rxjs.dev/api/index/class/BehaviorSubject) 而不是某个状态管理库中的存储，但前提是您不需要同时使用多个输入。

简化示例:

```
@Component({
  ...
  providers: [ExampleComponentStore]
})
export class ExampleComponent {
    protected readonly store = inject(ExampleComponentStore);

    @Input() set id(id: number) {
        this.store.patchState({id});
    }
}
```

就是这样！

即使多个输入将被更改，您也可以使用您喜欢的存储功能来正确处理它。

如何将存储添加到组件取决于您(以及您喜欢的库)。这个例子的要点是:只需使用 setter 修改状态。

Setters 提供了一个额外的好处:可观察值可以不止一次地产生值——如果可观察输入被 Angular 本身转换为原始值，开发人员将只读取第一个值，而连续的值将会丢失(没有那么多组件观察 ngOnChanges)。使用 setters，您的组件将被迫在每次输入改变时更新其状态。

Chau Tran 有一个相同技术的例子:

使用这种技术，您的组件将保持可重用，将对输入变化做出反应，并将避免所有可观察到的输入的缺陷。