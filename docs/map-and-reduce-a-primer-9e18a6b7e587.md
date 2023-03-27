# 地图和缩小-初级读本

> 原文：<https://itnext.io/map-and-reduce-a-primer-9e18a6b7e587?source=collection_archive---------7----------------------->

![](img/b860724d1dd5efa73e5f1118ac97c8b0.png)

拉斯伯里还原法。作者照片。

## 明智地使用 map 和 reduce 函数是一项核心技能，但很少有效地使用。

`map`函数是由所有`Iterable`对象(`Arrays`、`Sets`、`Lists`等)提供的，它允许你遍历条目，对条目进行转换，并返回相应的转换后的条目集合。

```
export const VERBS = ['start', 'stop']
export const ACTIONS = VERBS.**map**(verb => import(`acts/${verb}`))
```

这对于很多事情来说都很方便，但是上面所说的并不是非常有用。要得到一个函数，你必须做这样的事情:

```
import { VERBS, ACTIONS } from 'acts'export default function action(verb) {
  const i = VERBS.indexOf(verb)
  return ACTIONS[I]
}
```

更有用的是，如果你把`VERBS`转换成一个对象，你可以用它来通过名字查找正确的`action`。

一个**缩减器**是一个函数，它接受一个累积值和下一个数组项，对它们做一些事情，然后返回新的累积值。

```
array.reduce(reducer, init) // => some new array or object
```

示例:

```
export const VERBS = ['start', 'stop']
export const ACTIONS = VERBS.reduce((acc, verb) => {
  acc[verb] = import(`acts/${verb}`)
  return acc
}, {})
```

因此，对于每一个`verb`，你都要在提供的`{}`中添加一个`key:value`对，产生:

```
{
  start: import('acts/start'),
  stop: import('acts/stop')
}
```

将上面的`action`功能变为:

```
import { ACTIONS } from 'acts'export default function action(verb) {
  return ACTIONS[verb]
}
```

## 一个六个，另一个半打

有一种观点认为，如果将'*数组查找*'策略与'*使其成为对象*'策略进行比较，两者的代码行数大致相同。两者的意图都很清楚。

为什么将`VERBS`数组`reduce`到一个对象比将其`map`到另一个数组更好？

这个问题的答案在于优化那些消费数据的人。

在对象中查找`key`是一种语言特性，因此你可以依赖它进行高度优化。通过`import`对`actions`的实际设置通常是每个应用一次*，或者至少每个异常状态变化*一次*。这是一个常识性概念的直接应用，即你*索引你的文章*，而不是把工作推给你的读者。*

![](img/076e312b11b380ba2b96f1623898de4f.png)

索引你的写作。收拾餐具时要分类。

这也是为什么在打开洗碗机时，我会确保汤勺先放在顶端，甜点勺最后放在顶端。我可以在黑暗中瞬间找回正确的勺子。

我用大大小小的叉子也是这样，但奇怪的是，我从来没有觉得有必要用刀子这样做。

## 承诺呢？

你可以使用`map`和`reduce`来创建`Promises`的集合。让我们假设我们有一个带有异步`findByName`函数的`Person`数据库模型，并且我们想要编写一个异步函数`people`，它接受一个`names`数组并进行查找，然后返回相关的`Person`记录。

```
import { Person } from 'models'export default async function people(names) {
  return names.map(Person.findByName)
}
```

因为是一个`async`函数，`people`返回的不是一个`Person`记录的数组，而是一个*将*解析为一个未解析`Promises`的数组的`Promise`，每个数组最终*将*解析为一个`Person`。这是一个很常见的 bug。

下面就不行了，虽然看起来没错。

```
import people from './people'const summarise = ({ id, name, email }) => ({ id, name, email })async function meAndMyFriends(me, names) {
  const friends = (await people(names)).map(summarise)
  return { ...me, friends }
}
```

这里的`friends`数组将是一个:

```
{
  id: undefined,
  name: undefined,
  email: undefined
}
```

朱利叶斯·萨姆纳·米勒在 20 世纪 80 年代游历了澳大利亚，在他去世前的一段时间里，作为一种特殊的客串明星，在我的学校教了几堂物理课。他的电视节目“为什么会这样？”在我这个年龄的孩子中很受欢迎。就个人而言，他是一个表演大师，一个神话般的科学传播者，也可能是我见过的最了不起的教育家。我从来不会问自己“为什么”，而不加上“是这样吗？”

## 为什么会这样呢？

因为一个`Promise`的实例没有`id`、`name`或`email`。

为了解决这个问题，`people`函数需要首先将`names.map`包装在`Promise.all`中，其中*将`Promises`的数组*解析为保存结果数组的单个`Promise`:

```
import { Person } from 'models'export default async function people(names) {
  return Promise.all(names.map(Person.findByName))
}
```

现在`meAndMyFriends`函数将返回预期值。

## 用`reduce`连载承诺。

当您有一个`Promises`数组，并且您需要确保一次只调用一个时，该怎么办？例如，您可能正在调用一个远程 API，它需要一个‘nonce’(不仅仅是恋童癖者的俚语，也是计算机科学术语，表示一个在 API 调用之间必须递增的数字。去想想。)

您可以使用`reduce`来序列化`Promises`的数组，迫使它们一个接一个地运行。

从一个已经*解析的* `Promise`开始，您需要确保下一个`Promise`(第一个将是您的数组中的第一个)不会运行，直到前一个运行:

```
const inSequence = (previous, next) => previous.then(() => next);export default async function serialise(promises) {
  return promises.reduce(inSequence, Promise.resolve())
}
```

如果你有一个`boolean`值的数组，你想累加它们`and`呢？《T4》在这方面也很棒。

```
function absoluteTruth(truths) {
  return truths.reduce((acc, truth) => acc && truth, true)
}
```

## 链接

*   `Array.map`—[https://developer . Mozilla . org/en-US/docs/Web/JavaScript/Reference/Global _ Objects/Array/map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
*   `Array.reduce`—[https://developer . Mozilla . org/en-US/docs/Web/JavaScript/Reference/Global _ Objects/Array/Reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)
*   朱利叶斯·萨姆纳·米勒—[https://en.wikipedia.org/wiki/Julius_Sumner_Miller](https://en.wikipedia.org/wiki/Julius_Sumner_Miller)
*   拉斯伯里酱—[https://www . all recipes . com/recipe/241308/fresh-raspberry-Sauce/](https://www.allrecipes.com/recipe/241308/fresh-raspberry-sauce/)

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。