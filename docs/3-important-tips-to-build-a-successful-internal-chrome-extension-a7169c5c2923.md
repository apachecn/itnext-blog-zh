# 构建成功的内部 Chrome 扩展的 3 个重要技巧

> 原文：<https://itnext.io/3-important-tips-to-build-a-successful-internal-chrome-extension-a7169c5c2923?source=collection_archive---------2----------------------->

![](img/e6ae053b2e229566b240116d2f3d0421.png)

如果“内部铬延伸”抓住了你的眼睛，祝贺你！你在一个人数不多但不断增长的开发团队中，他们了解 Chrome 扩展对他们的公司有多强大。我是 [extension.dev](https://extension.dev) 的联合创始人，这是构建内部 Chrome 扩展的最快方式。

**在本文中，我将分享 3 个重要的技巧，帮助你更成功地构建和部署安全的 Chrome 扩展供内部使用。**

Chrome 扩展不仅仅是为了记住密码；它们帮助从初创公司到财富 500 强的公司快速扩展并节省大量时间。他们通过让开发人员能够为内部工具添加重要的缺失功能和补丁，定制 SaaS，或者自动化重复的浏览器工作流来做到这一点。通过扩展，单个开发人员可以通过以下方式提高整个组织的生产力:

*   向他们的 CRM/ERP 添加批量上传功能
*   为他们的用户管理内部工具添加急需的按钮
*   向 GitHub 和吉拉添加有用的 UI 元素
*   自动化过去在多个网站上进行的繁琐工作流程

但是开发和部署供内部使用的 Chrome 扩展可能会很困难。此外，扩展有时会引起 IT/安全部门不必要的关注。

以下是你可以做得更好的方法。

# 技巧 1:正确部署企业扩展

一旦您构建了将为您的销售人员每周节省时间的扩展，您如何将它安全地交到他们手中，并持续部署更新而不会被 Google 审查流程阻止？有丑陋的，好的，好的方法来做这件事。

如果你在快速迭代，我们推荐好的部署方式，如果你在内部发布，我们推荐好的方式。请继续阅读，看看是如何做到的。

## 丑陋的

有些开发人员会做这样的事情:

**发布公共扩展**

*   请不要这样做。任何人都可以找到您的扩展，嗅探您的 API，窃取您的凭证，等等。
*   这种情况我们已经见过很多次了。不信的话，在谷歌上查:`inurl: “[chrome.google.com/webstore/detail](http://chrome.google.com/webstore/detail)” “internal”`显示省略的结果。
*   每次你试图推出新版本时，你的扩展都是谷歌审查过程的受害者。这会让你的释放停止。

**发布未列出的扩展**

*   没那么糟糕，因为没有人能够公开找到这个扩展，但是任何有链接的人仍然可以下载它。如果链接泄漏，这是一个问题。
*   每次你试图推出新版本时，你的扩展仍将受到谷歌的审查。

## 很好

如果你不想让它彻底杀死你，你必须考虑其他选择，如:

**主持人。Google Drive 上的 zip 文件**

*   发送驱动链接，供人们下载并安装到开发人员模式的浏览器上。
*   这绕过了谷歌审查和你公司内部的审查。如果它不会打扰你，这是一个很好的方法，你也重视快速发布给你的利益相关者。

**主持人。crx 文件远程自动更新**

*   这让你可以完全独立地托管 Chrome 扩展，并且你可以公开 Chrome 扩展的链接以供查看。它也可能觉得这个解决方案更合法。
*   的更新。crx 可以立即部署，加速您的迭代。
*   将`update_url`键添加到您的`manifest.json`中

```
{
  "name": "My extension",
  ...
  "update_url": "<http://myhost.com/mytestextension/updates.xml>",
  ...
}
```

*   安装的扩展将定期轮询包含版本号的 XML 文件的链接。如果轮询的版本号不同于已安装的扩展，扩展将获取。属性`codebase`下的 XML 响应中指定的 crx 文件:

```
<?xml version='1.0' encoding='UTF-8'?>
<gupdate xmlns='<http://www.google.com/update2/response>' protocol='2.0'>
  <app appid='aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa'>
    <updatecheck codebase='<http://myhost.com/mytestextension/mte_v2.crx>' version='2.0' />
  </app>
</gupdate>
```

*   更多细节，请参考[这些](http://www.dre.vanderbilt.edu/~schmidt/android/android-4.0/external/chromium/chrome/common/extensions/docs/autoupdate.html) [文章](https://developer.chrome.com/docs/apps/autoupdate/)。

# 好的

**发布私有扩展**

*   这似乎是显而易见的，但是你不会相信有多少开发人员根本不这么做。
*   私下发布使您能够将对扩展的访问限制在您的电子邮件域中的人，或者指定的“可信测试者”电子邮件地址的白名单中。
*   私下出版可能仍然会使你受到谷歌的审查(更少)，但是这仍然会减慢你的迭代周期。根据我们的经验，私下出版将审查时间缩短到 1 天，而不是公开 Chrome 扩展所需的 3-7 天。

**在您的域中强制安装扩展**

*   你可以通过谷歌管理控制台强制安装应用和扩展。
*   查看 [*管理企业扩展*白皮书](https://support.google.com/chrome/a/answer/9296680?hl=en)中标题为“创建您自己的内部网络商店”的部分。

**结论:自主主持。拉链还是。crx 文件，一旦你准备好发布到你的组织，就私下发布。**

# 技巧 2:安全的消息传递可以拯救你的生命

让您的后台脚本(background.js)盲目地信任数据并监听内容脚本(content.js)可能会导致恶意网站控制您的扩展。你应该假设 content.js 生活在蛮荒的西部。

幸运的是，您可以控制何时将输入传递给 background.js，以及 background.js 如何处理它们。请继续阅读，找出方法。

但首先，这里有一些可能出错的方式:

## background.js 评估 content.js 中的输入

*   这有时可能很诱人，但请不要这样做。
*   评估使攻击者很容易窃取用户浏览器中的所有 cookies，嗅探存储在 background.js 中的 API 密钥，并调用 background.js 中的 API 来做可怕的事情。

## content.js 诱骗 background.js 做一些恶意的事情

*   即使不进行评估，content.js 也可能说服 background.js 调用破坏性的 API，比如修改/删除用户帐户、进行购买或采购等。如果 background.js 从不单方面进行有后果的 API 调用，这个问题可以得到缓解。
*   这可能是因为 [Spectre 攻击](https://spectreattack.com/)有时允许恶意网站从内容脚本中读取数据，即使这两者是“隔离的”。

以下是你在 background.js 中自救的方法:

## 严格限制权限匹配模式

*   这保证了您的扩展只在您公司信任的页面上运行。请尽可能做到这一点。
*   您信任的页面，尤其是如果它们是内部工具或 SaaS，不太可能欺骗您的扩展调用破坏性的 API，或者将不安全的输入传递到 background.js 中

## 对于不受信任的页面，坚持使用 background.js 中的只读 API 调用

*   有时，您希望将您的扩展部署到不可信的网站上，尤其是如果您正在构建一个 scraper。
*   为了将不受信任的网站劫持你的内容脚本的影响降到最低，尽量在 background.js 中尽量少调用 write APIs。

## 当直接对来自内容脚本的输入采取操作时要小心

*   永远不要对内容脚本输入调用 eval。
*   尽量不要做使用 background.js 打开链接或者调用 content.js 传过来的 API URIs 之类的事情
*   对于其他任何事情，在对输入做任何事情之前，都要对其进行净化和验证。对`&`、`<`、`>`、`"`、`'`等字符进行转义，防止意外 XSS。

**结论:小心消息传递，将不受信任的消息限制在 background.js 中，在所有情况下，确保 background.js 不会盲目地对来自内容的未经确认的输入采取行动。**

# 技巧 3:考虑使用远程托管的前端

这件事有很多好处。

你可以在你的 UI 上快速迭代，而不需要接触扩展代码，完全避免了 Google 的审核过程或者不得不重新部署。

你可以在任何你想要的框架中编写你的前端。

您可以更轻松地迭代您的接口，而不必将前端放在 popup.js 或扩展 Javascript 文件中。

*一定要考虑远程托管更多 UI 组件。这些组件可能看起来像覆盖图、模态图和侧边栏。*

方法如下:

## iframe 注射液

超级简单。只需在 Vercel 或 Netlify 之类的服务上托管您的前端，记下托管它的 URL，并使用 content.js 将前端作为 iframe 注入页面。

下面是一个代码示例:

```
this.sidebarWrapper = document.createElement("div");
this.sidebarWrapper.className = "sidebar-wrapper";
this.sidebarWrapper.id = "sidebar-wrapper";this.iframe = document.createElement("iframe");
this.iframe.id = "sidebar-iframe";
this.iframe.tabIndex = 0;
this.iframe.src = src;this.sidebarWrapper.appendChild(this.iframe);
document.body.appendChild(this.sidebarWrapper);
```

应该指向你的前端，你应该根据喜好调整 iframe 的位置和尺寸。

## 但是远程托管的前端将如何与我的扩展对话呢？

很好的问题，很容易回答。

**当你的前端想和你的分机通话时，从前端发送消息到 background . js……**

```
chrome.runtime.sendMessage("YOUR_EXTENSION_ID",
	{
		/* whatever data you want */
		command: "DO_SOMETHING",
		source: "REMOTE_FRONTEND",
		payload: {
			/* more random data */
		}
	},
	(response: any) => {
		resolve(response);
	}
);
```

**…然后，接收来自后台的消息:**

```
chrome.runtime.onMessageExternal.addListener(
  function(request, sender, sendResponse) {
		if (request.command === "DO_SOMETHING") {
			doSomething(request.payload);
		}
 });
```

**当你想从你的分机和你的前端通话时，从 content.js** 开始

```
window.postMessage({
	/* whatever data you want */
	source: "CONTENT_SCRIPT",
	payload: {
		/* more random data */
	}
}, "REMOTE_FRONTEND_URL" /* important for security */);
```

结论:远程托管的前端是你的朋友。利用这一点来加速您的开发，在您选择的框架中开发，并避免在 Chrome 扩展包的范围内编写您的前端。

*注意:*在将于今年晚些时候更新的清单 V3 中，公共扩展中将不允许远程托管的前端。如果您在企业内部署，则不受此限制。

# 结论

我们希望这些提示能够让你为你的公司构建安全的 Chrome 扩展，从而提高生产力和工作体验。

为了更容易地构建企业 Chrome 扩展，可以考虑注册 [extension.dev](http://extension.dev) ，这是我正在开发的一个低代码企业 Chrome 扩展构建器。

[扩展名. dev](http://extension.dev) :

*   给你一个低代码的 UI 编辑器，它也是完全托管的。
*   按照最佳安全措施烘烤，这样它就不会沾到你的箱子上。
*   为您处理基于角色的访问控制、身份验证和部署。
*   为您提供分析，以便您可以看到您的扩展所产生的影响。

# 参考

*   [https://docs . Google . com/document/d/1 pt 0 zsbgdrbgvucsvd 2 jjxrw-GVz-80 rms 2d gkkquhty/edit #](https://docs.google.com/document/d/1pT0ZSbGdrbGvuCsVD2jjxrw-GVz-80rMS2dgkkquhTY/edit#)
*   [https://support.google.com/chrome/a/answer/2714278#step4](https://support.google.com/chrome/a/answer/2714278#step4)
*   [http://www . dre . Vanderbilt . edu/~ Schmidt/Android/Android-4.0/external/chromium/chrome/common/extensions/docs/auto update . html](http://www.dre.vanderbilt.edu/~schmidt/android/android-4.0/external/chromium/chrome/common/extensions/docs/autoupdate.html)
*   https://developer.chrome.com/docs/extensions/mv3/messaging/
*   [https://developer . chrome . com/docs/extensions/mv3/security/# content _ scripts](https://developer.chrome.com/docs/extensions/mv3/security/#content_scripts)