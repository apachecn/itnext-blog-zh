# Iniz:使反应实际上是反应性的

> 原文：<https://itnext.io/iniz-makes-reactjs-actually-reactive-a25fd873370a?source=collection_archive---------1----------------------->

![](img/e5fb2aec09613779d7d20d76d3cab2a2.png)

[环球媒体](https://unsplash.com/@orbtalmedia?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

[Iniz](https://iniz.netlify.app) 是一个为 ReactJS 带来反应性的状态库。

# 动机

ReactJS 是一个视图库，它使用“推”策略来检测变化并应用于实际的 DOM。

为了让开发人员通过组件“推送”状态，引入了不同的模式，如 HOC、render-props、Context。这些自顶向下的方法都有不同的缺点，比如额外的重新渲染和“供应商地狱”。

Iniz 采取了不同的方法，在 FP 中将 ReactJS 的“组件”视为“效果”，将变量 update 视为“异常”。

当 ReactJS“组件”第一次呈现时，Iniz 自动收集其依赖关系，并在依赖关系更新时触发重新呈现。

# 基元

Iniz 提供了几个描述状态的原语:“atom `/“state”、“effect”和“computed”。

## 原子

若要创建状态，请调用“atom()”,并将第一个参数作为初始值。

如果使用 TypeScript，Iniz 已经将初始值推断为 type。您可以选择将类型作为泛型传入。

然后，您可以调用一个“原子”作为函数来获取它的值。要更新它，只需在函数调用中传递值。

## 影响

在声明了“原子”之后，可以用“效果”来跟踪它的值。

` effect()`接受函数作为参数并立即执行一次。

在执行过程中，它收集所有使用的“原子”,并在“原子”更新时再次触发。

## 计算

你也可以结合原子成为一个“计算”值。

“computed”也接受函数。但与“效果”不同，它返回一个带有计算值的“原子”。

# 反应

要在 ReactJS 中使用 Iniz 原语，只需将“ [@iniz/core](http://twitter.com/iniz/core) 替换为“ [@iniz/react](http://twitter.com/iniz/react) ”,并将“effect”替换为一个组件。

来源代码可从 [inizio/iniz](https://github.com/IniZio/iniz) 获得