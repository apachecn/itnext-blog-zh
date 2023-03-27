# 如何在创建-反应-应用程序上设置木偶和笑话时不绝望

> 原文：<https://itnext.io/how-not-to-despair-while-setting-up-puppeteer-and-jest-on-a-create-react-app-plus-ci-on-travis-b25f387ee00f?source=collection_archive---------3----------------------->

似乎我没有一篇文章是以悲伤的故事开始的。下面是今天的悲惨故事:设置我的 create-react-app 进行端到端测试。

![](img/3b225d6ffaf20c6d3f57334233a2a0d8.png)

# 背景

我们的生态系统中有许多 React 客户端，但其中只有一个面向公众(除了登录屏幕):一个应用程序表单。现在，我们希望将其他一些东西公开，因此我们需要制作一系列其他应用程序(令人讨厌),或者清理应用程序，使其成为面向公众的内容的一个家园。

但阻碍我们的是，这份申请表首先是我写的，因为我学会了反应，然后是我们的新员工写的，因为她学会了反应。也就是说，代码是一个该死的噩梦，而且完全没有经过测试。

所以，第一步:建立测试框架，这样我们就可以像这个人一样进行一千次测试。第二步是重构，但那是另一天的悲惨故事。

# 设置

首先要做的是:依赖性。你需要`puppeteer` : `yarn add -dev puppeteer`。Create-react-app(从现在开始我只说 CRA)预装了 jest 的东西，所以不要安装，即使教程这么说。

现在设置您的测试在`package.json`中作为脚本运行，除非它们已经存在(我的曾经存在)。你在找这条线:

```
"test": "react-scripts test --env=jsdom",
```

我在网上到处寻找合适的东西放在这里。坚持你已经拥有的。

## 港口

我们有很多应用程序，所以它们都需要在不同的端口上运行。我的配置将我们的端口设置为 3030，因此我们在`scripts`中有这样一行:

```
"start": "PORT=3030 react-scripts start",
```

我不敢相信这就是所有需要的设置，但它是。

# 编写第一个测试

是的，标准的“不崩溃地渲染应用程序”测试对我来说失败了。我不知道为什么，如果你不知道，这是一个令人沮丧的过程。这对我来说已经足够好了，可以运行一个测试来加载应用程序。这是代码。在`App.test.js`:

```
import React from 'react';
import puppeteer from 'puppeteer';

describe('renders without crashing', () => {
  test(
    'we view the welcome h1 header',
    async () => {
      let browser = await puppeteer.*launch*({
        headless: true,
      });
      let page = await browser.newPage();

      await page.goto('http://localhost:3030');
      await page.waitForSelector('.welcome-message');

      const header = await page.$eval('h1', e => e.innerHTML);
      expect(header).toEqual('Awesome that you want to apply!');

      browser.close();
    },
    16000
  );
});
```

这真的很贴心。我们可以使用所有这些`async / await`函数，这样我们就可以等待无头浏览器来定位页面中我们关心的点。在这一点上，我只是测试我们的头已经加载了一个欢迎我们的申请人的消息。

(顺便说一下，将`headless`切换到`false`，浏览器将实际打开，这样您就可以看到您的测试正在运行。)

你也可以设置浏览器的大小，但这不是必须的。你可以在这里查看[木偶师文档。](https://github.com/yomete/react-puppeteer)

现在开始吧。如果你和我一样，你已经运行了你的本地服务器，并且测试通过了，你非常高兴！是时候开始编写大量的测试了。

# 和特拉维斯一起安排线人

别急，我的朋友们。事实证明，本地服务器对于运行无头浏览器至关重要，这是我在 Travis 上用`net::ERR_CONNECTION_REFUSED at http://localhost`观察一个又一个构建失败时发现的。这是我最后的`.travis.yml`配置:

```
language: node_js
node_js:
  - "9"
dist: trusty
sudo: required
addons:
  chrome: stable
  hostname: localhost
before_install:
  - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
before_script:
  - yarn start &
cache:
  yarn: true
  directories:
    - node_modules
install:
  - yarn install
script:
  - yarn test
```

需要的两件事不会立即突出:`sudo: required`否则您会得到其他一些奇怪的错误。还有那个`before_script`钩子。我向上帝发誓，我没有在网上看到任何人告诉我必须这么做。我意识到的唯一原因是因为我关闭了我的本地服务器，在本地得到了同样的错误。“等一下……”我说。我还了解到命令末尾的`&`意味着在后台运行它们。

现在，我的本地服务器正在 Travis 中愉快地运行，只是等待一些测试的到来，并好好利用它。

# 参考

[这个教程](https://blog.logrocket.com/end-to-end-testing-react-apps-with-puppeteer-and-jest-ce2f414b4fd7)，很抱歉的说是一个好的开始但是已经过时了，因为这篇文章毫无疑问基本上马上就会。