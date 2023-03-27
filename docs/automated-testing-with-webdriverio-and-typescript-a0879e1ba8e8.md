# 使用 WebdriverIO (v5)和 TypeScript 进行自动化和可访问性测试

> 原文：<https://itnext.io/automated-testing-with-webdriverio-and-typescript-a0879e1ba8e8?source=collection_archive---------4----------------------->

![](img/3e0bdfa8f9d07711bd025bf295e74b24.png)

使用 JavaScript 太棒了。利用 [TypeScript 在 WebdriverIO](https://webdriver.io/docs/boilerplate.html#dalenguyen-webdriverio-typescript-boilerplate-https-githubcom-dalenguyen-webdriverio-typescript-boilerplate) 中编写测试用例怎么样？

**为什么使用 TypeScript 而不是 JavaScript？**

我相信你只要谷歌一下就知道为什么了。简而言之，它将有助于编写更好的代码，静态类型，类，接口，像 Visual Studio 代码一样的实时 IDE 支持…如果你看到一些不使用“类型”的 TypeScript 项目，他们没有获得 TypeScript 的主要核心。

我在 Github 上构建了一个 [Webdriver TypeScript 样板文件，你可以马上克隆并将其用于你的项目。如果您有任何建议、问题或贡献，请随意创建票证；)](https://github.com/dalenguyen/WebdriverIO-TypeScript-Boilerplate)

[](https://github.com/dalenguyen/WebdriverIO-TypeScript-Boilerplate) [## dalen guyen/web driver io-TypeScript-Boilerplate

### 这个项目将向你展示如何用 WebdriverIO 和 TypeScript 启动你的 UI 自动化测试…

github.com](https://github.com/dalenguyen/WebdriverIO-TypeScript-Boilerplate) 

**特性**

*   用于管理测试用例的页面对象模型
*   测试用摩卡+柴
*   与[倾城](https://webdriver.io/docs/allure-reporter.html)的互动报道
*   使用 Axe-core 进行可访问性测试

**例题**

我使用模型来创建网站的界面。

```
// models/ts
export interface Test {
  alias: string;
  title: string;
}
```

对于页面对象模型

```
// pages/test.page.tsimport { expect } from "chai";
import { Test } from "./../models";
import { Element } from "webdriverio";class TestPage {
  private test: Test = null;
  private editBtn: string; constructor() {
    this.test = {
      alias: "/docs/api.html",
      title: "API Docs · WebdriverIO"
    };
    this.editBtn = ".edit-page-link";
  } getTestAlias(): string {
    return this.test.alias;
  } private getEditBtnEl(): Element<void> {
    console.log("Btn:", $(this.editBtn).getText());
    return $(this.editBtn);
  } assertPageTitle(): void {
    expect(browser.getTitle()).to.equal(this.test.title);
  } assertEditBtn(): void {
    const editBtn = this.getEditBtnEl();expect(editBtn.getAttribute("href")).to.be.equal("[https://github.com/webdriverio/webdriverio/edit/master/docs/API.md](https://github.com/webdriverio/webdriverio/edit/master/docs/API.md)");
  }
}const testPage = new TestPage();
Object.freeze(testPage);
export default testPage;
```

对于真正的测试用例

```
// test/test.page.spec.tsimport testPage from "./../pages/test.page";describe("webdriver.io API Docs page", () => {
  before(() => {
    browser.url(testPage.getTestAlias());
  });it("should have the right title", () => {
    testPage.assertPageTitle();
  });it("assert edit button", () => {
    testPage.assertEditBtn();
  });
});
```

**无障碍测试**

```
// This example follows WCAG 2.0 Level Aimport { source } from 'axe-core';
import { expect } from 'chai';// Advoid type checking for axe
declare const axe: any;describe('Axe test', () => {
 it('Should return results', () => {
  browser.url('/'); // inject the script
  browser.execute(source);

  // [https://github.com/dequelabs/axe-core/blob/develop/doc/API.md](https://github.com/dequelabs/axe-core/blob/develop/doc/API.md)
  const options = { runOnly: { type: 'tag', values: ['wcag2a'] } };
  // run inside browser and get results
  const results = browser.executeAsync((options, done) => {
   axe.run(document, options, function(err, results) {
    if (err) throw err;
    done(results);
   });
  }, options);

  expect(results.violations.length).to.be.equal(
   0,
   `${browser.getUrl()} doesn't pass Accessibility test`
  );
 });
});
```

如果有任何你不能用 TypeScript 实现的情况，请创建一个标签，我会尽我所能帮助你:)