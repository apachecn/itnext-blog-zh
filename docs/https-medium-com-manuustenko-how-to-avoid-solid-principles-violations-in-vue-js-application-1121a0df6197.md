# 如何避免在 Vue 中违反坚实的原则？JS 应用程序

> 原文：<https://itnext.io/https-medium-com-manuustenko-how-to-avoid-solid-principles-violations-in-vue-js-application-1121a0df6197?source=collection_archive---------0----------------------->

几天前，我的朋友 Philipp 决定创建一个新的业务(一个制作美味饼干的面包店),并请我帮助他创建一个 todo list 应用程序，他可以在其中写下自己的任务。他决定用 Vue。JS 作为一个前端框架，但真的不知道如何在未来保持他的应用程序易于扩展和维护。他听到一些关于固体的神话(固体是什么意思？？？)原则和清晰的架构，但不知道如何在 Vue 中使用它们。JS app。

在这篇文章中，我想讨论如何在我们的 Vue 中避免违反坚实的原则。JS 项目。

什么是固体？SOLID 是由 Michael Feathers 创造的首字母缩写词，由美国软件工程师 Robert c .“*Bob 叔叔*”Martin 在他的书“*Design Principles and Design Patterns”中推广。这些原则是面向对象编程范例中非常重要的部分，旨在使我们的程序在下一次开发中更加灵活、可读和可维护。坚实的原则包括下一个概念:*

*   单一责任原则
*   开闭原理
*   利斯科夫替代原理
*   界面分离原理
*   从属倒置原则

让我们看看 real Vue 中的所有这些原则。JS 项目以及我们如何避免违反原则。我们将创建一个简单的待办事项列表应用程序。

## **先决条件**

