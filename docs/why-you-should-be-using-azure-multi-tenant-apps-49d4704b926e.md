# 使用 Azure 多租户应用注册

> 原文：<https://itnext.io/why-you-should-be-using-azure-multi-tenant-apps-49d4704b926e?source=collection_archive---------0----------------------->

![](img/0b91f46ff503404ab3f337a747756931.png)

杰森·理查德在 [Unsplash](https://unsplash.com/s/photos/tenant?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

Azure 中的应用程序注册是以编程方式访问 Azure 和 Office 365 中的资源的关键，但我发现它们经常被很多人混淆和误解！在过去的几天里，我回答了来自队友和客户的问题，他们要求一个租户中的应用程序能够检索另一个租户的信息。

退一步讲，我们为什么需要一个 App 注册？应用程序注册是 Microsoft 身份平台以及身份和访问管理(IAM)的一个组件。无论是 web 或移动应用程序之类的客户端应用程序，还是支持客户端应用程序的 web API，注册它都会在您的应用程序和身份提供商 Microsoft identity platform 之间建立信任关系。

以微软的 graph API 为例，应用程序注册可以通过两种方式与 Graph API 进行交互:

*   **委托权限:**有登录用户在场的应用。
*   **应用程序权限:**在没有登录用户的情况下运行的应用程序。

但是如果我们需要从外部租户访问数据呢？这就是多租户应用注册的用武之地。多租户应用程序基本上是任何 Azure AD 租户使用您的应用程序的一种方式，通过在租户之间创建无密码信任，最终使您的应用程序能够从外部租户获取数据并与之交互。

![](img/23329d8705f5dc19794b9e8448a9d274.png)

image:[https://docs . Microsoft . com/en-us/azure/active-directory/develop/how to-convert-app-to-be-multi-tenant](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-convert-app-to-be-multi-tenant)

*注意:接下来的这些步骤假设你在创建应用注册、配置平台 URL 和创建秘密方面有一定的经验。如果没有，请通读* [*快速入门*](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app) *以加快速度。*

那么如何在 Azure 中设置多租户 app 呢？作为开发人员，在创建应用程序注册时，您需要做的只是选择一个多租户选项。

![](img/08695029c47245d0bb678f87f2b314ce.png)

然后，对于 API 的委托权限，当用户向您的应用程序进行验证时，系统会提示他们授予您在 API 权限中选择的任何范围的访问权限。但是，如果您的后台工作人员也需要访问外部租户中的数据，该怎么办呢？委派的用户权限在这里不起作用—这就是应用程序权限的用武之地。

首先，您需要确保您的应用程序选择了“应用程序”权限类型，而不是“委托”(在本例中是从图形 API 中选择的)。

![](img/4a3b5c1f14c0bfb86c1cf5e6b95b72b1.png)

对于将使用您的应用程序的任何客户，他们还需要在浏览器中输入管理员同意 URL，以便将您的应用程序添加到其租户内的企业应用程序中，例如，
*https://log in . Microsoft online . com/organizations/v 2.0/admin condence？client_id= <您的客户端 id>&scope = https://graph . Microsoft . com/。默认*

最后，您需要获得客户租户 id 的副本，以便在调用 Graph API 时以编程方式使用。

我们现在准备进行 API 调用，从您客户的租户那里获取信息！使用[微软文档](https://docs.microsoft.com/en-us/graph/auth-v2-service)中的步骤，我们首先需要代表您客户的租户向微软进行认证:

```
POST [https://login.microsoftonline.com/{your](https://login.microsoftonline.com/{your) client's tenant}/oauth2/v2.0/token HTTP/1.1 
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded client_id=<your app registration client id>
&client_secret=<your app registration client id>
&scope=https://graph.microsoft.com/.default
&grant_type=client_credentials
&resource=[https://graph.microsoft.com](https://graph.microsoft.com)
```

这个请求应该会产生一个成功的响应和一个如下所示的对象:

```
{
  "token_type": "Bearer",
  "expires_in": "3599",
  "ext_expires_in": "3599",
  "expires_on": "1630420066",
  "not_before": "1630416166",
  "resource": "https://graph.microsoft.com",
  "access_token": "eyJ0eXAiOiJKV1QiLCJub2..."
}
```

然后使用结果中的 access_token 值，您现在可以通过将它作为承载令牌传入来对 Graph API 进行后续调用:

```
GET [https://graph.microsoft.com/v1.0/users/](https://graph.microsoft.com/v1.0/users/)
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJub2...
```

这应该会产生一个响应，其中列出了来自外部租户的所有用户—您可以通过查看响应中的 *@data.id* 部分的租户 id 来验证这一点:

```
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users",
  "value": [{
    "@odata.id":
"https://graph.microsoft.com/v2/4...3f5/directoryObjects/f...407/Microsoft.DirectoryServices.User",
    "displayName": "Gavin Lewis",
    "givenName": "Gavin",
    "preferredLanguage": "en-AU",
    "surname": "Lewis",
    "userPrincipalName": "gavin@....onmicrosoft.com",
    "id": "f...407"
  }]
}
```

现在你知道了！您现在可以通过使用多租户应用程序注册从外部租户检索信息！