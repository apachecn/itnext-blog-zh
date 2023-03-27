# 用这个完整的指南准备一次 VueJS 面试

> 原文：<https://itnext.io/prepare-a-vuejs-interview-with-this-complete-guideline-c90d072482a5?source=collection_archive---------4----------------------->

## 前端面试

## 它包括关于 Javascript、Vue、Vuex、Typescript、HTML/CSS 和 Web 开发文化的问题

无论你是面试官还是被面试者，这里有一些信息将有助于展示你的技能或确定候选人的能力。

以下是我们将要讨论的要点:

*   Javascript:一切的基础
*   **Vue** :基础和高级主题
*   Vuex:Vue 商店怎么样
*   **类型脚本**:让我们在 Vue 应用中利用 JS 和用法
*   **HTML/CSS** :常识
*   **网络开发文化**

![](img/4696dccd3ae9adaabdf814291dadd462.png)

布兰科·斯坦切维奇在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

# Javascript

> ***【JS】关键词*** `***const***` ***和*** `***let***` ***有什么区别？***

一个很容易开始的问题，你会在这里找到答案(也包括`var`关键词)。

**TL；DR :** `let`允许你重新定义一个变量，而`const`在声明时被初始化，不能被重新定义。

> ***【JS】如何去除数组中的重复项？***

我真的很喜欢这个，因为它在现实世界中有很多应用。此外，它提供了几个答案来确定候选人关于 Javascript 的知识。具有阵列功能的 ES6 版本是:

使用`Set`结构，我们可以缩短获得独特物品的时间:

> ***【JS】给出 3 种声明函数的方法。***

```
function firstVersion() { ... }
const fn = function() { ... }
(arrow, function) => { ... }
```

> ***【JS】执行这段代码，给出输出。***

这是一个非常有趣的练习，可以证明您的范围界定技能。

答案是`20`和`NaN`。

