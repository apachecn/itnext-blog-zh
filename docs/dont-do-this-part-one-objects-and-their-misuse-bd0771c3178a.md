# 不要做“这个”——第一部分

> 原文：<https://itnext.io/dont-do-this-part-one-objects-and-their-misuse-bd0771c3178a?source=collection_archive---------1----------------------->

![](img/dcbadd3849bb2e70e22f9bd47e4b4e99.png)

即使是最好的工具也会被误用。(图片由作者提供)

## 对象及其误用

# 背景

我在之前的一篇文章中指出，没有人应该使用 Typescript，我认为最好是详细说明我为什么这么认为。

这是由两部分组成的文章的第一部分。跳到[下一部分](https://medium.com/@davesag/dont-do-this-part-two-the-trouble-with-typescript-8ea9b26892e2)，在那里我表达了对 Typescript 的强烈意见。

## Javascript 是面向对象的语言吗？

是的，也不是。很多人，尤其是来自传统面向对象语言如 Java 或 SmallTalk 的人，看到 Javascript 对`class`和`this`关键字的支持，就认为他们能够并且应该将他们在 Java 中应用的原则应用到 Javascript 中。但是 Javascript 中的`class`不像 Java 类。它仅仅是语法上的糖衣，并没有给 Javascript 开发者提供与真正的面向对象语言中真正的`class`相同的保护。

在我深入探讨 Javascript 特殊的(有些人会说是奇特的)面向对象方法之前，有必要问问我们自己，面向对象是否值得追求。

TL；DR:我会说不，不多，尤其是在 Javascript 中。

# 物品的问题

用面向对象的话来说，一个对象是一段代码，*封装了*数据和作用于该数据的函数，并且可以用构造函数*实例化*。对象可以*从其他对象继承*数据和功能，形成具有可预测特性的对象层次结构。

## 纯面向对象示例(伪代码)

```
class Animal {
  legs: 4,
  walk() { /* do walking */ }
}class Cat extends Animal {
  purr() { / do purring */ }
}class Dog extends Animal {
  woof() { /* do woofing */ }
}class PetWalker {
  pets: [],
  addPet(pet) {
    this.pets.push(pet)
  }

  walkPets() {
    this.pets.forEach(walk)
  }
}class Application {
  static run() {
    const walker = new PetWalker()
    const mrFluffy = new Cat()
    const mrsFluffy = new Cat()
    const fido = new Dog()
    walker.addPet(mrFluffy)
    walker.addPet(mrsFluffy)
    walker.addPet(fido)
    walker.walkPets()
  }
}
```

那这张图有什么问题呢？

首先，如果这是一种纯粹的面向对象语言，那么创建 walker 的*实例*的代码也需要封装在一个更高级的对象中，直到你得到运行一切的`Application`对象。

另一方面，我不知道你有没有试过带一只猫去散步，但是…

猫不是特别适合走路

所以你聪明的 OO 程序员很可能会创建一个`Walkable`接口并重构上面的内容(也是用伪代码)

```
interface Walkable {
  walk()
}class Animal {
  legs: 4
}class Cat extends Animal {
  purr() { / do purring */ }
}class Dog extends Animal, is Walkable {
  walk() { /* do walking */ }
  woof() { /* do woofing */ }
}class WalkablePetWalker {
  pets: [],
  addPet(pet<Walkable>) {
    this.pets.push(pet)
  }

  walkPets() {
    this.pets.forEach(walk)
  }
}class Application {
  static run() {
    const walker = new PetWalker()
    const fido = new Dog()
    const mrFluffy = new Cat()
    try {
      walker.addPet(fido)
      walker.addPet(mrFluffy) // => boom throws error.
    } catch (err) {
      logger.error(err)
    } finally {
      walker.walkPets()
    }
  }
}
```

因为继承迫使你预先定义你的问题域的深层结构，它变得越复杂就越脆弱。

尝试将`Possum`添加到组合中。负鼠和猫一样，也不太愿意被人遛，但它们愿意。所以有理由说:

```
class Possum extends Cat {}
```

但这真的有意义吗？比方说，有些动物是可以被拍的，如果你试图拍一只不能被拍的动物，你就会被咬。

```
interface Pattable {
  pat()
}interface Purrs {
  purr()
}interface Woofer {
  woof()
}interface Bitey {
  bite()
}class Cat extends Animal is Pattable, Purrs {
  purr() { /* do purring */ } pat() {
    this.purr()
  }
}class Dog extends Animal is Walkable, Pattable, Woofs {
  woof() { /* do woofing */ }

  pat() {
    this.woof()
  }
}class Possum extends Animal is Bitey {
  bite() { /* do biting */ }

  pat() {
    this.bite()
  }
}
```

为了让这个有意义，我们需要`Possum`也是`Pattable`，这样外部对象就可以使用它的`pat`功能。另外，与`purr`和`woof`不同的是，`bite`需要一个目标。负鼠不仅会咬人，还会咬人。

所以，为了让`bite`函数知道谁在咬，我们需要修改`Pattable`接口来指定哪个`hand`在拍。

```
interface Pattable {
  pat(hand)
}class Cat extends Animal is Pattable, Purrs {
  purr() { /* do purring */ } pat(hand) {
    this.purr()
  }
}class Dog extends Animal is Walkable, Pattable, Woofs {
  woof() { /* do woofing */ }

  pat(hand) {
    this.woof()
  }
}class Possum extends Animal is Bitey {
  bite(hand) { /* do biting of hand */ }

  pat(hand) {
    this.bite(hand)
  }
}
```

那么负鼠的`bite(hand)`功能发生了什么？什么是`hand`？

```
interface Biteable {
  bittenBy(animal<Bitey>)
}
```

让我们把蛇也加入进来。`Snakes`明明是`Animals`，他们不是`Walkable`，而是`Pattable`，他们不是`purr`而是他们做`bite`。但是他们没有`legs`。

```
class Snake extends Animal is Pattable, Bitey {
  legs: 0 bite(hand<Bitable>) {
   hand.bittenBy(this)
  } pat(hand<Bitable>) {
    this.bite(hand)
  }
}
```

事情就是这样。

你原来的`Animals`类现在可能需要了解`bitable` `hands`和其他各种事情。

Erlang 的创造者 Joe Armstong 在 Peter Seibel 所著的《工作中的编码者》一书中有一句名言:

> “面向对象语言的问题是，它们拥有所有这些它们随身携带的隐含环境。你想要一个香蕉，但你得到的是一只大猩猩拿着香蕉和整个丛林”

## 宾语是名词，功能是动词

切换到一个晚餐聚会的类比，在一个 OO 系统中，你不能广播一个“传递黄油”的一般请求，你必须查找实现`Passable`的`butter`，可能通过某种`TableItemsManager`服务，然后调用它的`pass`函数，把`self`作为目标。太疯狂了。

宾语通常被认为是名词的类似物；真实的东西。而对象的功能，或者用面向对象的说法是*方法*，是作用于那些名词的动词。但是现实世界并没有如此紧密地联系在一起。

史蒂夫·耶格在他的文章《名词王国中的执行》中明确讨论了这一点。

> “Javaland 中的动词负责所有的工作，但因为它们受到所有人的蔑视，所以任何动词都不允许自由走动。如果一个动词要在公共场合出现，它必须始终由一个名词伴随。
> 
> 当然“护送”，作为一个动词本身，几乎不允许光着身子跑来跑去；必须获得马鞭草以方便护送。但是“促成”和“促成”呢？碰巧的是，促进者和拉皮条者都是相当重要的名词，他们的工作是分别通过促进和拉皮条伴随低级动词“促进”和“采购”。"

避免面向对象编程的原因有很多，但主要原因是*过度使用*对象，以及过度使用对象继承和多态(即使用接口对常见行为进行分组)。面向对象编程给了你一些惊人的工具来创建高度结构化，但非常脆弱的意大利面。没有人喜欢易碎的意大利面条。

# 面向对象和 Javascript

对象的一个关键特性是它们能够在内部引用自己。在 Javascript 中，这意味着`this`关键字。

关于关键字`this`的脆弱本质，已经有很多文章了。Javascript 开发新手在编写以下形式的回调函数时经常遇到困难:

```
function doSomething(data, callback) {
  try {
    const result = blahBlahSomethingWith(data)
    callback(null, result)
  } catch (err) {
    callback(err)
  }
}
```

然后在一个对象中调用它

```
class SomethingDoer {
  constructor(data) {
    this.data = data
    this.result = undefined
  }

  doThing() {
    doSomething(data, function(err, result) {
      if (!err) {
        this.result = result
      }
    })
  }
}
```

这是行不通的，因为回调中的`this`不是来自包含对象的`this`，而是回调本身。

您要么需要将回调`bind`到外部上下文，要么可以使用箭头函数:

```
doThing() {
  doSomething(data, (err, result) => {
    if (!err) {
      this.result = result
    }
  })
}
```

箭头功能使用外部上下文作为`this`。

但是回调混乱现在不是问题了，因为现在我们有了`async` / `await`没有人再使用回调了，除非他们被强迫。

尽管如此，类似这样的代码并不少见:

```
class Compressor {
  constructor(data, algorithm) {
    this.algorithm = algorithm
    this.data = data
    this.result = {}
  }

  compress() {
    Object.keys(this.data).forEach(function(key) {
      this.result[key] = this.algorithm(this.data[key])
    })
  }
}
```

当然，这和上面的原因是一样的。用箭头函数来修复它只能解决一部分问题:

```
compress() {
  Object.keys(this.data).forEach(key => {
    this.result[key] = this.algorithm(this.data[key])
  })
}
```

会工作，但它很丑。最好使用[curry](https://codeburst.io/currying-in-javascript-how-why-a0d66f1366b)并采取更实用的方法。

```
const compressor = algorithm => data =>
  Object.keys(data).reduce((acc, elem) => {
    acc[elem] = algorithm(data[elem])
    return acc
  }, {})
```

然后用它来代替写作:

```
const compressor = new Compressor(data, gzip)
compressor.compress()
console.log(compressor.result)
```

你可以写:

```
const compress = compressor(gzip)
console.log(compress(data))
```

在 Javascript 中，除非明确需要一个事物的多个实例，否则最好还是坚持使用函数式方法。

## 对象与对象

Javascript 支持两种形式的 OO 风格对象

```
class SomeThing { ... }
```

或者

```
function SomeThing() { ... }
```

这些本质上是一回事。

Javascript 还允许以下对象:

```
const someThing = {
  x: 'some x',
  y: 'some y'
}
```

哪个更像其他语言中的`struct`或`dictionary`。大多数 Javascript 开发人员使用一个术语“对象”来指代这两种类型，其含义来自上下文。

一般来说，如果你用`class`来定义它或者使用`new`关键字来创建它的一个实例，那么你就用大写字母来命名它。如果它只是一个容器，那么你就不能把它资本化。

```
class Thing {}
const thing = new Thing()const somethingElse = {
  yay: 'That makes sense'
}
```

# 题外话:什么是函数式编程？

函数式编程强调[不变性](https://en.wikipedia.org/wiki/Immutable_object)、[幂等性](https://en.wikipedia.org/wiki/Idempotence)和[确定性](https://www.simplethread.com/pure-and-deterministic-functions/)。它更喜欢 [*纯*函数](https://en.wikipedia.org/wiki/Pure_function)而不是有副作用或者结果不确定的函数，以及[函数组合](https://en.wikipedia.org/wiki/Function_composition)，又名 currying。函数式编程语言的一个关键特征是函数是“一级”语言元素，这意味着你可以传递它们并对它们赋值，就像它们只是另一个变量一样。

函数式编程植根于λ演算，这是一种数学抽象，为描述函数及其评估提供了理论框架。

## 纯函数

纯函数就是不影响函数外部任何东西的函数，对于相同的输入，它将总是返回相同的输出。

所以下面的`pusher`函数在改变数组的值时并不纯粹。

```
function pusher(array, data) {
  array.push(data)
}
```

一个纯粹的版本会像

```
function pusher(originalArray, data) {
  return [...originalArray, data]
}
```

在这种情况下，原始数组没有被修改，而是将其元素复制到结果中。

纯函数很容易推理，也很容易单独测试。

# 更多 Javascript OO 的怪异之处

这是我见过的一段真实的代码:

假设这个文件在`src/utils/Cache.js`中。

```
class Cache {
  constructor() {
    this.store = {}
  }

  put(key, value) {
    this.store[key] = value
  }

  take(key) {
    return this.store[key]
  }

  clear(key) {
    delete this.store[key]
  }

  reset() {
    this.store = {}
  }
}
```

然后在`src/utils/sharedCache.js`里。

```
const Cache = require('src/utils/Cache)module.exports = new Cache()
```

然后在`src/server.js`。

```
const { put } = require('./utils/sharedCache')
const makeServer = require('./utils/server/makeServer')function async start() {
  const server = makeServer()
  await server.start()
  put('server', server)
}
```

但是有一件奇怪的事情:这根本行不通。`put`函数中的`this`并不是您所期望的`Cache`对象的实例，而是您调用它的函数。

正如在 Javascript 的 this 关键字指南中所解释的:

> 当变量被定义为指向实例方法的函数时，该函数将丢失此

这是一个非常微妙的错误，我见过甚至是经验丰富的 Javascript 程序员，包括我自己，在很多情况下都会犯这个错误。

除非您需要一个以上的共享缓存，老实说，您为什么要这样做，那么最好只写:

在文件`src/utils/sharedCache.js`中

```
let storeconst put = (key, value) => {
  store[key] = value
}const take = key => store[key]const clear = key => {
  delete store[key]
}const reset = () => {
  store = {}
}reset()module.exports = { put, take, clear, reset }
```

在 Javascript 中，模块中的代码在模块第一次加载时就被执行。一旦加载，就不会再执行。您可以利用这个事实来避免使用`this`或`bind`。

## Javascript 中的对象比纯函数更难测试

假设你有一个函数，比如:

```
const Client = require('api-access-client')
const config = require('../config')
const mapUsers = require('../utils/mapUsers')const client = new Client(config)const getUsers = async function() {
  const client = new Client(config)
  const userData = await client.get()
  return userData.map(mapUsers)
}
```

为了测试这一点，用`mocha`和`proxyquire`模拟客户端，您需要创建一个模拟`Client`类。要用`Jest`模拟它，你需要写一个[手动模拟](https://jestjs.io/docs/en/manual-mocks.html)。不管怎样，这都是一堆额外的工作。如果`Client`只是一个漂亮的 curried 函数，那么做这些模拟将是微不足道的。

# 第一部分到此结束

转到第二部分,我在其中详述了我的面向对象体系结构，解释为什么没有人应该使用 Typescript。

## 我们了解到:

*   分层面向对象的软件设计只适用于非常有限的问题领域，过度使用面向对象会导致糟糕的软件架构
*   在 Javascript 中过度使用对象是一件可怕的事情
*   如果你以一种有纪律的方式接受 Javascript 的灵活的 T21 类型和功能方法，你将会写出更好的代码
*   你应该测试一切——至少是单元测试和集成测试，但是也要加入一些突变测试来测试你的测试

# 链接

## 文章

*   [工作中的编码员](http://codersatwork.com)
*   [名词王国里的处决](https://steve-yegge.blogspot.com/2006/03/execution-in-kingdom-of-nouns.html)
*   [干意大利面的脆断](https://www.sciencedirect.com/science/article/abs/pii/S1350630704000123)
*   [Javascript 中的 Currying 如何和为什么](https://codeburst.io/currying-in-javascript-how-why-a0d66f1366b)
*   [Javascript 的 this 关键字指南](https://medium.com/datadriveninvestor/a-potterheads-guide-to-javascript-s-this-keyword-7908399d0e93)
*   [狗屁 Javascript 编码员说](https://medium.com/@davesag/shit-javascript-coders-say-7a2d2881228d)

## 技术

*   [打字稿](https://www.typescriptlang.org)
*   [NodeJS](https://nodejs.org)
*   [摩卡](https://mochajs.org)
*   [代理查询](https://github.com/thlorenz/proxyquire)
*   [巴别塔](https://babeljs.io)
*   [网络包](https://webpack.js.org)
*   [笑话](https://jestjs.io)

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。