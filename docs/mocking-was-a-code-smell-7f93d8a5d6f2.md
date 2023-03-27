# “嘲讽是一种码味”，没有类型。

> 原文：<https://itnext.io/mocking-was-a-code-smell-7f93d8a5d6f2?source=collection_archive---------0----------------------->

> *当我们的分解策略失败时，嘲讽是必须的。——*[埃里克·埃利奥特](https://medium.com/javascript-scene/mocking-is-a-code-smell-944a70c90a6a)

![](img/503808463e3cd50c9614e81f67961df9.png)

来自“如何发现代码味道并不让它变坏，第 2 部分”

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fmocking-was-a-code-smell-7f93d8a5d6f2)

首先你要明白，Javascript 中的**嘲讽不是代码味**，因为 JS 中的嘲讽更类似于真正的 DI，而不是“普通语言”中的嘲讽。

> 流行的库，如 Proxyquire 或 mock，只是重新连接内部节点逻辑，并将`require`或`import`请求重新路由到某个合成文件。
> 
> 如果你只是用 AMD 语法替换“ES6”或 CommonJS 模块，你将得到“DI”。NodeJS 或 Webpack 是 IoC 框架。

第二，你要明白 Javascript 中的**嘲讽是一种代码气味**。惊喜！但是，老实说，

> 当你嘲笑你的测试时，你是在测试你的模仿。不是真正的代码。

