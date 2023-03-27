# 等待，不要让你的 E2E 测试睡眠

> 原文：<https://itnext.io/await-do-not-sleep-your-e2e-tests-df67e051b409?source=collection_archive---------3----------------------->

睡眠:E2E 测试中最糟糕的实践和确定性事件的概念。

![](img/5a788bc6b233d54d1afca907f762ecd1.png)

[乔·罗伯茨](https://unsplash.com/photos/7aGbdAhEbKg)在 [Unsplash](https://unsplash.com) 上拍照

我正在 GitHub 上做一个大的 [UI 测试最佳实践](https://github.com/NoriSte/ui-testing-best-practices)项目，我分享这个帖子来传播它，并对其中一章有直接的反馈。

当测试你的用户界面时，你定义了一个应用程序必须通过的关键点。达到这些关键点是一个异步的过程，因为几乎 100%的时候，你的 UI 不会同步更新。那些关键点被称为**确定性事件**，被认为是你知道必须发生的事情。
这取决于你定义的事件和你的用户界面到达它们的方式，但是通常会有一些“长时间”的等待，比如 XHR 请求，以及一些更快的等待，比如重新渲染更新。

异步更新的解决方案似乎很方便:**休眠/暂停测试**几毫秒，几十分之一秒，甚至几秒。它可以让你的测试工作，因为它给应用程序时间来更新自己，并移动到下一个确定性事件进行测试。

考虑一下，除了特定的和已知的等待(比如当你使用 *setInterval* 或 *setTimeout* )之外，**睡眠时间应该有多长是完全不可预测的**，因为它可能取决于:
-网络状态**(对于 XHR 请求)
-可用的机器资源总量**(CPU、RAM 等)。)
—CI 管道可以限制它们，例如
—其他应用程序也可以在您的本地机器上使用它们
—其他资源消耗更新(canvas 等)的**并发**。)
-像**服务人员、缓存**管理等一堆捉摸不定的游戏玩家。这可以使 UI 更新过程更快或更慢****

****每一个固定的延迟都会导致你的测试更加脆弱，并且**增加它的持续时间**。你将在假阴性(由于睡眠时间太短而导致测试失败)和夸大测试持续时间之间找到平衡。等待适当的时间怎么样？使测试尽可能快的时间量！****

****等待分为四个主要类别:
- **页面加载等待**:第一个等待管理在测试你的应用时， 等待一个让你明白页面是交互的事件
- **内容等待:**等待与选择器匹配的 DOM 元素
- **XHR 请求等待**:等待 XHR 请求开始或收到相应的响应
- **自定义等待**:等待与不属于上述类别的 app 严格相关的一切
每个 UI 测试工具都以不同的方式管理等待，有时自动，有时手动
。 下面你可以找到一些
实现所列等待的例子。****

# ****页面加载等待****

****每个 E2E 测试工具都以不同的方式管理页面加载等待(在考虑加载页面之前等待什么)。****

******柏树******

```
**cy.visit('http://localhost:3000')**
```

******木偶师(和剧作家)******

```
**await page.goto('http://localhost:3000')**
```

******硒******

```
**driver.get('http://localhost:3000')
driver.wait(*function*() {
  return driver
    .executeScript('return document.readyState')
    .then(*function*(*readyState*) {
      return readyState === 'complete'
    })
})**
```

******测试咖啡馆******