这个练习摘自 Lydia Hallie 的[高级 Javascript 问题列表](https://github.com/lydiahallie/javascript-questions)。你可以用额外的问题来写作。

# Vue [V]

> ***【V】关于 Vue 模板的主要规则是什么？***

Vue `<template>`必须有 1️⃣，并且只能有 1️⃣根元素。

> ***【V】一个*** `***computed***` ***和一个*** `***method***` ***有什么区别？***

`computed`是一种基于其反应依赖性缓存在内存中的方法。

关于`methods`，每当重新渲染发生时，方法调用**将总是**运行该函数。

> ***【V】什么是*** `***mixin***` ***？什么是*** `***filter***` ***(举个例子)？***

一个`mixin`帮助在 Vue 组件之间共享属性、功能或任何组件选项。这些选项将与“调用”它的组件的选项合并。这类似于 OOP 中的继承。

一个`filter`转换一个输入参数来修改它的呈现方式。例如，它可以是隐藏 IBAN 或信用卡号码的掩码。

> ***【V】***`***v-if/v-else***`***和*** `***v-show/v-hide***` ***有什么区别？***

`v-if/v-else`从 DOM 中添加或删除一个元素或元素块，而`v-show/v-hide`只是从一个元素或元素块中转移`display` CSS 规则。

> ***【v】解释你所知道的管理父母↔️孩子沟通的不同方法？***

`**props**`使用`props`的➡️允许从父组件向其子组件提供一个值:

`**Custom events**` ➡️自定义事件帮助子节点向其父节点发送数据，使用的事件*可能由用户*触发。

➡️我们可以添加一个组件的引用并访问它的属性。

⚠️不推荐这种方法，因为它不遵循 Vue 组件的生命周期。另外，这是一扇通向副作用的大门。

`**Vuex store**` ➡️网络应用规模解决方案:使用商店。Vuex 是 Vue app 的 app 状态管理解决方案。

[](https://vuex.vuejs.org/) [## Vuex 是什么？Vuex

### Vuex 是 Vue.js 应用程序的状态管理模式+库。它作为一个集中的商店为所有的…

vuex.vuejs.org](https://vuex.vuejs.org/) 

*可选地，* [***总线事件***](https://www.digitalocean.com/community/tutorials/vuejs-global-event-bus) *可能已经被提及，即使它用于兄弟通信。但这是一个很好的点来展示一个可选的解决方案。*

> ***【V】如何从后端获取数据？***

有几种方法可以从后端检索数据。使用一个 REST API，我们可以谈论 [Vue 资源](https://www.npmjs.com/package/vue-resource)、 [**Axios**](https://github.com/axios/axios) 当然还有`fetch` API 甚至 GraphQL(使用 [vue-apollo](https://apollo.vuejs.org/) )。

# Vuex [Vx

> ***【Vx】Vuex 是什么？它的优点是什么？***

Vuex 是一个 **app 状态管理**工具，它实现了[状态管理模式](https://vuex.vuejs.org/#what-is-a-state-management-pattern)。

它有助于在一个地方集中和修改数据。因此，有**一个真理源**被管理。

> ***【Vx】这家店的核心理念是什么？***

Vuex 商店由以下部分组成:

*   **状态**:一个复杂的对象，收集了将在应用程序中传播的所有数据
*   **Getters** :应用程序状态的访问器(它们可以被参数化)
*   **突变**:使*状态突变*的一组函数
*   **动作**:动作调用(或*提交*)变异，并且是异步的(在调用后端 API 时很有用)
*   **模块**:之前关键部分的子集。它允许您将状态/获取器/突变/动作集中在一个专用的特性上。因此，主状态将由几个模块组成。

![](img/bc030264d0d23087d7609c3823ed0330.png)

来自[https://vuex.vuejs.org/#what-is-a-state-management-pattern](https://vuex.vuejs.org/#what-is-a-state-management-pattern)

# 打字稿[TS]

> **【TS】什么是 Typescript？**

Typescript 是 Javascript 的一个超集，它根据 OOP 原则增强了代码。

> ***【TS】ES6 和 Typescript 有什么区别？***

与 ES6 类封装不同，Typescript 类可以扩展另一个类或实现一个接口

这里有一些比较点:

`**Description**`

TS 是 Javascript 的超级集合(到达它的顶端)。这种免费的开源编程语言是由微软开发和维护的。在构建时，它被编译成 JS
**ES6** 是 ECMAScript (ES)的一个版本，是由 ECMA 国际标准化的脚本语言规范

`**Benefits**`

**TS** 支持所有原始数据类型+ `any`、`unknown`、`never` +封装(*私有*、*保护*、*公共字段* )
**ES6** 不支持

`**Scoping**`

**TS** 支持 3 种类型的变量作用域:*全局*，*类*，*局部* **ES6** 仅支持*全局*和*局部*作用域

`**Modules**`

**TS** 允许创建、管理两种类型的模块:内部和外部
ES6 模块，分类为导入模块或导出模块

更多详情:

[](https://www.educba.com/typescript-vs-es6/) [## Typescript 与 ES6 |您需要知道的最有用的 7 个区别

### Typescript vs ES6 指南。在这里，我们还将讨论直接比较、关键差异以及信息图和…

www.educba.com](https://www.educba.com/typescript-vs-es6/) 

> ***【TS】一个接口可以扩展一个类吗？***

是的，这是一个例子:

```
class Point {
  x: number;
  y: number;
}
interface Point3D extends Point {
  z: number;
}
```

您可以创建一个名为`Point3D`的新接口，它扩展了`Point`类。

## 带打字稿的 Vue

> 使用 Typescript 编写 Vue 组件有哪两种可能的方法？

```
import Vue from 'vue' 
import Component from 'vue-class-component'// Option #1
**export default Vue.extend**({ ... })// Option #2 (class-style component syntax)
**@Component({**
  // All component options are allowed in here
**})
export default class MyComponent extends Vue** { ... }
```

> ***【V w/TS】如何为 Vue 实例中的*** `***data***` ***和*** `***props***` ***定义类型？***

向*投*一个`**data**` 或*`**props**`**4 种语法都是可能的:***

```
***props: {
  myProp: { type: **Object as unknown as MyObject** },    // #1
  myArray: { type: **Array as unknown as** **MyObject[]** },
},
data: () => ({
  myDataset: **[] as unknown as () => Data[]**,           // #2
  anotherData: data **as () => AnotherData**,             // #3
  myPropFromData: this.myProp **as MyObject** //#4
})***
```

> ******【V w/TS】我们可以在 vue 模板中使用 Typescript 吗？******

***是的，我们可以，`enums`就是一个很好的例子，如下片段所示:***

> ******【V】如何在 Vue 构造函数中全局声明变量或函数？******

***这可以通过使用[模块增强](https://vuejs.org/v2/guide/typescript.html#Augmenting-Types-for-Use-with-Plugins)来实现。TL；博士:***

# ***HTML/CSS [HC]***

> ******【HC】说出一些 CSS 布局概念。******

***两个主要布局概念是:***

*   ****网格系统* ( [更多详情](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout))***
*   ***[*Flexbox*](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox) *系统*(完整指南[此处](https://css-tricks.com/snippets/css/a-guide-to-flexbox/))***

> ******【HC】什么是 Flexbox？******

***Flexbox 是一种使用灵活的盒子在 CSS 布局中构建行和列的方法。***

# ***网络开发文化***

> ******【WDC】什么是响应性？怎么处理？******

***响应性就是让网站适应读者的屏幕(桌面、智能手机、平板电脑)。***

***为了使网站响应迅速，我们使用 [CSS 媒体查询](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries)。它包括一组规则，这些规则将根据例如设备类型和/或媒体特征来应用。***

> ******【WDC】i18n 和 a11y 分别代表什么？******

***`**i18n**`代表 **i** 国际化 **n** (国际化= 18 个字母)。是一个网站适应读者母语的能力( [vue-i18n](https://kazupon.github.io/vue-i18n/) )。***

***`**a11y**`代表 **a** 可访问性 **y** (可访问性= 11 个字母)。这是一个网站考虑各种读者的能力，包括那些有障碍的读者。可能是添加`aria`属性，提供高对比度模式，管理键盘导航，… ( [a11y 项目](https://a11yproject.com/))。***

> ******【WDC】说出并解释你所知道的浏览器 WebAPIs。******

***`[**IntersectionObserver**](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)` 异步观察一个元素是否在视口中。***

***`[**Notification API**](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API/Using_the_Notifications_API)` 在浏览器中管理通知。***

***`[**Fetch API**](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)` 通过 web 获取资源得益于`Request`和`Response`标准化对象。***

***`[**L10n API**](https://developer.mozilla.org/en-US/docs/Archive/B2G_OS/API/L10n_API)` 定位设备。***

***您可以在此找到更多 Web APIs:***

***[](https://developer.mozilla.org/en-US/docs/Web/API) [## Web APIs

### 为 Web 编写代码时，有大量的 Web APIs 可用。下面是所有 API 和…

developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/API) 

# 结尾词…

有些问题看起来很难，但这是测试应聘者好奇心的一种方式。就把面试看作是测试应聘者知识和学习技能的一种方式吧。***