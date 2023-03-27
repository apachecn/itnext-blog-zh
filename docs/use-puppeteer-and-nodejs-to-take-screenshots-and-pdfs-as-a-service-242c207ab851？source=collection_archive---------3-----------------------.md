# 使用 Puppeteer 和 Nodejs 来截图和 pdf——作为一项服务

> 原文：<https://itnext.io/use-puppeteer-and-nodejs-to-take-screenshots-and-pdfs-as-a-service-242c207ab851?source=collection_archive---------3----------------------->

## 你有没有想过一个简单的解决方案，以编程方式创建截图？

![](img/58ab924b4352f8ee699dd64a1c386cd9.png)

在 [Unsplash](https://unsplash.com/s/photos/printing?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上[银行 Phrom](https://unsplash.com/@bank_phrom?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄的照片

作为一名开发人员，我想创建一个简单的服务，创建动态网站，并通过截图和 PDF 轻松共享。我本可以创建一个简单的 NodeJS 应用程序([，就像本文](https://thenextweb.com/syndication/2021/01/10/how-to-turn-web-pages-into-pdfs-with-puppeteer-and-nodejs/)中描述的那样)来一次性解决问题，但是我想:“为什么不把它变成一个简单的服务呢？”。

基础部分——一个简单的 NodeJS 应用程序生成一个由木偶师控制的无头浏览器来从网站上捕捉屏幕截图和图像——已经就绪，但现在我想让它更加通用和易于使用。

## 什么是多一点？

*   一个简单明了的 API
*   参数验证
*   证明文件
*   测试
*   安全考虑
*   Docker 图像
*   简单部署到例如 Heroku

## API

我决定用 ExpressJS 生成一个简单的 NodeJS 服务。很简单，由两个端点组成: **/api/shot** 和 **/api/pdf** 。

我使用 [apidoc](https://www.npmjs.com/package/apidoc) 在我的代码内创建文档，一旦我构建了文档，它就可以在 **/docs** ( [预览](https://the-snap.herokuapp.com/docs/))下获得。

```
//...
/**
 * @api {get} /pdf obtain a pdf of a page
 * @apiName TakeScreenshot
 * @apiGroup PDF
 * @apiVersion 1.0.0
 *
 * @apiParam {String} url a url to be looked up
 * @apiParam {Number} [w] width of the viewport
 * @apiParam {Number} [h] height of the viewport
 * @apiParam {String} [d] device to use for the viewport - overwrites other v/h parameters
 *
 * @apiSuccess {File} pdf the generated pdf
 * @apiError {Object} Errors returned errors
 */
//...
```

*文件*

除了文档之外，我决定使用 [ajv](https://www.npmjs.com/package/ajv) 进行参数验证，因为它非常快并且维护良好。模式位于 schema.js 中，非常简单。

```
const { devices } = require('./util');

const screenshotSchema = {
  title: 'Page screenshot',
  description: 'Take screenshot of a page',
  type: 'object',
  properties: {
    url: {
      type: 'string',
      format: 'uri',
      pattern: '^(https?|http?)://',
      minLength: 1,
      maxLength: 255,
    },
    selector: {
      type: 'string',
      minLength: 1,
      maxLength: 255,
    },
  },
  required: ['url'],
};

const pdfSchema = {
  title: 'PDF page print',
  description: 'Take pdf of a page',
  type: 'object',
  properties: {
    url: {
      type: 'string',
      format: 'uri',
      pattern: '^(https?|http?)://',
      minLength: 1,
      maxLength: 255,
    },
    w: {
      type: 'integer',
      minimum: 1,
      maximum: 12288,
    },
    h: {
      type: 'integer',
      minimum: 1,
      maximum: 12288,
    },
    d: {
      type: 'string',
      enum: Object.keys(devices),
    },
  },
  required: ['url'],
};

module.exports = { screenshotSchema, pdfSchema };
```

现在，我只需要确保用每个端点的模式验证我的请求参数。

```
//...
router.get('/shot', async (req, res) => {
  const validate = ajv.compile(screenshotSchema);
  const result = await validate(req.query);
  if (!result) {
    const errors = await parseAJVErrors(validate.errors);
    // return with the validation errors
    return res.status(400).json({ errors });
  }
  const { url, selector } = req.query;
//use the params now
```

*参数验证*

最后，我们希望有一个简单的 API——只有两个端点回复 GET 请求。一份截图可在 */api/shot* 获取，另一份 pdf 可在 */api/pdf* 获取。当我们处理 GET 请求时，我们只支持查询参数。

*简单的应用编程接口*

# 测试

这是一个关键的区别，不仅要将项目发布为开源项目，而且还要有一组测试，其他人可以通过运行测试来验证功能。

我决定在这个项目中尝试一下[](https://github.com/avajs/ava)**，我不得不承认，它为你的主要功能提供了一个真实的体验。**

```
const test = require('ava');
const { shot: screenshot, pdf } = require('./capture');
const fs = require('fs');

test('create screenshots', async (t) => {
  const screenshotBuffer = await screenshot({ url: 'https://www.google.com' });
  t.assert(
    screenshotBuffer.toString('binary').length > 1,
    'Should have generated a screenshot',
  );
});
test('create screenshot by selector', async (t) => {
  const screenshotSelectorBuffer = await screenshot({
    url: 'https://card-joy.web.app/v?img=c1&t=Merry%20Christmas!&p=topLeft',
    selector: '#root > div > div > div > img',
  });
  t.assert(
    screenshotSelectorBuffer.toString('binary').length > 1,
    'Should have generated a screenshot',
  );
});
test('create pdf', async (t) => {
  const generated = await pdf({ url: 'https://www.google.com' });
  t.assert(generated.length > 0, 'Should have generated a pdf');
});
```

**最后，我将 package.json 中的测试命令设置为*“test”:“ava—verbose”*—使用 verbose 来获取每个测试用例的更多信息。**

***测试***

# **安全性**

**由于这是一项服务，可以作为单一用途的微服务轻松地纳入现有项目，我决定使用 [**头盔**](https://www.npmjs.com/package/helmet) 来保护 Express 应用程序，该应用程序设置了各种 HTTP 头以减少常见的安全隐患。**

**最后但同样重要的是，我添加了速率限制，默认为每个 IP 每 15 分钟 20 个请求，并存储在内存中。**

***涵盖的常见安全隐患***

# **部署**

**这项服务可以很容易地用 docker 运行。该映像包包括 chrome 浏览器在内的所有依赖项。**

```
$ docker run -it --rm -p 3000:3000 chrkaatz/the-snap
```

**或者如果你已经有了一个 Heroku 帐号，你可以使用项目上的*部署到 Heroku* 按钮。**

**对于设置，我添加了包含以下内容的 *app.json* ,以使用 Nodejs 和一个 Buildpack 来支持在 Heroku 中使用 puppeteer。**

```
{
    "name": "The Snap",
    "description": "A simple service to obtain screenshots and PDFs from pages",
    "repository": "https://github.com/chrkaatz/the-snap",
    "logo": "https://github.com/chrkaatz/the-snap/raw/main/logo.png",
    "keywords": ["node", "puppeteer", "screenshot", "api", "pdf"],
    "buildpacks": [
        {
            "url": "heroku/nodejs"
        },
        {
          "url": "https://buildpack-registry.s3.amazonaws.com/buildpacks/jontewks/puppeteer.tgz"
        }
    ],
    "env": {
        "HUSKY": {
            "description": "Disable git hooks managed by husky",
            "value": "0"
        }
    }
  }
```

***部署***

# **密码**

**在 https://github.com/chrkaatz/the-snap 可以找到源代码，在 https://the-snap.herokuapp.com/[可以找到工作版本。](https://the-snap.herokuapp.com/)**

**最后说明:主页是用 [MVP.css](https://github.com/andybrewer/mvp) 做的，用起来超级简单。只花了 5 分钟…**