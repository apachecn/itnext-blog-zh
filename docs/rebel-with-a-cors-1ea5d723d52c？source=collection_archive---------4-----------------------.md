# 用 CORS 反叛

> 原文：<https://itnext.io/rebel-with-a-cors-1ea5d723d52c?source=collection_archive---------4----------------------->

## 如何从一个禁用了 CORS 的 API 中创建自己的简单的支持 CORS 的 API

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Frebel-with-a-cors-1ea5d723d52c)

作为一名前端开发人员，我在开发时经常使用各种第三方 API。这些 API 可以用于天气、加密货币价格或最新的 XKCD 漫画。

其中一些服务的问题是它们不支持跨来源请求(CORS)，这意味着客户端 AJAX 调用这些服务不起作用。这是令人沮丧的，但可以很容易地在你自己的微服务中的几行代码的帮助下修复。

# 创建微服务

创建一个简单的微服务只需要一个名为 [*micro*](https://github.com/zeit/micro) 的包。这是一个非常简单的包，支持创建异步微服务，如果您阅读了该项目的自述文件，您会发现只需几行代码就可以创建一个简单的服务:

```
module.exports = (req, res) => {
  res.end(‘Hello world’)
}
```

显然上面的完全没用，但是让我展示一下使用 micro 消费几乎任何 API 是多么容易，至少是任何不需要认证的 API。

![](img/ee830aa699e19a3e8362128d33da6950.png)

来自[漫画#1810](https://xkcd.com/1810/)

对于这个例子，我将使用来自 [XKCD 漫画](https://xkcd.com/)的免费 API，它不能用于客户端的 AJAX 调用，因为 CORS 被禁用了。调用网址[https://xkcd.com/info.0.json](https://xkcd.com/info.0.json)返回最新漫画，像这样:

```
{
  "month": "2",
  "num": 1954,
  "link": "",
  "year": "2018",
  "news": "",
  "safe_title": "Impostor Syndrome",
  "transcript": "",
  "alt": "It's actually worst in people who study the DunningâKruger effect. We tried to organize a conference on it, but the only people who would agree to give the keynote were random undergrads.",
  "img": "https://imgs.xkcd.com/comics/impostor_syndrome.png",
  "title": "Impostor Syndrome",
  "day": "12"
}
```

如果通过了正确的漫画 ID([https://xkcd.com/1500/info.0.json](https://xkcd.com/1500/info.0.json))，它可以返回一个特定的漫画:

```
{
  "month": "3",
  "num": 1500,
  "link": "",
  "year": "2015",
  "news": "",
  "safe_title": "Upside-Down Map",
  "transcript": "((A mercator projection of the world map is shown. All the continents have been rotated one hundred eighty degrees.))\n\n((Cuba  is next to alaska, and alaska is touching the tip of south america, which is all near the equator. Mexico is now friends with greenland.\n\n((Iceland, the UK, and asia are all close together. Japan and Taiwan haven't moved with the asian continent, and are technically European.))\n\n((Siberia is now equatorial. Africa is pretty temperate, except for the north bits which are somewhat antarctic.))\n\nCaption: This upside-down map will change your perspective on the world!\n\n{{Title text: Due to their proximity across the channel, there's long been tension between North Korea and the United Kingdom of Great Britain and Southern Ireland.}}",
  "alt": "Due to their proximity across the channel, there's long been tension between North Korea and the United Kingdom of Great Britain and Southern Ireland.",
  "img": "https://imgs.xkcd.com/comics/upside_down_map.png",
  "title": "Upside-Down Map",
  "day": "18"
}
```

微服务需要做的只是将任何请求传递给原始 API，并设置一些头以允许跨源请求，这样它们就可以在客户端的 AJAX 调用中使用，如下所示:

```
const axios = require('axios')
const { send } = require('micro')
const microCors = require('micro-cors')
const cors = microCors({ allowMethods: ['GET'] })
const DOMAIN = '[https://xkcd.com/'](https://xkcd.com/')const handler = async function(req, res) {
  const params = req.url
  const path = `${DOMAIN}${params}`
  const response = await axios(path)
  send(res, 200, response.data)
}module.exports = cors(handler)
```

那是 14 行代码！

上面的例子将任何 slug 信息传递给 API(例如`1000/0.json`)，因此调用`https://xkcd.now.sh/1000/0.json`(我的 API 版本)，将映射到`https://xkcd.com/1000/0.json`。这可能是我们旅程的终点，但我想通过改变端点来改进 API UX:

*   xkcd.now.sh 应该还最新漫画
*   【xkcd.now.sh/1000】应回漫画 ID 1000

请参见下文，了解如何实现这一目标:

```
const axios = require('axios')
const { send } = require('micro')
const microCors = require('micro-cors')
const cors = microCors({ allowMethods: ['GET'] })
const DOMAIN = 'https://xkcd.com/'
const PATH = 'info.0.json'const handler = async function(*req*, *res*) {
  let id = req.url.replace('/', '')
  const comicId = id ? `${id}/` : ''
  const path = `${DOMAIN}${comicId}${PATH}`
  const response = await axios(path)
  id = response.data.num
  let newResponse
  if (id >= 1084) {
    newResponse = {
        ...response.data,
        imgRetina: `${response.data.img.replace('.png', '')}_2x.png`,
      }
    } else {
      newResponse = {
      ...response.data,
    }
  }
  send(res, 200, newResponse)
}*module.exports* = cors(handler)
```

那是 29 行代码！看到了吗[这里](https://github.com/mrmartineau/xkcd-api/blob/master/index.js)👀

你可以在上面看到，除了 *micro* ，该服务还依赖另外两个包:

*   [*axios*](https://github.com/axios/axios) 为 HTTP 请求
*   [*微 cors*](https://github.com/possibilities/micro-cors) 是对微的简单 cors

我的 XKCD API 示例返回所有原始数据，实际上稍微改变了响应数据以及 API 的使用方式。我决定添加 retina 图像路径(如果有)并简化对 API 的调用。所以你可以调用`xkcd.now.sh/1894`而不是调用`xkcd.com/1894/info.0.json`。

因此，举例来说:调用[https://xkcd.now.sh/1894](https://xkcd.now.sh/1894)将从最初的 XKCD API:[https://xkcd.com/1894/info.0.json](https://xkcd.com/1894/info.0.json)请求这个 URL。

```
{
  "month": "9",
  "num": 1894,
  "link": "",
  "year": "2017",
  "news": "",
  "safe_title": "Real Estate",
  "transcript": "",
  "alt": "I tried converting the prices into pizzas, to put it in more familiar terms, and it just became a hard-to-think-about number of pizzas.",
  "img": "https://imgs.xkcd.com/comics/real_estate.png",
  "title": "Real Estate",
  "day": "25",
  "imgRetina": "https://imgs.xkcd.com/comics/real_estate_2x.png"
}
```

💪这项服务的代码托管在 https://github.com/mrmartineau/xkcd-api 的 GitHub 上，可以在 T21 使用 Postman 进行测试。

# 托管您的新 API

我现在使用[](https://zeit.co/now)*by[*zeit*](https://zeit.co)来托管我的各种 app 和 API。*现在*支持这个微服务需要的 JavaScript 语言特性(async/await)以及开箱即用的 HTTPS。如果你的主机不支持这些特性，你需要将代码转换回它支持的版本。*

*![](img/84a9f4d88fe7efbfd6678b5d284b1524.png)*

*来自[漫画#1700](https://xkcd.com/1700/)*

# *其他示例*

*作为一个更简单的传递 API 的例子，您可以看到我的 CORS 版本的插接板 feeds API。代码托管在 https://github.com/mrmartineau/pinboard-api[的 GitHub 上](https://github.com/mrmartineau/pinboard-api)*

*我感谢[安德鲁·威廉姆斯](https://medium.com/u/bc0058830195?source=post_page-----1ea5d723d52c--------------------------------)、[阿什利·诺兰](https://medium.com/u/f2a3cc045295?source=post_page-----1ea5d723d52c--------------------------------) & [恰兰·帕克](https://medium.com/u/a6cf81c6fa33?source=post_page-----1ea5d723d52c--------------------------------)对这篇文章标题的帮助。他们的其他建议包括:*

*   *不用担心 CORS:获取 API*
*   *CORS，你值得拥有*
*   *上帝保佑*
*   *CORS，呃，这有什么好处*
*   *只有 CORS*

*🤦‍♂️*