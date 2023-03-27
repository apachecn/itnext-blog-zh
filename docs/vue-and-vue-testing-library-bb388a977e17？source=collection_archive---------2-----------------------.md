# Vue 和 Vue 测试库

> 原文：<https://itnext.io/vue-and-vue-testing-library-bb388a977e17?source=collection_archive---------2----------------------->

![](img/e2d5c828ecf47f5e383cdab97b91e12f.png)

克里斯蒂安·霍尔辛格在 [Unsplash](https://unsplash.com/search/photos/neusiedlersee?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

我是 TDD(测试驱动开发)的超级粉丝，我真的建议每个人都去实践它，或者，至少，用一些形或小练习来尝试一下。

今天，有了相关测试库支持的最新 UI 框架，TDD 比以前容易多了，它实际上帮助了很多定义、捕捉和测试可能的组件场景。

在用*酶*测试*反应*组件时，我清楚地看到了这些实践的好处(在那些库的真正早期阶段)。如果组件易于测试(和布局场景)，那么您就走在了一条好的道路上。

在我的职业生涯中，我经历过许多不同 UI 技术、方法、过程和实践的单元测试。

看看下面的图片，你可能会发现**单元测试**应该是测试策略的基础。为什么？其中一个原因是，它们很快，而且可以被隔离。

![](img/6275717b7bf8c4b2b4171a4b08cfc78c.png)

迈克·科恩的概念。图片摘自[https://Martin fowler . com/articles/practical-test-pyramid . html](https://martinfowler.com/articles/practical-test-pyramid.html)

在这篇文章中，我想展示一种方法，这种方法可能有助于我们的测试代码更加灵活，更重要的是，**遵循最终用户方法**(而不是*开发人员方法**)。*

****Vue*** 和***Vue/Test-Utils***是一个很棒的组合！我真的很喜欢它，用传入的道具、商店等等测试组件真的很容易。我非常推荐这些技术，并查看了这两个资源(第一个是 Vue/Test-Utils 的官方页面，第二个是 Vue 的 UT-TDD 实践/解决方案的集合——太棒了):*

 *[## 简介| Vue 测试工具

### 测试 Vue 组件的实用程序

vue-test-utils.vuejs.org](https://vue-test-utils.vuejs.org/)* *[](https://github.com/lmiller1990/vue-testing-handbook) [## lmiller 1990/vue-测试手册

### Vue 组件和应用测试指南-lmiller 1990/Vue-测试-手册

github.com](https://github.com/lmiller1990/vue-testing-handbook)* 

*这真的很棒，但它可能会与另一个名为“***Vue-testing-library***”的库相结合，该库强制执行良好的测试实践和更为 **BDD 化的**方法(*行为驱动开发*)。*

*一种 BDD 也可以在单元测试中实现，将 TDD 测试用例与 BDD 测试用例分开是一个很好的实践。BDD 更关注与功能/组件相关的实际行为，而不是“输入/输出”,我想展示一下在这些情况下“ *vue-testing-library* ”是如何方便的。*

*根据记录， *Vue-testing-library* *基于 Vue/Test-Utils 的 top* ，所以你可以只用 Vue/Test-Utils 实现类似的东西，但是它需要更多的自律。*

*可以用一个命令安装它:*

```
*npm install --save-dev vue-testing-library*
```

*在这个例子中，我使用了 **Jest-Dom** ，它与 **Jest** 结合起来非常有用(它有助于 Dom 断言和期望)。*

*[](https://www.npmjs.com/package/jest-dom) [## 笑话世界

### 自定义 jest 匹配器来测试 DOM 的状态

www.npmjs.com](https://www.npmjs.com/package/jest-dom) 

一个简单的挑战是流行的*“计数器”*，最终用户可以通过页面上的几个按钮增加或减少一个数字。

首先，让我们从最终用户的角度关注这一点，并收集需求:

*   显示实际数字的文本(我们可能期望开始时是 0)；
*   一个显示“*增加*的按钮，它应该增加数字；
*   一个显示“*减少*的按钮，该按钮将减少数字；

让我们编写我们的第一个测试用例"*一个显示实际数字的文本(我们可能期望开始时是 0)*:

```
//counter.spec.jsimport *{* render *}* from 'vue-testing-library';
import Counter from '@/components/Counter.vue'

describe*(*'**Component: Counter.vue**', *()* => *{* describe*(*'**Behavior**', *()* => *{* test*(*'**User should see 0 at the beginning**', *()* => *{* const *{* getByText *}* = render*(*Counter*)*;
            const textNumberWrapper = getByText*(*/Counter is:/*)*;
            expect*(*textNumberWrapper*)*.toHaveTextContent*(*'Counter is: 0'*)*;
        *})*;
    *})*;
*})*;
```

导入的 *Render* 函数将负责独立安装所需的组件，并返回一组特定于包装实例的实用程序。更多信息可在以下链接中找到:

[](https://testing-library.com/docs/vue-testing-library/api) [## API 测试库

### Vue 测试库从 DOM 测试库中重新导出所有东西。它还公开了这些方法:render 函数…

testing-library.com](https://testing-library.com/docs/vue-testing-library/api) 

测试用例中的另外两条指令实际上很简单:第一条指令选择包含所需文本的元素，最后一条指令只是检查先前找到的元素包含的文本是否是我们所期望的(“*计数器在开始时应该是 0*”)。

将该测试转化为成功测试的一个简单方法是以下 Vue 组件实现:

```
// src/components/Counter.vue*<*template*>
    <*div*>
        <*div class="counter-text"*>* Counter is: 0
        *</*div*>
    </*div*>
</*template*>*
```

您可能会注意到，我们的测试并不搜索特定的标签。这是一个好处，因为它有助于依赖测试优先开发，并留有一定的灵活性。此外，终端用户并不关注 HTML 标签的本质或层次结构，而只关注显示的内容。

所以，现在你可以想象增加按钮的测试用例，看起来可能如下:

```
// counter.spec.jsimport *{* render,
    fireEvent
*}* from 'vue-testing-library';
import Counter, *{* INITIAL_COUNTER *}* from '@/components/Counter.vue'

describe*(*'Component: Counter.vue', *()* => *{* describe*(*'Behavior', *()* => *{
        ...*
        test*(*'**When User clicks on Increase, Counter should be increased by 1 unit**', **async** *()* => *{* const *{* getByText *}* = render*(*Counter*)*;
            const textNumberWrapper = getByText*(*/Counter is:/*)*;
            expect*(*textNumberWrapper*)*.toHaveTextContent*(*`Counter is: $*{*INITIAL_COUNTER*}*`*)*;
            **await** fireEvent.click*(*getByText*(*'Increase'*))*;
            expect*(*textNumberWrapper*)*.toHaveTextContent*(*`Counter is: $*{*INITIAL_COUNTER + 1*}*`*)*;
        *})*;
    *})*;
*})*;
```

首先，您可以看到这个测试用例正在异步运行，因为 fireEvent 函数正在返回一个承诺，我们正在等待 await 关键字的执行。

这个测试用例非常简单:

*   它将组件呈现为隔离状态；
*   它检查我们的计数器跟踪器是否显示初始状态；
*   它在任何带有“增加”文本的元素上触发一个点击事件，并等待执行；
*   最后，它检查我们的计数器跟踪器是否更新了一个新的数字(在这种情况下，我们期望多 1 个单位)。

为了解决这个测试用例，让我们实现这个组件:

```
// src/components/Counter.vue*<*template*>
    <*div*>
        <*div class="counter-text"*>* Counter is: {{ count }}
        *</*div*>
        <*div class="controls"*>
            <*button @click="increase"*>*Increase*</*button*>
        </*div*>
    </*div*>
</*template*>
<*script lang="js"*>* export const INITIAL_COUNTER = 0;

    export default *{* name: 'Counter',
        data *() {* return *{* count: INITIAL_COUNTER
            *}*;
        *}*,
        methods: *{* increase*() {* this.count += 1;
            *}
        }
    }
</*script*>*
```

酷！！

减少功能呢？

```
// counter.spec.js...describe*(*'Component: Counter.vue', *()* => *{* describe*(*'Behavior', *()* => *{
        ...*
        test*(*'**When User clicks on Decrease, Counter should be decreased by 1 unit**', async *()* => *{* const *{* getByText *}* = render*(*Counter*)*;
            const textNumberWrapper = getByText*(*/Counter is:/*)*;
            expect*(*textNumberWrapper*)*.toHaveTextContent*(*`Counter is: $*{*INITIAL_COUNTER*}*`*)*;
            await fireEvent.click*(*getByText*(*'Decrease'*))*;
            expect*(*textNumberWrapper*)*.toHaveTextContent*(*`Counter is: $*{*INITIAL_COUNTER - 1*}*`*)*;
        *})*;
    *})*;
*})*;
```

并且实现类似于另一个:

```
// src/components/Counter.vue*<*template*>
    <*div*>
        ...
        <*div class="controls"*>
            ...
            <*button @click="decrease"*>*Decrease*</*button*>
        </*div*>
    </*div*>
</*template*>
<*script lang="js"*>* export const INITIAL_COUNTER = 0;

    export default *{* name: 'Counter',
        data *() {* return *{* count: INITIAL_COUNTER
            *}*;
        *}*,
        // define methods under the `methods` object
        methods: *{* increase*() {* this.count += 1;
            *}*,
            decrease*() {* this.count -= 1;
            *}
        }
    }
</*script*>*
```

完整的示例如下:

Jest 输出如下:

> 通过测试/单元/计数器. spec.js
> 
> 组件:Counter.vue
> 
> 行为
> 
> 用户应该在开头看到 0
> 
> 当用户点击增加时，计数器应增加 1 个单位
> 
> 当用户点击减少时，计数器应减少 1 个单位

现在有趣的部分来了。

如果您更改标签或层次结构(例如:更改类，span 而不是 div，或者不同的嵌套 HTML 标签)，会发生什么？除非我们修改文本，否则没有什么会真正影响我们的测试。太棒了。

以我的拙见，我真的相信这是在我们的单元测试中覆盖用户行为的一个很好的方式，因为这些测试运行得非常快并且是孤立的(并且它们对实现的变化也很灵活)。

您可以在以下链接中找到更多示例:

[](https://github.com/testing-library/vue-testing-library/tree/master/tests/__tests__) [## 测试库/vue 测试库

### 轻量级适配器允许使用 dom-testing-library 测试构建在@vue/test-utils 之上的 Vue.js 组件…

github.com](https://github.com/testing-library/vue-testing-library/tree/master/tests/__tests__) 

干杯:)*