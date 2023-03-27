# 是时候杀密码了

> 原文：<https://itnext.io/its-time-to-kill-the-password-bbab40f36ba0?source=collection_archive---------2----------------------->

如何通过 Web 身份验证为无密码的未来做好准备

![](img/46b07e43eb6f482fdcae09c87d011efa.png)

[Icons8 团队](https://unsplash.com/@icons8?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

想象一下，你在某个地方的保险箱里藏了一些非常贵重的珠宝。

打开这个保险箱的唯一方法是把密码交给另一个人，他会为你打开它。你把密码写在一张纸上，放进信封，合上信封，然后交给这个人。

这个人也必须沿着街道走到你的保险箱。在这个过程中，代码可能会丢失，甚至被盗。

此外，你甚至不知道这个人是否可信。

*你会同意这种保障吗？似乎不太安全，是吗？*

这里的问题是，打开你的保险箱的密码现在是一个*共享秘密*，这也是密码的问题。

当你需要登录一个网站来访问你的电子邮件或银行账户时，你向服务器发送你的密码来证明你就是你所声称的那个人。

这是你和你的银行账户、电子邮件或其他敏感数据之间唯一的障碍。这也是站在黑客和你的敏感数据之间的唯一考虑。

因此，如果黑客设法窃取你的密码或猜测它，因为它是薄弱的，他们可以完全访问你的数据。大约 80%的黑客入侵都与被盗密码或弱密码有关。

# 密码是通向您数据的大门

你可以在大多数通往你公司的门上装上厚重的锁，但是如果只有一扇门仍然敞开着，这些锁就帮不了你了。

一个弱密码就足以危及你所有的安全措施。

密码管理器有助于使您的数据更加安全，但仍然不能防止密码通过网络钓鱼攻击被窃取。这种风险可以通过使用双因素身份认证(2FA)来最小化，幸运的是，2FA [的采用率从 2017 年的仅 28%上升到 2019 年的 53](https://duo.com/blog/the-2019-state-of-the-auth-report-has-2fa-hit-mainstream-yet)。

但是即使是 2FA 也不能幸免于复杂的网络钓鱼攻击。这对人们来说也不方便，而方便是人们仍然使用[令人震惊的弱密码](https://www.nbcnews.com/better/lifestyle/worst-passwords-2019-they-re-so-weak-even-novice-hacker-ncna1106626)的主要原因。

# 如何停止分享你的秘密

如果你的密码是一个秘密，那么保护它的最好方法显然是停止分享它。但由于你需要与网络应用程序共享它来登录，这不是一个选项。

网络认证为此提供了一个解决方案。

它通过使用非对称加密消除了对密码的需求，非对称加密涉及一对由公钥和私钥组成的密钥对。公钥被用来加密只有相应的私钥才能解密的东西*。*

想象一下你想给我发一条秘密信息。我可以给你一个有钥匙的盒子，把这个秘密信息放进去。你可以把消息放在盒子里然后锁住，但是你不能*解锁*它。

用于消息的盒子是公钥，而可以打开盒子的密钥是私钥。

我可以自由地将我的公钥(只能上锁的盒子)分发给任何想给我发送秘密消息的人，因为没有相应的私钥，它实际上是没有用的。所以我的私钥是保密的，不应该被共享。

这是 Web 身份验证(WebAuthn)的基础。

# WebAuthn 如何工作

WebAuthn 允许你使用名为*授权码的硬件设备登录网站，比如苹果的 TouchID、Windows Hello 或 USB 安全密钥。*

这些授权码通常会生成并存储一个由公钥和私钥组成的新密钥对(称为*凭证*)。然后，该凭证可用于向网站注册。

注册后，可以从身份验证器中检索该凭证，以便用户登录该网站。

不是客户端向服务器发送密码，而是服务器向客户端发送一个非常大的随机字符串，称为*挑战*。然后，客户端用私钥对这个质询进行签名，这意味着它生成了这个质询的散列。

然后，客户机将这个散列连同相应的公钥一起发送回服务器。

然后，服务器可以用公钥验证这个散列，这证明客户端拥有相应的私钥，并且认证成功。

没有共享的秘密，只有一个公钥，没有相应的私钥是没有用的。这意味着存储这些公钥的数据库对黑客不再有吸引力，网络钓鱼也不再有用。

私钥安全地存储在硬件验证器中。最重要的是，WebAuthn 中使用的所有凭证都是有作用域的，这意味着为某个网站(来源)创建的凭证只能*用于该来源。*

创建用于注册的凭证，然后使用它进行身份验证是相当简单的步骤，在客户端需要两次调用:

`navigator.credentials.create()`创建注册凭证，`navigator.credentials.get()`检索认证凭证

让我们详细了解一下注册新凭据并使用它进行身份验证的步骤。

# 注册凭据

我们需要采取的第一步是创建一个新的凭证(keypair ),并将其注册到我们想要进行身份验证的网站。

要创建新凭据:

```
const credential = await navigator.credentials.create({
    publicKey: {
      challenge: {
        type: "Buffer",
        data: [53, 69, 96, 194, ...]
      }, 
      rp: {
        name: "What PWA Can Do Today",
        id: "whatpwacando.today"
      },
      user: {
        id: {
          type: "Buffer",
          data: [134, 196, 104, ...]
        }, 
        name: "webauthn@whatpwacando.today", 
        displayName: "WebAuthn"
      },
      pubKeyCredParams: [{alg: -7, type: "public-key"}],
      authenticatorSelectionCriteria:  {
        attachment: "platform",
        userVerification: "required"
      },
      timeout: 60000
    }
});
```

对`navigator.credentials.create()`的调用接收一个带有单键`publicKey`的对象，该对象包含创建新凭证的配置。

让我们来分解属性:

`challenge`是从服务器接收的随机字符串，必须用公钥签名，以证明用户拥有相应的私钥。

`rp`代表依赖方，这是负责注册和认证用户的实体，即我们要注册的网站。`id`必须是网站域名的子集。

`user`包含当前注册用户的信息。它必须至少包含属性`id`，该属性用于将用户与凭证相关联。它是在服务器上生成的。

`pubKeyCredParams`是一个对象数组，描述将要创建的凭证的特性。目前，`type`唯一可能的值是“公钥”，而`alg`的值-7 表示具有 SHA-256 的椭圆曲线算法 ECDSA。

`authenticatorSelectionCriteria`为允许创建凭据的授权码设置标准。
`attachment: "platform"`意味着我们想要一个绑定到客户端的不可移除的认证器。
`userVerification`表示是否需要用户交互来验证用户。因为我们希望使用指纹进行身份验证，所以该值应该为“true”。

`timeout`包含脚本等待注册过程完成的时间(毫秒)。*注意，这是一个可能被浏览器覆盖的提示。*

当注册过程完成时，返回一个`PublicKeyCredential`对象:

```
PublicKeyCredential {
  id: "Aa-aGSY1jGxNZlF9...",
  rawId: ArrayBuffer(103), 
  response: AuthenticatorAttestationResponse {
    attestationObject: ArrayBuffer(265),
    clientDataJSON: ArrayBuffer(266)
  },
  type: "public-key"
}
```

`id`是新创建的凭证的 ID，用于在用户尝试进行身份验证时识别它。

`rawId`是二进制形式的`id`。

当我们*注册*一个新凭证时`response`是类型`AuthenticatorAttestationResponse`。
当我们*认证*时，它的类型是`AuthenticatorAssertionResponse`。

`response`中的`clientDataJSON`是从浏览器传递到认证器的数据，用于将凭证与服务器和浏览器相关联。

`attestationObject`包含新生成的公钥、可选的证明证书和其他用于在服务器上验证的元数据。

`type`当前可能只保存值“公钥”。

我们现在可以将凭证发送到服务器，以下列形式进行注册:

```
const data = {
  rawId,
  response: {
    attestationObject,
    clientDataJSON,
    id: credential.id,
    type: credential.type
  }
};
```

# 用凭证认证

现在我们已经注册了我们的凭证，让我们尝试进行身份验证！

就像注册一样，我们只需要一个简单的呼叫来进行认证:

```
const credential = await navigator.credentials.get({
  publicKey: {
    challenge: {
      type: "Buffer",
      data: [236, 146, 80, ...]
    },
    allowCredentials: [
      {
        id: credential.rawId,
        type: "public-key",
        transports: ["internal"]
      }
    ],
    rpId: "whatpwacando.today",
    userVerification: "required"
  }
});
```

我们现在调用`navigator.credentials.get()`来检索与我们在 options 对象中传递的所有标准相匹配的凭证。这个对象也有一个保存实际标准的键`publicKey`。

`challenge`也是来自服务器的随机字符串，必须用公钥对其进行签名，以证明用户拥有相应的私钥。

`allowCredentials`是包含凭据描述符的数组，这些凭据描述符限制了用于身份验证的凭据。
`id`是我们之前创建的凭证的`rawId`属性。
目前，`type`唯一可能的值是“公钥”。
`transports`是一个数组，指定客户端和认证者之间可能的传输。`internal`表示认证器与客户端设备绑定，不可移除，如指纹识别器。

*根据规范* `allowCredentials` *是可选的，但我注意到当它被省略时，Chrome 会回退到注册一个新的凭据，而不是检索一个现有的凭据，从而导致错误。*

*还要注意，这需要我们传入凭证的* `rawId` *，这意味着我们需要将它存储在某个地方。在我稍后提供的演示中，* `rawId` *存储在客户端的* `localStorage` *中。在真实的场景中，用户将提供一个用户名，然后使用该用户名从服务器上的数据库中检索* `rawId` *。*

`rpId`是依赖方的标识符。如果未提供，则默认为网站的域。

`userVerification`表示是否需要用户交互来验证用户。因为我们希望使用指纹进行身份验证，所以该值应该为“true”。

当找到符合所有标准的凭证时，它将作为`PublicKeyCredential`返回:

```
PublicKeyCredential {
  id: "AdvZkPu73G7...",
  rawId: ArrayBuffer(103), 
  response: AuthenticatorAssertionResponse {
    authenticatorData: ArrayBuffer(37),
    clientDataJSON: ArrayBuffer(372), 
    signature: ArrayBuffer(72),
    userHandle: ArrayBuffer(32)
  },
  type: "public-key"
}
```

`id`是检索到的凭证的 ID。

`rawId`是二进制形式的`id`。

`response`现在属于`AuthenticatorAssertionResponse`类型。

`response`中的`authenticatorData`与注册步骤中的`attestationObject`相似，但不包含公钥。它用于身份验证过程中的验证。

`response`中的`clientDataJSON`是从浏览器传递到认证器的数据，用于将凭证与服务器和浏览器相关联。

`signature`是由检索到的凭证的私钥生成的(惊奇)签名，将使用私钥来验证它是否有效。

`userHandle`是在注册步骤中提供的`user.id`的值，将用于将用户与凭证相关联。

`type`目前可能只持有值“公钥”。

我们现在将以下中的凭据发送到服务器进行身份验证:

```
const data = {
  rawId: credential.rawId,
  response: {
    authenticatorData,
    signature,
    userHandle,
    clientDataJSON,
    id: credential.id,
    type: credential.type
  }
};
```

# 服务器端代码

为了在服务器端验证凭证，我使用了 [fido2-lib](https://github.com/webauthn-open-source/fido2-lib) 库，这使得这个过程相当简单。

但是由于这个库包含一个 bug，使我无法在演示中使用它，所以我分叉了 repo 并修复了这个 bug。这个叉子在演示中使用，可以在[这里](https://github.com/DannyMoerkerke/fido2-lib)找到。

如果您对解析和验证认证数据的确切步骤感兴趣，您可以[阅读规范](https://w3c.github.io/webauthn/)或阅读 [webauthn.guide](https://webauthn.guide) 上的精彩描述。

服务器端代码包含执行注册和身份验证所需的四个路由。这些路线将按下列顺序依次调用。

`/registration-options`返回传递给`navigator.credentials.create()`的配置对象。这包含来自服务器的质询、用户 id、关于依赖方的信息以及要创建的凭证的其他标准。

`/register`接收创建的凭证并验证它。如果通过验证，公钥将被存储。请注意，该演示将它存储在用户会话中。在实际场景中，用户名将存储在数据库中，以检索凭证的 id，并在下一个路由中将其发送回去。

`/authentication-options`返回传递给`navigator.credentials.get()`的配置对象。它再次包含来自服务器的挑战。在演示中，在客户端添加了`allowCredentials`，并从`localStorage`中检索所需的`rawId`凭证。
在实际场景中，用户将提供用户名，然后使用该用户名从数据库中检索凭证的`rawId`。在这种情况下，也可以在服务器端填充`allowCredential`。

`/authenticate`接收从客户端检索到的凭证并验证它。如果通过验证，则认证成功。

你可以在 [Github](https://github.com/DannyMoerkerke/webauthn-demo) 上查看演示和源代码，工作演示也是[PWA 今天可以做的事情](https://whatpwacando.today)的一部分。

# 结论

Web 身份验证使 web 应用程序能够提供非常可靠和安全的身份验证机制。它消除了密码的使用，极大地提高了安全性，因为不再共享机密，网络钓鱼攻击变得无用。

它让用户不必记忆密码，让网络应用程序不必存储和管理密码。用户可以使用他们的指纹或 USB 密钥方便地登录，使弱密码和重复使用的密码成为历史。

你可以[在 Twitter](https://twitter.com/dannymoerkerke) 上关注我，我经常在那里写关于 PWAs、网络组件和现代网络功能的文章。