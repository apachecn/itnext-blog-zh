# 不要做“这个”——第二部分

> 原文：<https://itnext.io/dont-do-this-part-two-the-trouble-with-typescript-8ea9b26892e2?source=collection_archive---------0----------------------->

![](img/f0b559e93cd9ad6d5a833c221ac718ff.png)

瑞士阿尔卑斯山的漫步(图片由作者提供)

## 打字稿的问题是

# 背景

我在之前的一篇文章中指出，“该死的 Javascript 编码员说的是‘T1’”，没有人应该使用曾经的[打字稿](https://www.typescriptlang.org)，我认为最好是我详细解释一下为什么我认为是这样。

这是由两部分组成的文章的第二部分。跳到[前一部分](https://medium.com/@davesag/dont-do-this-part-one-objects-and-their-misuse-bd0771c3178a)，在那里我对面向对象软件进行了大规模的倾倒，尤其是当它与 Javascript 相关的时候。

# 所以让我们来谈谈 Typescript

Typescript 是 Javascript 的强类型(因此得名)超集。对于来自强类型 OO 语言(如 Java 或 C#)的开发人员来说，这是一种很好的方式，可以用强类型和 IDE 代码完成的安全毯来开发 Javascript 应用程序。对于没有经验或不称职的开发人员来说，这也是一个非常好的方法来创造一个真正壮观的技术债务山。

Typescript 鼓励开发人员认为他们是在使用类型安全和面向对象的语言工作。但事实是，Typescript 可以编译成 Javascript，并允许您将您非常熟悉的类型安全的 OO 代码与原始的、纯粹的 Javascript 混合在一起。这就是问题所在。

Javascript 本质上并不是一种面向对象语言。当然，它假装是一个，而且，如果你小心的话，你可以写 OO 风格的 Javascript，但是如果你出错了，你*就会*出错，那么你就会遇到很多很难调试的小错误。

## 打字稿是拐杖

但这是一个松散的，不稳定的拐杖。

Typescript 鼓励使用 OO。在[第一部分](/dont-do-this-part-one-objects-and-their-misuse-bd0771c3178a)中，我讨论了面向对象，特别指出了 Javascript 的面向对象是应该避免的，除非在极少数情况下你真的想要可重用的对象。

## Typescript 冗长而难看

这里有一个例子。对象分解是任何 Javascript 开发人员工具箱中的重要工具。但是 Typescript 真的会让你搞砸。

将属性名从一个对象映射到另一个对象是一项非常常见的任务，比方说，因为您已经从一个 API 中提取了一些 JSON，并且您需要不同的属性名。所以你可以写一个映射函数，比如:

```
const mapUser = ({
  user_name: username,
  first_name: firstName,
  last_name: lastName
}) => ({
  username,
  firstName,
  lastName
})
```

这是完全有效的，也是非常可疑的 Javascript，但是它肯定会把 Typescript 搞得一团糟。要在 Typescript 中编写它，您需要像这样做:

```
const mapUser = ({
  user_name: username,
  first_name: firstName,
  last_name: lastName
} : {
  username: string,
  firstName: string,
  lastname: string
}) => ({
  username,
  firstName,
  lastName
} : any);
```

一个非常简洁的映射函数现在变成了一堆可怕的重复，所有这些都是为了确保你的输入是字符串。这还不包括有毒的 T2 仿制药 T3。扔一些尖括号在那里，为所有失去的美丽掉一滴眼泪。

如果您使用该函数来映射 JSON 文件中的字段，那么无论如何，您的花哨的类型安全检查已经过时了。

```
const getUsers = async url => {
  const { json } = await fetch(url)
  const data = await json()
  return data.map(mapUser)
}
```

Typescript 怎么知道`data`里有什么？并没有。而且一旦被编译成 Javascript，Typescript 假定的类型安全就消失了。

## Typescript 鼓励 IDE 依赖

也许这只是我的看法，但我实在无法忍受大多数想法。人们似乎喜欢它们的一些东西，比如自动完成，让我抓狂。软件开发不是靠提高你的打字速度来提高的，而是靠提高你的推理能力来提高的。只要你有体面的语法着色，一个集成的文件浏览器，和良好的搜索和替换，这真的是所有体面的 Javascript 开发人员需要的。我使用 Atom 和很少的插件。我有时也会使用 TextMate，因为出于某种难以理解的原因，Atom 不允许您拖动文件从一个项目复制到另一个项目。

但是要正确使用 Typescript，你真的需要购买整个微软 VS Code 的东西，或者在 WebStorm 或更糟的地方混日子。来自 Java 或 C#的人可能会渴望 Eclipse 或 NetBeans，但其他人不会。

## Typescript 引入了自己奇怪的错误

在他的博客文章“[打字稿的好、坏、丑](https://blog.codeminer42.com/the-good-the-bad-and-the-ugly-of-typescript-58a3ff3e248)”中，莱昂纳多·弗雷塔斯解释道:

> 如果你像我一样是一个函数式编程的人，将 curried 方法与承诺联系起来有时会有问题。这是因为一些框架使用了原生的 ES Promises，而另一些使用了第三方 promise 库，比如 [Bluebird](http://bluebirdjs.com/docs/getting-started.html) 。尽管在 JS 中链接它们没有问题，但 TypeScript 认为它们是不同的类型，甚至会拒绝传输，因此抛出了一个`*TypeError*`。

## Typescript 意味着您需要一个预处理器

前端开发人员非常习惯于使用像 [Babel](https://babeljs.io) 这样的预处理器和像 [Webpack](https://webpack.js.org) 这样的构建工具来将他们的超级现代 Javascript 转换成浏览器可以理解的东西。但是对于使用 NodeJS 的后端开发人员来说，不需要构建工具，预处理器只是增加了一层烦人的复杂性，没有任何价值。部署一个需要在使用之前就构建好的服务器是令人厌烦的。

## 使用 Typescript 在单元测试中进行模拟更加困难

前端开发人员可以使用像`[Jest](https://jestjs.io)`这样的测试框架，其中包括真正令人惊讶的模仿能力。但是，当您将 Typescript 添加到组合中时，这些[就会变得混乱](https://stackoverflow.com/search?q=jest+typescript)。对于使用 Node 的后端开发人员来说，更常用的是`[mocha](https://mochajs.org)`测试框架，以及像`proxyquire`这样的模仿工具，插入*任何*类型的预处理构建步骤都会使您的测试设置更加困难。

单元测试很重要。我读过各种各样的文章，声称使用 Typescript 时不需要进行严格的单元测试，因为它具有编译时的类型安全性，但这只是无稽之谈。我也读到过这样的观点:如果你在嘲笑，你就是在做错误的测试；那也是废话。事实恰恰相反。

## 题外话:为嘲笑辩护

真正的单元测试只测试有问题的特定代码单元。如果您的代码是这样的:

```
const fetch = require('node-fetch)
const mapUser = require('src/utils/mapUser')const getUsers = async url => {
  const { json } = await fetch(url)
  const data = await json()
  return data.map(mapUser)
}
```

您可以像这样对它进行单元测试:

```
const { expect } = require('chai')
const { stub } = require('sinon')
const proxyquire = require('proxyquire')describe('getUsers', () => {
  const fetch = stub()
  const json = stub()
  const data = ['some data']
  const mapUser = stub()const response = { json }
  const url = 'some-url'const getUsers = proxyquire('src/getUsers', {
    'node-fetch': fetch,
    'src/utils/mapUser': mapUser
  })
  const expected = ['some result']let resultbefore(async () => {
    fetch.resolves(response)
    json.resolves(data)
    mapUser.returns(expected[0])
    result = await getUsers(url)
  })

  it('called fetch with the supplied url', () => {
    expect(fetch).to.have.been.calledWith(url)
  })

  it('called json', () => {
    expect(json).to.have.been.calledOnce
  })

  it('called mapUser once with the right data', () => {
    expect(mapUser).to.have.been.calledOnceWith(data[0])
  })

  it('returned the expected result', () => {
    expect(result).to.deep.equal(expected)
  })
})
```

像这样的测试既全面又快速。任何编写单元测试而不无情嘲讽的人，根本就不是在编写高效的单元测试。

单元测试当然不能替代实际的集成测试，但那是另一回事了。

## Typescript 是编程语言的宝库

你知道沃尔沃的司机认为他们是世界上最安全的司机，因为他们驾驶的汽车据称有更多的安全功能。但是这种虚假的安全感实际上会让他们成为危险的不安全司机。

**牛群** — 1997 年由[希瑟·克罗尔](https://en.wikipedia.org/wiki/Heather_Croall)创作，音乐由[汤姆·史密斯](https://www.youtube.com/watch?v=-h9j5N8aX1M)创作。经许可使用。

# 第二部分到此结束

回到[第一部分](https://medium.com/@davesag/dont-do-this-part-one-objects-and-their-misuse-bd0771c3178a)阅读我对面向对象的一般性批评，尤其是因为它是用 Javascript 完成的。

## 我们了解到:

*   Typescript 鼓励过度使用对象
*   Typescript 给了开发人员一种虚假的安全感
*   Typescript 使您的代码变得难看并且更难测试
*   不应该使用 Typescript

# 链接

## 文章

*   [打字稿的好、坏、丑](https://blog.codeminer42.com/the-good-the-bad-and-the-ugly-of-typescript-58a3ff3e248)
*   [狗屁 Javascript 编码员说](https://medium.com/@davesag/shit-javascript-coders-say-7a2d2881228d)

## 技术

*   [打字稿](https://www.typescriptlang.org)
*   [NodeJS](https://nodejs.org)
*   [摩卡](https://mochajs.org)
*   [代理查询](https://github.com/thlorenz/proxyquire)
*   [巴别塔](https://babeljs.io)
*   [网络包](https://webpack.js.org)
*   [笑话](https://jestjs.io)
*   [手动模拟](https://jestjs.io/docs/en/manual-mocks.html)

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。