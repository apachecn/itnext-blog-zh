# 用苹果万能钥匙走向无密码网络

> 原文：<https://itnext.io/towards-a-passwordless-web-with-apple-passkeys-5ebe274922d7?source=collection_archive---------5----------------------->

使用密钥无需密码即可登录服务和应用

![](img/22ae7ee3126b7f3492512f07229e19b3.png)

肯尼·埃利亚松在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

苹果即将发布被称为下一代认证技术的 Passkeys。据苹果公司称，这种解决方案不仅可以改善用户体验，还可以排除一些安全风险，如凭据泄露或网络钓鱼。

# 密码的问题

几十年来，密码一直是认证机制的标准。从登录页面到锁屏，到处都在用。

但是密码有一个问题。你必须记住他们。

根据 NordPass 委托[进行的一项研究](https://tech.co/password-managers/how-many-passwords-average-person)，普通人需要记住大约 100 个密码。哦，我差点忘了密码必须是强而唯一的。它们必须至少有 15 个字符，包含数字和各种奇怪的字符。

这个问题的解决方案来自于**密码管理器**。他们会记住你所有的密码。

但是等等，有没有更好的解决方案来解决这个问题呢？根据苹果公司的说法，这种解决方案被称为 *passkeys* 。

# 万能密码简介

> 密钥是密码的替代品。它们登录更快，更容易使用，也更安全。

这是我在苹果官网上读到的关于 passkeys 的第一句话之一。好像很牛逼。

我们来分解一下这句话。

*   **密钥代替密码**。他们依靠生物认证，像触控 ID 或 Face ID。你的苹果设备被称为*认证器*。对于每个帐户，身份验证者生成一个公钥-私钥对，并与服务共享公钥。
*   **使用密码登录更快。**无论是自动填写还是手动填写，您都不必填写两个字段，然后等待一次性代码，只需使用密码点击表单并登录即可。
*   **更容易使用。**密钥由您的设备自动生成。它们会使用 *iCloud 钥匙串*自动同步。你什么都不用做。只需点击并登录。
*   **更安全。**万能钥匙依赖于多层安全保护。首先，它们基于不对称加密。第二，它们使用 *iCloud Keychain* 进行同步，它使用端到端加密，使用苹果甚至不知道的强密钥。此外，对 *iCloud 钥匙链*的访问也是限速的，以防止暴力攻击，即使是来自云基础设施内部的攻击。第三，任何使用 iCloud 钥匙扣的 Apple ID 都需要双因素认证。

需要强调的是，您不会与您登录的服务共享任何密码。您共享一个公钥，如果没有在您的设备中端到端加密的私钥，它就没有那么有价值。

# 发射日期

从 iOS 15 和 macOS Monterey 开始就有了 Passkeys，但只是在开发者预览版中。在 macOS Ventura 和 iOS 16 中，每个人都可以使用 Passkeys。

iOS 16 将于 9 月 12 日周一发布。然后，iPadOS 16 和 macOS Ventura 都将在 10 月份推出。

# 资源

[1][https://developer.apple.com/passkeys/](https://developer.apple.com/passkeys/)

[https://support.apple.com/en-us/HT213305](https://support.apple.com/en-us/HT213305)