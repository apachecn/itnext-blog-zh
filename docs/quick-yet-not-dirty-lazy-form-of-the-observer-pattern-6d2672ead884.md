# 快速但不脏——观察者模式的懒惰形式

> 原文：<https://itnext.io/quick-yet-not-dirty-lazy-form-of-the-observer-pattern-6d2672ead884?source=collection_archive---------5----------------------->

## 让我们用几行代码引起一些反应并轻松地共享状态。

![](img/0ab6b437e4b4b55a05ffc6c4b8de1310.png)

[https://www.flaticon.com/free-icon/design_1792026?term = observer % 20 eye&page = 1&position = 13](https://www.flaticon.com/free-icon/design_1792026?term=observer%20eye&page=1&position=13)

声明:本帖不会教你“观察者模式”。

在这篇短文中，我将分享我的一个用例，以及我是如何在不引入外部工具的情况下解决这个问题的。

在我的一个项目中，(仿真回路)，每当状态改变时，**视图层**(几个按钮)必须更新**逻辑层**。

**逻辑层**会做出相应的反应。

这是一个 Typescript 项目，我没有使用任何框架，只有 DOM 和我。

在这种情况下，我看不出使用外部工具的任何意义——(顺便说一句，我喜欢使用 rxjs 主题，真棒 lib😎 👏 ).

[下面是代码](https://github.com/LironHazan/rocking-looper/blob/master/src/looper-view.ts#L32)！

## [观察者模式](https://en.wikipedia.org/wiki/Observer_pattern)

观察者模式是一种非常常见的软件设计模式。它的实现通常隐藏在第三方库中，所以您可能在过去多次使用过它，甚至没有意识到它..

由一个[专用类](https://github.com/gztchan/design-patterns-in-typescript/blob/master/observer/observer.ts#L29)生成一个可重用的通用“主题”的常见实现。

我觉得我甚至不需要它，我的项目只有 2 个组件，所以我只写了下面 14 行:

```
**static viewInteractionSubject**: ObserverType = {
  **observers**: [],

  on: (action: { **name**: **string**, **fn**: Function}) => {
    LooperViewLayer.viewInteractionSubject.**observers**.push(action);
  },

  dispatch: <T>(name: **string**, arg?: T) => {
    **if** (LooperViewLayer.viewInteractionSubject.**observers**.**length** === 0) **return**;
    **const** exec = LooperViewLayer.viewInteractionSubject.**observers** .find((event) => event.**name** === name);
    exec.fn(arg);
  }
};
```

我不会在大型项目中使用它(例如，在工作中)，它还没有完全扩展，我可以做一些改进——例如，添加**取消订阅**功能，使用**唯一标识符**像符号而不是“名称”字符串等..但是对于特定情况来说，这是可以的。

下面是使用 from wherever 实体响应视图的方法(希望只有一个)。

```
*// Subscribe to view interactions* LooperViewLayer.viewInteractionSubject
  .on({**name**: **'startRecording'**, fn: () => **this**.mediaRecorder && **this**.mediaRecorder.start()});

LooperViewLayer.viewInteractionSubject
  .on({**name**: **'stopRecording'**, fn: () => **this**.mediaRecorder && **this**.mediaRecorder.stop()});
```

我要感谢 [Uri Shaked](https://medium.com/u/355b1dfe86ae?source=post_page-----6d2672ead884--------------------------------) 认为这个话题值得一提，并为我审阅了这篇帖子:)

仅此而已，

干杯，勒荣。