比如—TS 界有一个通用的 tslint 规则—“[无默认导出](https://palantir.github.io/tslint/rules/no-default-export/)”。默认导出是一种“代码味道”,因为您可以更改模块的内部，而外部用户对此一无所知。我不知道你们内部的烹饪。

无论如何，如果你改变你的模拟依赖关系，你的生产**代码会破坏**，但是你的**测试不会破坏**！他们没有使用真正的依赖和真正的 API。测试是(快乐的)自己生活。

![](img/61e81ffc0fca47ec35bf4a1c9b2621e0.png)

当测试是绿色的，但是 prod 被淹没…

# 对测试的信心？

第一层信心是关于正确的模拟名称——然后你就可以确定你确实需要你要模拟的文件，并且你最终模拟了它。不像在声音中那样简单，尤其是对于 proxyquire。

第二层信心是关于具体化一个文件的已知依赖关系。如果目标文件获得或失去依赖性，您应该知道。Proxyquire 毫无价值，但是嘲弄可以提供一些(小)支持。

第三层信心是正确地模仿真实文件。提供一个正确的模拟，这将正确覆盖真正的出口。

让我们想象下面的代码:

```
// a.js
import MainAction, { helper } from './b.js';
export default () => MainAction(helper(42));// b.js
export const helper = (a) => a * 2;
export default b => SuperNetworkCall(b); // should be mocked
```

接下来，你有一个测试，完全模仿 b.js，一切都很棒……接下来——你改变 b.js，使其更加一致..

```
export const helper = (a) => a * 2;
export const action = b => SuperNetworkCall(b);
```

**测试是绿色的。房地产是红色的。生产已死。很难理解和解决根本问题，因为没有人会帮助和指导你。有一天**我**刚**改名**一个叫出口**花了一个小时**才明白为什么我的存根*不叫*。**

> 这就是我做这一切的原因。只是为了减轻我自己的生活。

静态类型检查可以解决所有的问题。并消除 javascript 模仿的味道。

# 类型来拯救！

如果使用严格的类型检查(TypeScript 和 Flow ),就不会发生这种情况。甚至更多——实际上，您可以仅在测试中使用 TypeScript 或 Flow，如果您遇到问题，可以在全局范围内使用它们。

但是——没有办法对模拟进行类型检查。模拟的标准接口做不到这一点。没有办法“得到”你应该匹配的类型。

Rewiremock v3 引入了一个新的 API

```
// common API
rewiremock('./b.js').withDefault(overrideDefault).with({ helper:2});// new async API
rewiremock(() => import('./b.js')).withDefault(overrideDefault)...
```

使用 imports 启动魔法:`mock`现在不是一个字符串，而是一个实际的模块——系统可以访问模块的出口类型。

不幸的是，旧的不能工作，你必须使用导入，并且你必须在导入和模拟声明之间有一个“链接”,所以“简单”的结构像…

```
const mocked = rewiremock.proxy('fileName.js',{
   'dep1.js': { override: "me" }
 });
```

…不适合。只有“mock-like”(rewiremock 的原生语法)可以工作。但是使用真正的导入完全解决了名称解析问题。

# 开拍！

给定一个代码:

```
rewiremock(() => require('./b.js'))
  .withDefault(overrideDefault)// orrewiremock(() => import('./b.js'))
  .withDefault(overrideDefault)
```

因此，在您将`default export`从`b.js`中移除后，您将得到一个异常:

```
TypeScript
error TS2339: Property 'withDefault' does not exist on type 'NamedModuleMock<typeof "b.js">'.FLOW: 
withDefault(fn: $PropertyType<$Exact<T>, 'default'>)
Property not found in () => import('./b.js')
                  ^^^^^^^^^^^^^^^ exports of "./b.js".
```

TypeScript 支持是完美的。withDefault 只是在没有默认导出的情况下不存在。但是 Flow 远不如 sound(而 long Flow 作为 type system 是 _more_ sound)，而且不容易理解错误消息。

具名出口的情况更好。

给定一个代码

```
rewiremock(() => require('./b.js'))
  .with({
    halper: 'help!',
  })
```

我在这里打了个错别字(你在这篇文章里发现了几个错别字？)—两种类型的系统都会清楚地报告。

```
TypeScript:
Object literal may only specify known properties, and 'halper' does not exist in type '{ readonly helper.... }'.Flow:
.with({halper: () => 10});
       ^^ property `halper` of object literal. Property not found...
```

如果你用一个数字模拟函数，或者用对象类型模拟数字，系统将抛出。换句话说，如果你做错了什么，它会一直通知你。

“只是类型”呢？

```
const file = rewiremock.proxy('a.js'); // 👎 not typed. How it can be?// using .proxy helperconst file = rewiremock.proxy(() => require('a.js')); // 👍// using plain requirerewiremock.enable();
const file = require('a.js'); // 👍, sure - that's requireconst file = await import('a.js'); // 👍, sure - that's import
// 😉 ^ thats ESM compatible
```

# 类型..类型…

花了两个月的时间重新发明轮子，并理解如何提供一个类型安全的模拟。创建 TypeScript d.ts 定义花了一个小时，创建 Flow variant 花了将近一个星期。启用 rewiremock 摘要函数作为模拟名称需要几分钟时间。

没错。这里的关键点是，所有的魔法都在外面。在 d.ts 或 index.js.flow 文件中。向 TS 和流量创造者致敬。向 TS 和流程维护者致敬。如果你正在嘲笑你的 TS 或 Flow 应用程序中的依赖关系——恭喜你，让我们最终让 javascript 嘲笑伟大的 aga……

# 结论:

Rewiremock，我为`fix`构建的嘲讽库，嘲讽现在已经到了 v3。

这是一个混合的 Proxyquire，嘲弄，和所有“可以做得更好”我曾经发现。它可以模仿任何东西，提供隔离模式，不仅可以帮助你模仿 node.js，还可以模仿 webpack decencies，现在它获得了类型安全。

[](https://github.com/theKashey/rewiremock) [## theKashey/rewiremock

### rewiremock——在 Node.js 或 webpack 环境中模拟依赖关系的正确方法。

github.com](https://github.com/theKashey/rewiremock) 

阅读更多关于 rewiremock 如何工作以及为什么工作的信息

[](https://medium.com/tag/rewiremock/latest) [## 关于 Rewiremock - Medium 的最新故事和新闻

### 阅读关于 Rewiremock 的最新文章。每天，成千上万的声音在阅读、写作和分享重要的故事…

medium.com](https://medium.com/tag/rewiremock/latest) 

对于 JS，ES 或者 TS 来说是最好的嘲讽库，我还是很确定的。

# 那么他们现在闻起来怎么样？

首先，请阅读关于仿制品气味的文章。请记住，好的代码不是没有模仿的代码，这一点在文章中已经指出。

它们相互垂直。有能力静态分析你的代码-是一个加分。有能力跳到实现(需要隐式(紧密)声明)——是一个加号。

[](https://medium.com/javascript-scene/mocking-is-a-code-smell-944a70c90a6a) [## 嘲讽是一种代码气味

### 注意:这是学习函数式编程和组合软件的“组合软件”系列的一部分…

medium.com](https://medium.com/javascript-scene/mocking-is-a-code-smell-944a70c90a6a) 

但是拙劣的模仿会毁掉一切。普通的嘲笑会毁掉一切。Javascript 模仿太假了。太假了。

# PS: Jest

Jest 是一个非常受欢迎的任务运行者，它有自己的模拟系统，包括手动、魔法和自动锁定。

Rewiremock 仍然可以帮助你，但它将只是取代一些 Jest 的内部，这并不总是可以接受的。为了获得与自动锁定相同的安全类型，即`__mocks__`你可以使用。这只是一个很酷的 cli 工具。

[](https://github.com/theKashey/jest-typed-mock) [## Kashey/jest-typed-mock

### 玩笑式的嘲弄式的安全！。并利用 TS/Flow 的力量让 mocks 变得更好。

github.com](https://github.com/theKashey/jest-typed-mock)