# 使用 Selenium 和 JavaScript 进行自动化 UI 测试

> 原文：<https://itnext.io/automated-ui-testing-with-selenium-and-javascript-90bbe7ca13a3?source=collection_archive---------0----------------------->

![](img/e29ff17d68fb9cab870a659a3ad0076b.png)

并不是所有的事情都可以用自动化来完成，但是如果你能自动化那些重复无聊的任务或者测试，那就太好了。

[Selenium](https://www.seleniumhq.org/) 非常棒，它可以与许多不同的语言集成，今天我将向您展示如何用 JavaScript 实现它。这是一个正在进行的项目，我计划在未来更新更多的例子或测试用例，所以请随时提出问题，并在项目上创建票证。

[](https://github.com/dalenguyen/selenium-javascript) [## dalenguyen/selenium-javascript

### 用 Selenium 和 JavaScript 进行自动化 UI 测试

github.com](https://github.com/dalenguyen/selenium-javascript) 

1.  **安装**

你需要在你的机器上安装[纱线](https://yarnpkg.com/en/)或[节点](https://nodejs.org/en/)。在这个项目中，我将使用 Yarn 来安装软件包和运行脚本。

首先，我们需要开始一个项目

```
yarn init 
OR
npm init
```

然后安装依赖项

```
yarn add chromedriver selenium-webdriveryarn add mocha -D
```

2.**入门**

在那之后，我们可以开始编写我们的测试用例

```
// test.js// Import requirement packages
require('chromedriver');
const assert = require('assert');
const {Builder, Key, By, until} = require('selenium-webdriver');describe('Checkout Google.com', function () {
    let driver; before(async function() {
        driver = await new Builder().forBrowser('chrome').build();
    }); // Next, we will write steps for our test. 
    // For the element ID, you can find it by open the browser inspect feature. it('Search on Google', async function() {
        // Load the page
        await driver.get('[https://google.com'](https://google.com'));
        // Find the search box by id
        await driver.findElement(By.id('lst-ib')).click();
        // Enter keywords and click enter
        await driver.findElement(By.id('lst-ib')).sendKeys('dalenguyen', Key.RETURN);
        // Wait for the results box by id
        await driver.wait(until.elementLocated(By.id('rcnt')), 10000); // We will get the title value and test it
        let title = await driver.getTitle();
        assert.equal(title, 'dalenguyen - Google Search');
    }); // close the browser after running tests
    after(() => driver && driver.quit());
})
```

3.**运行测试用例**

向 package.json 添加脚本是一个很好的做法。

```
// package.json
"scripts": {
   "test": "mocha --recursive --timeout 10000 test.js"
},
```

每当您想要运行测试时，只需使用以下命令:

```
yarn run test
```

以下结果表明测试已成功完成。

```
dnguyen$ yarn run test
yarn run v1.9.4
$ mocha --recursive --timeout 10000 test.jsCheckout Google.com
    ✓ Search on Google (2058ms)1 passing (3s)✨  Done in 3.79s.
```

希望这有所帮助。请随意提问并[创建一张票](https://github.com/dalenguyen/selenium-javascript/issues/new)来改进这个项目；)

[**在 Twitter 上关注我**](https://twitter.com/dale_nguyen) 了解 Angular、JavaScript & WebDevelopment 的最新内容👐