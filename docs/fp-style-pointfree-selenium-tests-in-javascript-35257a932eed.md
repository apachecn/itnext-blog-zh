# JavaScript 中的 FP 风格的无指针硒测试

> 原文：<https://itnext.io/fp-style-pointfree-selenium-tests-in-javascript-35257a932eed?source=collection_archive---------6----------------------->

![](img/5f70cc8344edd85057d27bb1957e966d.png)

selenium-webdriver

这篇文章是一个以自由风格编写 [Selenium](https://www.seleniumhq.org/) 测试的教程。这应该会减少你的用户体验测试的规模，因为以一种无点数的方式编写它们可以消除大多数变量和“等待”的测试代码(aysnc/await)。这里有一个例子来说明我的意思。

```
test('User can login', () => S.seq(
  S.call('get', ROOT_URL),
  S.click('[data-go="login"]'),
  S.fillForm(forms.login, user.credentials),
  S.click('button[type="submit"]'),
)(opts))
```

![](img/eabb04d34c02e93b85b914cf83f75dda.png)

[Ashim D'Silva](https://unsplash.com/@randomlies?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的“按钮打印帖子”

## Selenium-webdriver 和 Node.js

我使用 [Selenium-Webdriver](https://seleniumhq.github.io/selenium/docs/api/javascript/index.html) 和 [Jest](https://jestjs.io/) 来编写自动化浏览器测试。我不会详细讨论这个问题，但是如果你搜索“selenium-webdriver jest”或者“selenium-webdriver mocha ”,你会发现一些很好的博客文章，它们更详细地描述了如何将两者结合使用。一般来说，Jest 或 [Mocha](https://mochajs.org/) 是测试运行程序，selenium-webdriver 在测试中用于与浏览器交互。下面的代码摘自[其中一篇博文](https://medium.com/@andrey.dobra/web-testing-using-selenium-webdriver-part-4-adding-javascript-node-js-and-mocha-656df2a12787)。

```
test('01 Drums Access Await', async function () {
  await driver.get("https://andreidbr.github.io/JS30/");
  const drumsLink = await
    driver.findElement(By.xpath('/html/body/div[2]/div[1]'));
  await drumsLink.click();
  const pageTitle = await driver.getTitle();
  await assert.equal(pageTitle, "JS30: 01 Drums");
}
```

## 自由点样式和函数组合

函数组合的工作原理是将一系列函数调用组合成一个函数。还有像 [Ramda](https://ramdajs.com/) 这样的实用程序库，包含组合函数的帮助器。这些助手通常被称为合成或管道/序列。

```
const log => txt => () => console.log(txt)
const name = { first: 'Bob', last: 'Cobb' }// Composed functions run last to first
let logName = compose(log(name.first), log(', '), log(name.last))
logName() // prints "Cobb, Bob"// Piped or sequenced functions run first to last
logName = pipe(log(name.first), log(' '), log(name.last))
logName() // prints "Bob Cobb"
```

Pointfree 是一种可以通过函数组合实现的编码风格。上面的例子没有显示出来，但是组合函数接收前一个函数的结果作为唯一的参数。只有第一个运行的函数可以接收多个参数。

```
const add = x => y => x + y
const add7 = compose(add(2), add(5))const value = add7(10)
console.log(value) // prints "17"
```

上面，第一个运行的函数是“add(5)”。它接收 10 作为参数，结果是 15。然后，“add(2)”接收 15 作为它的参数，最终结果是 17。它被称为 pointfree，因为我们从来不需要在自己的代码中把 15 存储在一个变量中。

## 异步序列

现在，我们如何构造异步函数呢？有一些库可以完成这项工作，但是如果您使用的是 node 版本 8 或更高版本，完成这项工作的代码很简单。

```
const seq = (...fns) => async function seq (...args) {
  if (!fns.length) return
  let result = await fns[0](...args)
  for (let i = 1; i < fns.length; i++) {
    result = await fns[i](result)
  }
  return result
}
```

除了传递结果，我们还可以调整这个“seq”函数，这样就可以在函数之间传递上下文对象。这将使每个函数能够访问配置设置和状态，以及先前函数的结果。

```
const seq = (...fns) => async function seq (ctx) {
  for (let i = 0; i < fns.length; i++) {
    ctx = Object.assign(ctx, await fns[i](ctx))
  }
  return ctx
}
```

## 与硒一起使用

最后一步是开始将 Selenium-Webdriver API 封装到可以插入异步管道的函数中。让我们将这些助手函数分组到一个名为“selenium-webdriver-fp”的实用程序库中。下面是使用我们的 fp helper 库从上面重写代码的样子。

```
const S = require('selenium-webdriver-fp')test('01 Drums Access Await', () => S.seq(
  S.call('get', 'https://andreidbr.github.io/JS30/'),
  S.click(By.xpath('/html/body/div[2]/div[1]')),
  S.call('getTitle'),
  ctx => { assert.equal(ctx.callResult, 'JS30: 01 Drums') }
)({ driver }))
```

关于助手函数，需要注意以下几点:

*   组合函数是 S.seq 的结果，必须向其传递一个具有“driver”属性的对象，即 selenium-webdriver 实例。
*   运行时，所有函数都以上下文作为唯一参数来调用
*   配置助手时，所有参数都可以是值或函数。如果是函数，它接收上下文并返回值。
*   如果选择器是一个字符串，则假定它是一个 CSS 选择器。
*   上下文有一些关键字。“callResult”是最后一次调用“S.call”的值，“element”和“selector”是最后一次调用“S.getElement”的值。

## Selenium-webdriver-fp

[https://gist . github . com/rhythnic/FB 6d 4 f 0 F5 EC 0 d9 c 1266 e 827 f 145 a 62 f 6](https://gist.github.com/rhythnic/fb6d4f0f5ec0d9c1266e827f145a62f6)