```
**fixture`Page load`.page`http://localhost:3000`**
```

# ****内容等待****

****看看下面的例子，看看如何在可用的工具中实现对 DOM 元素的等待。****

******柏树******

*   ****等待元素:****

```
**// it waits up to 4 seconds by default
cy.get('#form-feedback')// the timeout can be customized
cy.get('#form-feedback', {timeout: 5000})**
```

*   ****等待具有特定内容的元素:****

```
**cy.get('#form-feedback').contains('Success')**
```

******木偶师(和剧作家)******

*   ****等待元素:****

```
**// it waits up to 30 seconds by default
await page.waitForSelector('#form-feedback')// the timeout can be customized
await page.waitForSelector('#form-feedback', {timeout: 5000})**
```

*   ****等待具有特定内容的元素:****

```
**await page.waitForFunction(*selector* *=>* {
 *const* el = document.querySelector(selector)
  return el && el.innerText === 'Success'
}, {}, '#form-feedback')**
```

******硒******

*   ****等待元素:****

```
**driver.wait(until.elementLocated(By.id('#form-feedback')), 4000)**
```

*   ****等待具有特定内容的元素:****

```
***const* el = driver.wait(until.elementLocated(By.id('#form-feedback')), 4000)
wait.until(ExpectedConditions.textToBePresentInElement(el, 'Success'))**
```

******测试咖啡馆******

*   ****等待元素:****

```
**// it waits up to 10 seconds by default
await Selector('#form-feedback')// the timeout can be customized
await Selector('#form-feedback').with({timeout: 4000})**
```

*   ****等待具有特定内容的元素:****

```
***await* Selector('#form-feedback').withText('Success')**
```

******DOM 测试库******

*   ****等待元素:****

```
**await findByTestId(document.body, 'form-feedback')**
```

*   ****等待具有特定内容的元素:****

```
***const* container = await findByTestId(document.body, 'form-feedback');
await findByText(container, 'Success')**
```

# ****XHR 请求等待****

****即使这是非常重要的一点，在撰写本文时(2019 年 7 月)，等待 XHR 的请求和响应似乎并不常见。****

****除了 Cypress 和 Puppeteer 之外，其他工具/框架强迫您在 DOM 中寻找反映 XHR 结果的东西，而不是寻找 XHR 请求本身。****

****您可以阅读更多关于该主题的内容:
Selenium WebDriver:[在 Selenium web driver 中测试 AJAX 调用的 5 种方法](https://www.blazemeter.com/blog/five-ways-to-test-ajax-calls-with-selenium-webdriver)
Test café:[XHR 和获取请求的等待机制](https://devexpress.github.io/testcafe/documentation/test-api/built-in-waiting-mechanisms.html#wait-mechanism-for-xhr-and-fetch-requests)
DOM 测试库: [await API](https://testing-library.com/docs/dom-testing-library/api-async#wait)****

******柏树******

*   ****等待 XHR 请求/响应****

```
**cy.intercept('http://dummy.restapiexample.com/api/v1/employees').as('employees')
cy.wait('@employees')
  .its('response.body')
  .then(*body* *=>* {
    /* ... */
  })**
```

******木偶师(和剧作家)******

*   ****等待 XHR 的请求:****

```
**await page.waitForRequest('http://dummy.restapiexample.com/api/v1/employees')**
```

*   ****等待 XHR 的回应:****

```
***const* response = await page.waitForResponse('http://dummy.restapiexample.com/api/v1/employees')
*const* body = response.json()**
```

# ****定制等待****

****各种 UI 测试工具/框架都有内置的解决方案来执行大量的检查，但是让我们专注于编写一个定制的等待。由于 UI 测试是 100%异步的，自定义等待应该面对递归承诺，这个概念在开始时并不容易管理。****

****幸运的是，有一些方便的解决方案和插件来帮助我们。考虑一下，如果我们想等到一个全局变量( *foo* )被赋予一个特定的值( *bar* ):下面你会找到一些例子。****

******柏树**
多亏了[柏树——等待插件](https://github.com/NoriSte/cypress-wait-until)你才能做到:****

```
**cy.waitUntil(() *=>* cy.window().then(*win* *=>* win.foo === 'bar'))**
```

******木偶师(和剧作家)******

```
**await page.waitForFunction('window.foo === "bar"')**
```

******硒******

```
**browser.executeAsyncScript(`
  window.setTimeout(function(){
    if(window.foo === "bar") {
      arguments[arguments.length - 1]();
    }
  }, 300);
`)**
```

******测试咖啡馆******

```
**const waiting = ClientFunction(() => window.foo === 'bar')
await t.expect(waiting()).ok({ timeout: 5000 })**
```

******DOM 测试库******

```
**await wait(() *=>* global.foo === 'bar')**
```

****一些注意事项:****

*   ****不像赛普拉斯，木偶师等。DOM 测试库是一个完全不同的工具，这就是为什么不是每个部分都有例子的原因。****
*   ****如果有更好的解决方案或插件来做同样的事情，请让我知道！我非常了解 Cypress、Puppeteer 和 DOM Testing Library，但是我不能说 Selenium 和 TestCafé也是如此。****

****你好👋我是 Stefano Magni，我是一名热情的 JavaScript 开发人员和 T21 赛普拉斯大使。我喜欢创造高质量的产品，测试和自动化一切，学习和分享我的知识，帮助别人，在会议上发言和面对新的挑战。我在意大利比特币初创公司 Conio 工作。
你可以在 [Twitter](https://twitter.com/NoriSte) 、 [GitHub](https://github.com/NoriSte) 、 [LinkedIn](https://www.linkedin.com/in/noriste/) 上找到我。你可以找到我最近所有的文章/演讲等等。这里。****