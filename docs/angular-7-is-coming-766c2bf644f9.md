# Angular 7 来了

> 原文：<https://itnext.io/angular-7-is-coming-766c2bf644f9?source=collection_archive---------0----------------------->

![](img/49a1b0cb7aa41f7d25afb01580128b78.png)

弗兰克·麦肯纳拍摄的照片

今天 Angular 团队发布了 Angular 7 beta 版。我们中的许多人会对我们在这个测试版中看到的功能和错误修复感到好奇。

# 特征

先说新特性。

## 角度兼容性编译器(ngcc)

这个编译器会把用`ngc`编译的`node_modules`转换成看起来是用`ngtsc`编译的`node_modules`。这种转换将允许 **Ivy 渲染引擎**使用这样的**遗留**包。

## 多靴带

新的生命周期钩子接口`ngDoBootstrap`。

**例如:**

```
class AppModule implements DoBootstrap {
  ngDoBootstrap(appRef: ApplicationRef) {
    appRef.bootstrap(AppComponent);
  }
}
```

## 编译程序

更新了 XMB 占位符(`<ph>`)，以在示例顶部包含原始值。现在占位符根据定义可以有一个示例(`<ex>`)标签和一个文本节点。TC 使用文本节点作为占位符的“原始”值，而示例应该表示一个空值。让我们看看下面的例子:

**旧行为:**

类似于`<foo>Band: {{yourName}}</foo>`的消息会生成这个 **xmb** 消息:

`<msg id=123>Name: <ph name=”INTERPOLATION”><ex>{{yourName}}</ex></ph></msg>`

**新行为:**

类似于`<foo>Name: {{yourName}}</foo>`的消息会生成这个 **xmb** 消息:

`<msg id=123>Name: <ph name=”INTERPOLATION”><ex>{{yourName}}</ex>{{yourName}}</ph></msg>`

TC 使用`<ph>` PCDATA(文本节点)进行一些验证。

# 错误修复

## 巴泽尔

现在`compile_strategy()`用于决定是使用`ngc`(传统)还是`ngtsc`(本地)构建角度代码。为了让 g3 构建规则正确切换并允许在 g3 中测试 Ivy，`compile_strategy()`变成了`importable`。

## 常春藤

动态创建的视图的模板函数不再相互嵌套。不是嵌套函数和使用闭包来获得父上下文，而是通过一条指令显式地重新定义父上下文。这意味着我们不再为嵌套在其他循环中的循环创建多个函数实例。

示例:

代码示例由[卡拉](https://github.com/kara)

之前:

由[卡拉](https://github.com/kara)编写的代码示例

之后:

代码示例由[卡拉](https://github.com/kara)

`TView.components`不再跟踪指令索引(只跟踪宿主元素索引)。我们可以将组件数组切成两半，因为组件的上下文现在存储在组件的`LViewData`实例中，可以从那里访问。

现在我们有了新的指令，`reference()` ( `r()`)。以前，我们使用闭包来访问父视图中的本地引用。既然嵌套模板函数是平面的；他们不能通过闭包访问本地引用，所以我们需要另一种方法来遍历视图树。`reference`取一个嵌套层次，然后局部引用的索引遍历视图树，找到加载局部引用的正确视图。

之前:

由[卡拉](https://github.com/kara)编写的代码示例

之后:

由[卡拉](https://github.com/kara)编写的代码示例

## 核心

如果属性没有初始化，Angular 现在对`@Output`有了更好的错误处理。

# 参考

[](https://github.com/angular/angular) [## 角度/角度

### 角-一框架。移动和桌面。

github.com](https://github.com/angular/angular) 

# 其他角形物品

[](https://hackernoon.com/dipi-your-angular-development-assistant-3e8ed7b4d355) [## Dipi:你的棱角发育助手

### Dipi 是一个简单的 Angular 库，它包含了一个巨大的、对 Angular 开发者有用的指令和管道包。这个…

hackernoon.com](https://hackernoon.com/dipi-your-angular-development-assistant-3e8ed7b4d355) [](https://medium.com/@vyakymenko/increasing-rendering-performance-in-angular-with-lazy-render-ngfor-ae8c5d16e194) [## 使用 Lazy Render *ngFor 提高 Angular 中的渲染性能

### 我认为许多 Angular 开发人员在呈现大型数据列表时存在问题。所以我想和你分享一些…

medium.com](https://medium.com/@vyakymenko/increasing-rendering-performance-in-angular-with-lazy-render-ngfor-ae8c5d16e194)