让我们创建一个新的 Vue。使用 vue cli 的 JS 应用程序([https://cli.vuejs.org/](https://cli.vuejs.org/))。

`vue create todo-app`

在我们的应用程序中，我将使用 vue 2.6.10 + typescript 3.4.3，所以如果你还不熟悉 typescript，你可以在这里找到文档([https://www.typescriptlang.org/docs/home.html](https://www.typescriptlang.org/docs/home.html))。

安装后，我们必须清理一下，并删除所有演示组件。我们的`src`目录结构如下所示

```
src
----views
-------Home.vue
----App.vue
----main.ts
----shims-tsx.d.ts
----shims-vue.d.ts
----types.ts
```

我们准备好出发了…

## **单一责任原则**

假设我们修改了`views/Home.vue`组件来获取任务列表并显示给用户。它可能看起来像这样:

基本上，我们在一个`views/Home.vue`组件中创建了整个应用程序。这里我们可以看到 SRP 的违反，它告诉我们:“ ***每个组件应该只有一个理由来改变*** ”。但是，我们有多少理由去改变这个成分呢？

1.我们希望更改 fetchTodos()方法来获取 Todos。这可能是任何改变的原因:获取到 axios 或任何其他库、api、方法本身等等。

2.我们想添加更多的元素到这个组件:侧栏，菜单，页脚等。

3.我们想改变现有的元素:标题或待办事项列表。

我们发现了至少三个我们想要改变`views/Home.vue`的理由。真正的问题开始于应用程序何时会增长和变化。它变得太大，我们不再记得我们的代码，失去了我们的控制。我们可以通过将每个原因提取到它自己的组件、类或函数来避免 SRP 违规。让我们做一些重构。

首先，我们可以通过创建一个新的 Api 类来提取 fetching todos 方法，并将其放到`api.ts`文件中。

接下来，让我们将标题提取到一个新的功能组件`components/Header.vue`

最后一步是提取待办事项列表。

我们在`views/Home.vue`中的代码现在看起来更加清晰易读。

## **开闭原理(OCP)**

让我们更仔细地看看我们的`components/TodoList.vue`。我们可以看到，该组件获取待办事项列表，并创建一堆卡片来表示我们的任务。然而，如果我们想改变这些卡片，甚至将我们的任务显示为表格，而不是卡片，会发生什么呢？我们必须改变(修改)我们的组件。现在看起来有点呆板。OCP 告诉我们:“ ***组件对于扩展应该是打开的，对于修改*** 应该是关闭的”。让我们解决这个违规问题。

我们可以使用 vue 插槽使我们的`components/TodoList.vue`更加灵活。

并将我们的卡片移动到一个单独的组件`components/TodoCard.vue`

并更新`views/Home.vue`

现在，我们可以轻松地用另一个组件替换我们的任务可视化。

**利斯科夫替代原理(LSP)**

让我们来研究我们的 Api 类。

首先，我们将 Api 类重命名并重构为 BaseApi 类，并将其移动到一个单独的目录中`api/BaseApi.ts`

正如你所看到的，BaseApi 类有一个 fetch()方法，它接受一个参数“url”。

出于某些原因，我们决定在应用程序中添加 axios 库。

```
npm install --save axios
```

并创建一个新类 AxiosApi，它是`api/AxiosApi.ts`中 BaseApi 的子类

如果我们在`views/Home.vue`的 fetchTodos()方法中用一个新的 AxiosApi(子类)替换我们的 BaseApi(父类)

```
import { AxiosApi } from '@/api/AxiosApi'*...
async* fetchTodos(): *Promise*<*ITodo*[]> {
  *const* api = *new* AxiosApi()
  *return await* api.fetch('todos')
}
```

它会破坏我们的应用程序，因为我们没有遵循 LSP:“***当扩展一个类时，记住你应该能够传递子类的对象来代替父类的对象，而不会破坏客户端代码*** ”。

正如您所注意到的，我们将 object 作为参数传递给 AxiosApi 类的 fetch()方法，但是 BaseApi 类接受 string。在这种情况下，我们不能毫无痛苦地用父类替换子类。

让我们迅速修理它。

现在我们可以两者都用，还有 BaseApi 和 AxiosApi。我们甚至可以更深入，通过在`api/api.ts`中创建 Api 类来改进我们的代码，该类扩展了基类并具有私有属性提供者

现在`views/Home.vue`中的方法 fetchTodos()不需要知道我们正在使用哪个库，我们可以很容易地在 Api 类中切换提供者。

```
*import* { Api } *from* '@/api/api'...*async* fetchTodos(): *Promise*<I*Todo*[]> {
  *const* api = *new* Api()
  *return await* api.fetch('todos')
}
```

## **接口隔离原理(ISP)**

我们现在把我们的任务想象成卡片。让我们也在`components/TodoRow.vue`中添加简单的任务列表

并用列表可视化代替卡片可视化

正如您所注意到的，我们在 TodoCard.vue 和 TodoRow.vue 中将整个 todo 对象作为 prop 发送，但是我们只使用了该对象的一部分。我们没有在两个组件中都使用`userId`属性，也没有在 TodoCard.vue 中使用`id`属性。我们违反了 ISP 规定:“**组件不应该被强制依赖于它们不使用的**属性和方法”。

有很多方法可以解决这个问题:

*   将 Todo 接口分割成几个较小的接口
*   仅将使用过的属性传递给组件

让我们重构代码，使用功能性(无状态)组件。

现在看起来好多了。

## **依存倒置原则(DIP)**

DIP 告诉:“**高级类(组件)不应该依赖低级类(组件)。两者都应该依赖于抽象。抽象不应该依赖于细节。细节应该依赖于抽象。**

什么叫高等级和低等级。

*   低级类实现基本操作，如使用 API。
*   高级类包含复杂的业务逻辑，指导低级类做一些事情。

让我们回到我们的 Api 类，在`types.ts`中为我们的 Api 类创建一个新的接口。

```
*export interface IApi* {
  fetch(url: *string*): *Promise*<*any*>
}
```

并更新我们在`api`目录和`views/Home.vue`中的所有 api 类。

现在我们的低级(api 类)和高级(views/Home.vue)类依赖于一个接口。原来依赖的方向已经颠倒了:低级 api 类现在依赖于高级抽象。

## **结论**

在这篇文章中，我们在 small Vue 中讨论了所有坚实的原理。JS 项目。我希望它能帮助您避免项目中的一些架构错误，并提高您对坚实原则的理解。

代码可以在这里找到[https://github.com/NovoManu/SOLID-vue](https://github.com/NovoManu/SOLID-vue)

在推特上关注我[https://twitter.com/ManuSEngineer](https://twitter.com/ManuSEngineer)

祝你好运！