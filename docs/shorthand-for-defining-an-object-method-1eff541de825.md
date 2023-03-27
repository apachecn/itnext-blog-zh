# 定义对象方法的简写

> 原文：<https://itnext.io/shorthand-for-defining-an-object-method-1eff541de825?source=collection_archive---------2----------------------->

![](img/b76a26d4c1827443e2483963f99f6117.png)

几桶酒。图片由作者提供。

我从事 Javascript 编程已经很长时间了，所以我很惊讶地发现了一个我以前从未见过的语法，尽管它在 2015 年 9 月发布的 Node 4 中就已经存在了。

我所指的语法叫做[对象文字速记方法](https://node.green/#ES2015-syntax-object-literal-extensions-shorthand-methods)，它是同时推出的更广为人知的[对象文字速记属性](https://node.green/#ES2015-syntax-object-literal-extensions-shorthand-properties)特性的一个伴侣。

大多数 Javascript 程序员都知道可以这样做:

```
const hello = 'world'
const obj = { hello }console.log(obj.hello === 'world') // ==> true
console.log(Object.keys(obj))      // ==> ['hello']
console.log(typeof obj.hello)      // ==> 'string'
```

此外

```
const hello = name => {
  console.log('hello', name)
}
const obj = { hello }obj.hello('world')            // ==> 'hello world'
console.log(Object.keys(obj)) // ==> ['hello']
console.log(typeof obj.hello) // ==> 'function'
```

但是对我来说新的是，因为我不能完全解释的原因，因为它看起来很明显，你也可以这样做:

```
const obj = {
  hello(name) {
    console.log('hello', name)
  }
}obj.hello('world')            // ==> 'hello world'
console.log(Object.keys(obj)) // ==> ['hello']
console.log(typeof obj.hello) // ==> 'function'
```

这意味着您可以通过编写以下代码来定义一个简单的对象:

```
const obj = {
  title: 'My useful object',
  hello(name) {
    return `Hello ${name}.`
  }
}
```

例如，当您想要定义一个可读的`Stream`时，这是一个有用的简写，比如:

```
const { Readable } = require('stream')const inStream = new Readable({
  read(size) {
    // invoked when the stream is being read
  }
})
```

或者可写的`Stream`:

```
const { Writable } = require('stream')const outStream = new Writable({
  write(chunk, encoding, next) {
    console.log(chunk.toString(encoding))
    next()
  }
})
```

当然，您也可以通过以下方式轻松定义相同的`Stream`:

```
const { Writable } = require('stream')function write(chunk, encoding, next) {
  console.log(chunk.toString(encoding))
  next()
}const outStream = new Writable({ write })
```

但是这有点难看，尤其是当你设置`Stream`时，你的`write`函数只会被定义一次。

## 结论

这里的教训是，不管你使用一门编程语言有多长时间，这门语言总会有你从未见过的东西。一旦你看到它，你会奇怪你怎么可能不知道。

## 链接和相关文章

*   我在阅读[萨梅尔·布纳](https://medium.freecodecamp.org/@samerbuna)的优秀文章 [Node.js Streams:你需要知道的一切](https://medium.freecodecamp.org/node-js-streams-everything-you-need-to-know-c9141306be93)时偶然发现了这个语法。如果流让你困惑，那么他的文章可以帮助你解决这个问题。
*   [计算属性名 FTW！](https://codeburst.io/computed-property-names-ftw-61000332fff1) —计算属性名有多酷？
*   另见:[冒名顶替综合征](https://en.wikipedia.org/wiki/Impostor_syndrome)。是真的，每个人有时候都觉得自己像个傻逼。

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。