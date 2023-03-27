# 使用 Keycloak 保护您的 NestJS API

> 原文：<https://itnext.io/protecting-your-nestjs-api-with-keycloak-8236e0998233?source=collection_archive---------0----------------------->

![](img/ff22f63d0a45a3477bbb3bfd8d2406d7.png)

“在古代，猫被当作神来崇拜；他们没有忘记这一点。”—特里·普拉切特

从 Keycloak 网站来看，“ **Keycloak** 是一个开源的身份和访问管理解决方案”。今天我们来看看如何用 Keycloak 保护你的 HTTP API。

如今，保护 HTTP API 的现代方法之一是通过“授权:载体<token>”HTTP 头，令牌是携带身份和声明(角色等)的 JWT。)的 API 的消费者。</token>

我们假设您已经有一个 JS 前端应用程序，或者至少有一个 HTTP 客户端执行了针对 Keycloak 的身份验证，并且拥有一个 JWT，可以将它作为 HTTP“Authorization:Bearer<token>”头传递到您的 NestJS 后端。</token>

JWT 可以是对称签名的(签名和验证 JWT 的秘密相同)，也可以是非对称签名的(用私钥签名的令牌，可以用相应的公钥验证)。Keycloak 使用后者，这很好，因为它允许多个后端能够验证 jwt，而不需要跨多个服务传播秘密。这意味着，如果您的一个服务受到威胁，至少攻击者无法自己伪造 jwt 来攻击其他服务。

# 履行

我们需要编写一个保护程序来修饰我们想要保护的控制器或单独的处理程序。

这个防护将使用一个**认证服务**，它将执行(以各种方式，如下所示)JWT 的**验证。**

所有需要的服务都将是 AuthenticationModule 的一部分，该模块将导出应用程序其余部分可能需要的一些服务。

我们将提供一个可行的实现，并在以后对其进行改进，使其在生产、E2E 测试等方面更加实用。

我们走吧:

如果您想要该状态下应用程序的完整代码。这里是提交:[https://github . com/Paz tek/nestjs-authentic ation-example/tree/7376 AFB 2691 FDD 6972 eceb 70 ceda 86 e 1103 b 716d](https://github.com/paztek/nestjs-authentication-example/tree/7376afb2691fdd6972eceb70ceda86e1103b716d)

# 丰富

我们可以改进的事情之一是提供一种方法，在您的 E2E 测试期间不运行 Keycloak。在您的 E2E 测试中使用真实的 Keycloak 实例，虽然更现实，但是由于需要额外的 HTTP 调用、等待 Keycloak 启动并预加载具有不同角色集的用户列表，因此会使您的测试变慢。

目前，在 E2E 测试中绕过 Keycloak 的一个合理方法是用一个**FakeAuthenticationService**替换 AuthenticationService。但随着您的 AuthenticationService 封装越来越多的逻辑(在您的本地数据库中创建用户、计算会话或分析等)，这将变得越来越不可接受。).

另一种方法是 **stub 使用 HttpService** 进行的 HTTP 调用，但是一旦您的 HttpService 也用于执行对其他服务的 HTTP 调用，这将变得越来越复杂。

下面，我提出一个解决方案，**保持你的测试足够真实，同时消除了运行 Keycloak 实例**的需要。

通过将 JWT 的验证委托给另一个服务，我们可以保留 AuthenticationService，并在测试期间使用不同的策略来验证 JWT。不要忘记添加 NODE_ENV=test 环境变量。

如果您对实际源代码中与测试相关的代码过敏，并且您对 AuthenticationModule 感到惊讶，您可以执行以下操作:

# 遗留问题

当 JWT 的发行者不同于用于获取用户信息的 URL 时，Keycloak 会发出抱怨。当你的前端根据 Keycloak 的公共 URL(例如 https://your-domain.com/auth 的)协商它的令牌，但是你的后端使用你的 Keycloak 实例的内部地址(例如 http://10.0.1.23/auth 的[)时，就会发生这种情况。奇克洛的团队不认为这是一个错误:](http://10.0.1.23/auth).)[https://issues.redhat.com/browse/KEYCLOAK-5045?_sscc=t](https://issues.redhat.com/browse/KEYCLOAK-5045?_sscc=t)所以我们必须解决它。

如果您不能轻松地实现所描述的解决方法(更新后端的/etc/hosts，以便 your-domain.com 解析到 10.0.1.23 ),如果您没有使用 Keycloak 的令牌撤销功能，并且您的访问令牌寿命很短(例如，5 分钟),您可能会发现以下解决方案是可以接受的。

**我们正在手动验证 JWT**，而不是向 Keycloak 请求。但是为了做到这一点，我们必须拥有与用于签名令牌的私钥相对应的公钥。
我们不能在 NestJS 应用程序上硬编码密钥，因为 Keycloak 会定期更改签名密钥。这意味着我们必须调用奇克洛来取回它们。

# 结论

建议对初始实现进行重构有几个好处:

*   这使得 E2E 测试更容易编写和运行
*   当我们在生产中遇到 Keycloak 的限制时，它允许我们快速编写一个解决方法。

下一次，我们将在这个实现的基础上，通过验证经过身份验证的用户的角色来研究授权。

一如既往，这里是 Github 库的链接:[https://github.com/paztek/nestjs-authentication-example](https://github.com/paztek/nestjs-authentication-example)