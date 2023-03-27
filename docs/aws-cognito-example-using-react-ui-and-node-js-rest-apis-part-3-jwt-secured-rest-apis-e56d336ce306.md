# 使用 React UI 和 Node.js REST APIs 的 AWS Cognito 示例—第 3 部分(JWT 安全 REST API)

> 原文：<https://itnext.io/aws-cognito-example-using-react-ui-and-node-js-rest-apis-part-3-jwt-secured-rest-apis-e56d336ce306?source=collection_archive---------0----------------------->

![](img/0d72841b86ccd6327c0c4ae985aabead.png)

在 [Unsplash](https://unsplash.com/search/photos/keys?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上由 [CMDR Shane](https://unsplash.com/@cmdrshane?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄的照片

在我的 AWS Cognito 系列的第三篇也是最后一篇文章中，我将讲述如何使用 AWS Cognito 发布的 JSON Web Token (JWT)访问代码创建和保护一个简单的基于 Express 的 Node.js REST API 服务。我们还将修改我们在本系列的第二篇文章[中创建的 React UI 应用程序，以调用这个 REST API，并包含它从 Cognito 接收的一个 JWT 访问代码。](https://medium.com/@arron.harden/aws-cognito-example-using-react-ui-and-node-js-rest-apis-part-2-react-ui-app-with-redux-6cc22610affe)

# JWT 概述

有大量的文档已经涵盖了 JWT(尝试[https://jwt.io/introduction](https://jwt.io/introduction/))，所以我不想在这里重复，相反，我将给出一个快速的概述，然后我们将看看一些具体方面的认知。

简而言之，JSON Web 令牌(JWT)是一段经过编码和数字签名的 JSON，被调用者可以对其进行验证，以便 I)检查调用者的授权，以及 ii)获取调用者的身份信息(例如姓名、电子邮件地址、帐户 id 等)。

与更集中的用户注册类型的方法相比，JWT 有几个优点，这就是为什么它成为 API 认证和授权的一个非常流行的解决方案。对我来说最大的优势是:

*   不需要外部呼叫来验证 JWT 接入码。签名机构(我们示例中的 Cognito 用户池)的公钥被下载、缓存，然后用于验证传入 API 请求上的 JWT 访问代码的签名。如果签名得到验证，那么这意味着 JWT 访问代码只能从我们的 Cognito 用户群中发出。这极大地减少了验证传入 API 请求的延迟和总体系统开销，并使扩展系统变得更加容易。
*   JWT 接入码可以包括关于呼叫者的补充信息(例如，姓名、电子邮件地址、账户 id 等)。同样，这些都是可以实现的，不需要任何外部调用。
*   JWT 接入码通常具有到期时间。我见过使用 5 分钟(基于系统的 JWT 接入码)到 60 分钟(基于用户的 JWT 接入码)的实现。过了这段时间后，它们就不能再使用了，这意味着与用户名/密码或 API 密钥的无意泄露不同，JWT 访问代码的泄露只有一个很小的可用时间窗口。

显然，应该注意不要在日志、诊断或任何其他正在进行的持久性操作中暴露用户的 JWT 访问代码。它们应该被视为秘密，就像 API 密钥、密码等。不要忘记，它们可能包含个人身份信息(PII ),因此即使在过期很久之后，也可能受到 GDPR 政策的保护。

# 科宁托和 JWT

作为 Cognito UI 登录流程的一部分，我们的 UI 应用程序实际上接收了 3 个 JWT 访问代码，如下所述。

## 认知 ID 令牌

*ID 令牌*包含关于呼叫者身份的信息(例如姓名、电子邮件地址、帐户 ID 等)。来自 Cognito 的(过期的)编码 JWT ID 令牌的示例如下所示:

```
eyJraWQiOiIwbjd6c2g0eDRlZzVCRmszZk9vNlNLeWkwenJHODNESysyQ1wvNXhtRnFNMD0iLCJhbGciOiJSUzI1NiJ9.eyJhdF9oYXNoIjoiNHhSNmpkODJlRUtiYkFXcnh2cVJlUSIsInN1YiI6IjFmMGJlNjJmLWZmY2QtNDljYS1iNWE0LTE4ZjBiZjYyZTBlNiIsImF1ZCI6IjR0bnA0azY0ZDV2NGFoOWR1ZDNwajFrYnMwIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsInRva2VuX3VzZSI6ImlkIiwiYXV0aF90aW1lIjoxNTY1MDIwNDQ5LCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAuZXUtd2VzdC0yLmFtYXpvbmF3cy5jb21cL2V1LXdlc3QtMl96MUdvNVhkcloiLCJjb2duaXRvOnVzZXJuYW1lIjoiYXJyb24iLCJleHAiOjE1NjUwMjQwNDksImlhdCI6MTU2NTAyMDQ0OSwiZW1haWwiOiJkZW1vQGFycm9uaGFyZGVuLmNvbSJ9.kh_Bdao1BzXIoGvcE8ByAAFdLYb8s_Tcf0RwKk6joJr43r0Q0N7Yz63c5O4DYoUKgwdHRNhKax93rkSCamabmNl8f3-K68OOC3MNn22Fn2p6prOn9jXeK4y_QDHzOj8LFSJTsQxjIq6JfaZReTfHdplEsIVfmqijFO7PPQHcK4GCgYCTxiyD8iaKF48tcdNrm9NczgtzVdT6ShNK8LRcGhdXSYxLk0h0k8lYRhV8Imy2uxAvuY9wgeKvrY7FPHJc8dsgfVuRXAn-QCtEaTCxpOlVuNgtJ06Ww09ur1eINIFFQ_WCqUV1vP7JQEylItrf1WxAYKJDNrKqWSuZeogr3A
```

使用 [jwt.io](https://jwt.io/) ，我们可以对此进行解码，并看到报头包含以下关于 jwt 访问码是如何构造的信息:

```
{
  "**kid**": "0n7zsh4x4eg5BFk3fOo6SKyi0zrG83DK+2C/5xmFqM0=",
  "**alg**": "RS256"
}
```

并且有效载荷包含以下身份信息:

```
{  
  **"at_hash"**:"4xR6jd82eEKbbAWrxvqReQ",
  **"sub"**:"1f0be62f-ffcd-49ca-b5a4-18f0bf62e0e6",
  **"aud"**:"4tnp4k64d5v4ah9dud3pj1kbs0",
  **"email_verified"**:true,
  **"token_use"**:"id",
  **"auth_time"**:1565020449,
  **"iss"**:"https://cognito-idp.eu-west-2.amazonaws.com/eu-west-2_z1Go5XdrZ",
  **"cognito:username"**:"arron",
  **"exp"**:1565024049,
  **"iat"**:1565020449,
  **"email"**:"demo@arronharden.com"
}
```

属性名遵循标准的 JWT 命名约定。一些有用的属性包括:

*   `sub`是代表用户的唯一且不变的 ID。当持久化与用户相关的内容时，使用该属性的值，而不是用户名或电子邮件地址，因为用户可以随意更改后者。
*   `aud`是用于获取 JWT 接入码的客户端 ID。这应该与用户池中定义的客户端 ID 相匹配。
*   `token_use`描述它是什么类型的 JWT 接入码— *ID 令牌*或*接入令牌*。
*   `iss`是发行者，对于 Cognito 来说是创建 JWT 访问代码的用户池的 URL。这应该与您的用户池相匹配。
*   `cognito:username`是包含用户名的自定义 Cognito 属性。用户可能希望改变这一点，所以避免在应用程序中持久化它。
*   `exp`是到期时间戳，在此之后，JWT 访问码将不再可信。
*   `email`是用户当前的电子邮件地址。用户可能希望改变这一点，所以避免在应用程序中持久化它。

根据 Cognito 用户池的配置方式， *ID* *令牌*可以包含更多的身份信息，如全名、电话号码等，但在本例中，用户池仅配置为存储姓名和电子邮件地址。

## 认知访问令牌

一个*访问令牌*包含关于调用者的访问权限的信息，并且包括用于获得令牌的 OAuth 范围。有效负载中的一个`scope`属性将用于决定允许调用者调用哪个(哪些)API。例如，如果有效负载中的范围是只允许`hello-world.read-only`，但是被调用的 API 是一个`POST /users`，以便创建一个新的资源——实现将被期望返回一个 HTTP 403(禁止的)响应，以指示调用者没有足够的权限来执行这个特定的 API 调用。

*访问令牌*的解码有效载荷的一个例子是:

```
{  
  **"sub"**:"1f0be62f-ffcd-49ca-b5a4-18f0bf62e0e6",
  **"token_use"**:"access",
  **"scope"**:"openid profile https://cognito-demo-api.arronharden.com/hello-world.all email",
  **"auth_time"**:1565020449,
  **"iss"**:"https://cognito-idp.eu-west-2.amazonaws.com/eu-west-2_z1Go5XdrZ",
  **"exp"**:1565024049,
  **"iat"**:1565020449,
  **"version"**:2,
  **"jti"**:"bb8bade7-2514-4152-8a41-a1524bf43f21",
  **"client_id"**:"4tnp4k64d5v4ah9dud3pj1kbs0",
  **"username"**:"arron"
}
```

请注意，当比较*访问令牌*和 *ID 令牌*的有效载荷时，包含相同信息的一些属性的名称是如何不同的，例如`client_id`对`aud`和`username`对`cognito:username`。如果需要的话，`token_use`属性可以用来确定提供了哪种类型的 JWT 访问代码。

## 认知刷新令牌

我们的 UI 从 Cognito 收到的第三个 JWT 访问代码是一个*刷新令牌。*该令牌用于在原始令牌和*访问令牌*过期后获取新的 *ID 令牌。*刷新令牌*实际上是加密的，这意味着只有 Cognito 服务能够看到有效负载的内容(您可以通过尝试 jwt.io 来确认这一点，jwt . io 也无法解码它)。刷新令牌*本身具有更长的寿命，以天而不是分钟来计算，因此出于这个原因，必须格外小心以保持刷新令牌的秘密。

# 验证认知 JWT 访问代码

## JWKS 下载

在我们能够验证任何东西之前，我们需要做的第一件事是一次性下载与我们的 Cognito 用户池实例相关联的公钥信息。这些信息被称为 JSON Web Key Set (JWKS ),可以通过在使用您的 Cognito 用户池的区域和 ID 构建的 URL 上发出 GET 请求来下载:

```
https://cognito-idp.<region>.amazonaws.com/<user_pool_id>[/.well-known/jwks.json](https://cognito-idp.eu-west-2.amazonaws.com/eu-west-2_z1Go5XdrZ/.well-known/jwks.json)
```

在本系列文章的第一篇[文章](https://medium.com/@arron.harden/aws-cognito-example-using-react-ui-and-node-js-rest-apis-part-1-cognito-setup-5597acb02db4)中输入我们创建的用户池的值，就会得到 URL[https://cogn ITO-IDP . eu-west-2 . Amazon AWS . com/eu-west-2 _ z1go 5 xdrz/。知名/jwks.json](https://cognito-idp.eu-west-2.amazonaws.com/eu-west-2_z1Go5XdrZ/.well-known/jwks.json) 。由于这些是公钥，因此无需任何授权令牌即可下载，URL 将返回以下 JWKS 内容:

```
{  
  **"keys"**:[  
    {  
      **"alg"**:"RS256",
      **"e"**:"AQAB",
      **"kid"**:"0n7zsh4x4eg5BFk3fOo6SKyi0zrG83DK+2C/5xmFqM0=",
      **"kty"**:"RSA",
      **"n"**:"tKQOwFsFCRSNiL18QN77UaYYQvBvgIr-ImRW-9Y6KnfYeS5Tk-O39JjllVCYNf6EB5xcZELQuaJmh_bm8uiOxFuk1t6pCghyQWUWU_uR8BRhrqxF-ugWbnWWfkGqPjMD9Tc3oGVIZRwqpXSXkGaid6DaqhkQIzrYQtPwUMge4w4oQgl5KuowBxrxjJdrRChi4FPABNoRaB8VJOHAgvoobP9VACRkooSXKz5b0aCzExuv5mkg8WnhJi_xQCjX41QX3RbpwvgoqPrCxYmCjngy5qwItsJSOozLdxwFp6EDQExp5VwRfd0xU112Ny0ea-EeDlgkQMgVLfvb7cOm74f5WQ",
      **"use"**:"sig"
    },
    {  
      **"alg"**:"RS256",
      **"e"**:"AQAB",
      **"kid"**:"iPeYlnLRSbCoyi+JdywuePKNdluh4+7tIPaOn8yWZuE=",
      **"kty"**:"RSA",
      **"n"**:"iPu6YsSrKbVMMMmhwcyQjdzJHY9y53nEj-oHF3VimQu8gZ0qXaQRckJudZ0AOzDHZvSfdP2mLUMa6HWa_1n7NNsBWCRWTjqOXex2iAOX2Ryo9Sa_pRmXOCEPAbQor2YdYQdKqetbUllhPXmzTRtCGEcXbn8bB40rhAAcRoFUhBqWyUxIQPzwZlhTzk41u5E0V3iIt2jnFTTXfxgHj2571VTCHYTyqdOqcmdx4zvVaY8SUEg-VFRy4GM76JBEgttv7AhPtRlGkhKgIpPb8UJPDXMdtfFaHPzWOGGe9qh1YsuCeEqIUCwTmhy4sO8yVn6ylhJZpFAF8zQLtxIvYRwzdw",
      **"use"**:"sig"
    }
  ]
}
```

管理下载的 JWKS 内容有多种技术。对于这个例子，我们在应用程序启动时下载一次，将密钥转换成 PEM 容器(这是我们稍后使用的 JWT 库所需要的),并将其保存在内存中。

## JWT 验证

为了验证给定的 JWT 访问代码是否可信，我们需要在允许任何请求的 API 操作继续之前对其执行一些测试。

1.  验证使用 JWT 库可以解码访问代码。这验证了格式是好的，但是没有(还没有)验证内容没有被第三方篡改。
2.  验证 JWT 访问代码的头部包含一个`kid`(密钥 ID)，该 ID 与我们的 Cognito 用户池的 JWKS 内容中返回的 ID 相匹配。
3.  使用 JWT 图书馆来验证访问代码。这将验证:

*   可以使用我们之前从 Cognito 用户池的 JWKS 内容中下载并缓存的公钥来验证签名。这确认了没有篡改，也确认了它是由我们的用户池实例签名的。
*   发卡行声明(`iss`)与我们的 Cognito 用户池实例相匹配。
*   访问代码没有过期。

如果以上都成功了，我们就可以确定 JWT 访问代码是从我们的 Cognito 用户池生成的，并且没有被篡改。因此，我们可以信任 JWT 接入码有效载荷中的所有内容。

# 创建 JWT 安全 REST API

既然我们知道了验证 JWT 接入码需要做什么，我们就可以开始编码了。我们首先使用`express` [CLI 命令](https://expressjs.com/en/starter/generator.html)创建一个新的 skeleton Express 应用程序。

```
express --no-view --git cognito-demo-service
```

这给了我们一个现成的 Node.js Express 应用程序，它监听端点`/users`。

因为我们希望能够与 UI 应用程序同时在本地运行，所以我们将其更改为监听端口 3010，而不是 3000(修改`/bin/www`)。我们还利用了`cors`包，这样当 REST API 托管在[https://cognito-demo-api.arronharden.com](https://cognito-demo-api.arronharden.com)上时，就可以从托管在不同域上的 UI 中调用它；https://cognito-demo.arronharden.com。这需要对`app.js`做一点小小的添加，以加入`cors`中间件:

```
*// Configure CORS for this service so our UI can make calls to us.* var corsOptions = {
  origin: [appConfig.signoutUri]
}
app.**use**(**cors**(corsOptions))
```

## 快速中间件

接下来，我们定义我们的 Express 中间件。这将做两件事:

*   验证传入的 REST API 调用上的 JWT 访问代码。我们将它定义为设置在`Authorization`报头上，包括前缀`Bearer`，后跟 JWT 访问码。
*   从 JWT 访问代码中提取声明，并将它们设置到传入请求对象上名为`user`的对象中。这允许任何下游中间件或处理程序直接从请求中访问解码的信息。

我们在自包含模块中定义我们的中间件，其入口点是`getVerifyMiddleware()`。这将启动 JWKS 内容的下载，并返回 Express 要使用的中间件功能。

创建并返回验证中间件

通过将中间件功能包含在`app.js`中`/users`端点的路由器之前，可以轻松地将其插入 Express:

```
const cognitoAuthMiddleware = cognitoAuth.**getVerifyMiddleware**()
app.**use**('/users', cognitoAuthMiddleware, usersRouter)
```

当调用中间件时，调用`_verifyMiddleware()`函数，该函数验证`Authorization`头，如果成功，则在调用链中的下一个快速处理程序之前，使用`next()`将解码的信息添加到请求对象中。另一方面，如果验证不成功，它会根据情况立即向调用者返回 HTTP 401(未授权)或 HTTP 500(内部服务器错误)响应，而不调用下一个处理程序。

调用中间件时的入口点— _verifyMiddleware()

上述`Authorization`报头的实际验证由`_verifyProm()`函数处理，该函数执行前面描述的 JWT 访问码验证步骤，并返回一个*承诺*，该承诺根据验证是否成功而被解决或拒绝。

验证授权头— _verifyProm()

值得一提的是，在失败的情况下，我们将大多数诊断和错误信息保存在我们的私有应用程序日志中。返回给 REST API 调用者的错误包含最少的信息，因为我们不想在无意中帮助恶意用户的情况下提供太多帮助。

## 在 API 中使用声明

到目前为止，我们所做的只是允许或拒绝传入的 API 调用，这很好，但是我们现在也可以在 API 实现本身中使用 JWT 访问代码中的声明。我们可以通过简单地检查由我们的中间件添加到请求中的`user`对象的内容来做到这一点。在本例中，我们将简单地在响应中返回`user`对象的内容，以演示如何从下游处理程序访问它:

```
*/* GET users listing. */* router.**get**('/', function (req, res, next) {
  *// Here we can check the req.user.scope array contains the scope
  // relevant for the REST API operation being invoked* res.**send**('Successfully verified JWT token. Extracted information '
    + JSON.**stringify**(req.user))
})
```

## 使用 curl 调用

有了这些改变，我们现在准备尝试调用`/users` API。但是在我们开始改变用户界面来做到这一点之前，我想确保我们在这项服务中的工作如我们所预期的那样，所以我们可以尝试几个快速的`curl`命令来进行测试。

在成功的情况下，我们在`Authorization`报头中传递一个有效的无记名令牌:

```
$ curl -H "Authorization:Bearer eyJraWQiOiJpUGVZbG5MUlNiQ295aStKZHl3dWVQS05kbHVoNCs3dElQYU9uOHlXWnVFPSIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiIxZjBiZTYyZi1mZmNkLTQ5Y2EtYjVhNC0xOGYwYmY2MmUwZTYiLCJ0b2tlbl91c2UiOiJhY2Nlc3MiLCJzY29wZSI6Im9wZW5pZCBwcm9maWxlIGh0dHBzOlwvXC9jb2duaXRvLWRlbW8tYXBpLmFycm9uaGFyZGVuLmNvbVwvaGVsbG8td29ybGQuYWxsIGVtYWlsIiwiYXV0aF90aW1lIjoxNTY1Nzk0Nzg3LCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAuZXUtd2VzdC0yLmFtYXpvbmF3cy5jb21cL2V1LXdlc3QtMl96MUdvNVhkcloiLCJleHAiOjE1NjU3OTgzODcsImlhdCI6MTU2NTc5NDc4NywidmVyc2lvbiI6MiwianRpIjoiMDJkODk3OTQtMzIyNy00MTVlLWE1N2UtNGI1YzhlZmU2MGVjIiwiY2xpZW50X2lkIjoiNHRucDRrNjRkNXY0YWg5ZHVkM3BqMWticzAiLCJ1c2VybmFtZSI6ImFycm9uIn0.FmV1wxFHuZWYhVIdR9JnVHBzhbaoJlaUe-aw3yUte__2tRHo-zw6KyyDXQycdZ0JOo8S054mtESdYhJTc4-WTn89ymKUiXLgJXooA7w0cSI2vku1te97loPr5qqh-pnvrtf2XadDjp63lEeUG3Hq9HZEGDbyEUURPL5jMPZrbUnJevKanmPNfJRouHv2OBK6fzq79LnllmGi-_iQo_hu5NBfR6Erx33JbPcVBnw8t9o0JF4rqNPJbYT7qIXXNpRJtMzYVeY6hgfo85eq1h83DouTP3qmyiWYXBg98FoEXq-ROHq2g1E1qPasdi6IO0afqqrkYG8QaNfgGxomlehDtg" [http://localhost:3010/users](http://localhost:3010/users)
Successfully verified JWT token. Extracted information: {"sub":"1f0be62f-ffcd-49ca-b5a4-18f0bf62e0e6","token_use":"access","scope":["openid","profile","https://cognito-demo-api.arronharden.com/hello-world.all","email"],"username":"arron"}
```

不要忘记失败的情况，我们传递一个无效的 JWT 访问代码:

```
$ curl -H "Authorization:Bearer badbadbad" [http://localhost:3010/users](http://localhost:3010/users)
Authorization header contains an invalid JWT token.
```

在 Node.js 控制台中，我们还可以看到相应的调试输出，显示 200 HTTP 响应和 401 HTTP 响应:

```
Valid JWT token. Decoded: {"sub":"1f0be62f-ffcd-49ca-b5a4-18f0bf62e0e6","token_use":"access","scope":"openid profile [https://cognito-demo-api.arronharden.com/hello-world.all](https://cognito-demo-api.arronharden.com/hello-world.all) email","auth_time":1565794787,"iss":"[https://cognito-idp.eu-west-2.amazonaws.com/eu-west-2_z1Go5XdrZ](https://cognito-idp.eu-west-2.amazonaws.com/eu-west-2_z1Go5XdrZ)","exp":1565798387,"iat":1565794787,"version":2,"jti":"02d89794-3227-415e-a57e-4b5c8efe60ec","client_id":"4tnp4k64d5v4ah9dud3pj1kbs0","username":"arron"}.
GET /users **200** 1.108 ms - 238
Invalid JWT token. jwt.decode() failure.
GET /users **401** 0.882 ms - 51
```

# 从 UI 调用 JWT 安全 REST API

现在我们已经有了我们支持的 REST API 服务，从我们的示例 React UI 中使用它是一件相对简单的事情。

首先，我们修改`getCognitoSession()`函数来返回 JWT 访问代码，这样它们就可以在 Redux store 中使用了。然后我们更新 *Home* 组件，这样如果用户登录，就会调用`/users` API 并使用`setState()`方法存储响应。

使用 JWT 访问代码调用 REST API 的主页组件

剩下的只是更新 *Home* 组件的`render()`方法，以显示我们从 API 调用中得到的响应。一切顺利，这应该是我们看到的从上面的`curl`示例返回的相同文本。

主页组件呈现 REST API 响应

运行 UI 并进行登录，现在显示的是 *Home* 组件，以及从 REST API 返回的响应，显示 JWT 访问代码已发送、解码、验证并提取了声明:

![](img/07570d659ec85220822d1798b64cdbdd.png)

更新的 React UI 显示成功的 API 响应

# 摘要

我们已经探索了 AWS Cognito 用户池的创建，它与基于 React 的单页面应用程序的集成，以及现在由 Cognito 发布的 JWT 访问代码保护的后端 REST API 的使用。

我们也可以使用 AWS API 网关服务为我们做一些 JWT 的工作，但是我想在这篇文章中深入 JWT 的细节，所以我将把 API 网关服务的探索留到另一个时间。

## 来源

这一系列文章中的所有源代码都可以在 GitHub 中找到:

*   React UI 应用程序来源:[https://github.com/arronharden/cognito-demo-ui](https://github.com/arronharden/cognito-demo-ui)
*   基于 Express 的 REST API 来源:【https://github.com/arronharden/cognito-demo-service 

## 实时实例

这两个 Node.js 应用程序的实例都在 AWS 中运行，只要我的免费帐户保持活动状态，我就会努力让它们运行:

*   React UI 应用:【https://cognito-demo.arronharden.com 
*   基于 Express 的 REST API:[https://cognito-demo-api.arronharden.com](https://cognito-demo-api.arronharden.com/)