# 用 WebdriverIO & WebdriverCSS 进行可视化回归测试

> 原文：<https://itnext.io/visual-regression-test-with-webdriverio-webdrivercss-d7675a1812b2?source=collection_archive---------2----------------------->

![](img/d882cf7bf6401824026b2e72b1cb81b8.png)

*视觉回归测试结果*

**TLTR** :它会捕捉图像作为第一次运行的基线，然后与下一次运行的图像进行比较！就是这样。

使用 Spec Reporter 将结果显示在您的日志中，这样您就可以知道您的测试是否失败。

**要求**

运行[可视化回归测试](https://github.com/mojoaxel/awesome-regression-testing)的工具太多了，我总是更喜欢免费的。在这篇文章中，我将向你展示如何用 [WebdriverIO v4](http://v4.webdriver.io) (它还不能与 v5 一起使用) [WebdriverCSS](https://github.com/visualregressiontesting/webdrivercss) 、 [Chromedriver](http://chromedriver.chromium.org/) 和 [GraphicsMagick](http://www.graphicsmagick.org/) 进行可视化回归测试。

在开始编写测试代码之前，您应该阅读关于如何安装 GraphicsMagick 的文档。

**入门**

为您的项目准备 package.json

```
// package.json{
  "name": "visual-regression-test",
  "version": "1.0.0",
  "description": "Visual Regression Test",
  "main": "index.js",
  "scripts": {
    "clean": "find . -name '*.diff.png' -delete",
    "test": "npm run clean && wdio",
    "dev": "wdio --spec"
  },
  "keywords": [
    "webdriverio",
    "webdrivercss",
    "visual-regression-testing",
    "regression-testing"
  ],
  "author": "Dale Nguyen",
  "license": "ISC",
  "dependencies": {
    "webdrivercss": "github:visualregressiontesting/webdrivercss",
    "webdriverio": "^4.14.4"
  },
  "devDependencies": {
    "wdio-mocha-framework": "^0.6.4",
    "wdio-selenium-standalone-service": "0.0.12",
    "wdio-spec-reporter": "^0.1.5"
  }
}
```

之后，你只需要安装所有的软件包。

```
npm install
```

**准备您的 WebdriverIO 配置文件**

```
./node_modules/.bin/wdio config
```

然后回答一些问题。

编写你的第一个测试

```
// test/home.page.spec.jsconst assert = require('assert');
const wdio = require('webdriverio');
var getDiffFiles = require('../utils/get_diff_files');let client;const options = {
    url: '[https://learn.visualregressiontesting.com/'](https://learn.visualregressiontesting.com/'),
    // url: '[https://learn.visualregressiontesting.com/broke.html'](https://learn.visualregressiontesting.com/broke.html'),
    screenshotRoot: 'screenshots/page1',
    failedComparisonsRoot: 'screenshots/page1/diffs'
}describe('my website should always look the same',function() {
    beforeEach('init webdrivercss', () => {
        client = wdio.remote({
            desiredCapabilities: {
                browserName: 'chrome',
                chromeOptions: {
                    args: [
                        '--headless', 
                        '--disable-gpu',
                        'window-size=1920,1080'
                    ]
                }        
            }
        }).init();

        require('webdrivercss').init(client, {
            // example options
            screenshotRoot: options.screenshotRoot,
            failedComparisonsRoot: options.failedComparisonsRoot,
            // misMatchTolerance: 0.05,
            // screenWidth: [320,480,640,1024]
        });
    })it('homepage should look the same', async (done) => {
        await client.url(options.url)
            .webdrivercss('homepage', [
                {
                    name: 'header',
                    elem: '.header'
                },
                {
                    name: 'benefits',
                    elem: '.benefits',
                    screenWidth: [320, 640, 1024]
                }
            ])
            .end()            
            .call(done); // test diff folder
        const files = await getDiffFiles(options.failedComparisonsRoot);        
        // console.table({files})
        assert.equal(files.length, 0, `There are changes on the site! Please check in ${options.failedComparisonsRoot} folder.`)
    });
})
```

测试将在 Chrome headless 模式下运行。第一次运行时，您将使用[https://learn.visualregressiontesting.com/](https://learn.visualregressiontesting.com/')站点创建基础映像。下一次，你可以用[https://learn.visualregressiontesting.com/broke.html](https://learn.visualregressiontesting.com/broke.html')，来展示两个站点的区别。如有差异，将保存在**截图/**/diffs** 文件夹下。

测试将检查 **diffs** 文件夹下是否有文件，并分割出结果。

这是等待 **diffs** 文件夹下文件的帮助函数。

```
// utils/get_diff_files.jsconst fs = require('fs');const getDiffFiles = (path) => {
    return new Promise((resolve, reject) => {
        fs.readdir(path, function(err, items) { 
            var temp = [];
            for (var i=0; i<items.length; i++) {
                if(items[i].includes('.png')) {
                    temp.push(items[i]);
                }                
            }
            resolve(temp);
        });
    })
}module.exports = getDiffFiles
```

就是这样。你只需要跑

```
npm test
```

以便显示在您更新或更改您的网站后是否有任何差异。

这里是 [Git 库](https://github.com/dalenguyen/visual-regression-testing)，以防你想检查工作代码。

[https://github.com/dalenguyen/visual-regression-testing](https://github.com/dalenguyen/visual-regression-testing)