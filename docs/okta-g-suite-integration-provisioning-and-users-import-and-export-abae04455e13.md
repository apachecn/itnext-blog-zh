# Okta: G-Suite 集成—供应和用户导入和导出

> 原文：<https://itnext.io/okta-g-suite-integration-provisioning-and-users-import-and-export-abae04455e13?source=collection_archive---------1----------------------->

![](img/8363e4592d20e5d200e3f53cded292f5.png)

继续我们项目的 Okta 设置。

以前的帖子:

*   [Okta:Gmail 和 Slack 的 SSO 认证](https://rtfm.co.ua/en/okta-sso-authentication-for-gmail-and-slack/)
*   [Jenkins:Okta SSO 和用户组的 SAML 认证](https://rtfm.co.ua/en/jenkins-saml-authentication-via-okta-and-users-groups/)
*   [Jenkins: SAML、Okta、用户组和基于角色的安全插件](https://rtfm.co.ua/en/jenkins-saml-okta-users-groups-and-role-based-security-plugin/)
*   [Github: SAML、Okta、Github 企业云—组织单点登录配置](https://rtfm.co.ua/en/github-saml-okta-and-github-enterprise-cloud-organization-sso-configurartion/)

下一个任务是将我们的 [Google Suite](https://gsuite.google.com/) 与 Okta 集成:需要配置一个将用户从 G Suite 导入 Okta 的能力，反之亦然。

实际上，我们将使用 Okta 作为用户数据库和认证系统的真实来源，Okta 将管理 G Suite 中的用户帐户，但在本文中，我们将测试这两种能力。

文档:

*   [为 G 套件配置供应](https://support.okta.com/help/s/article/28208416-Configuring-Provisioning-for-Google-Apps)
*   [G 套件供应](https://help.okta.com/en/prod/Content/Topics/Provisioning/Google/google-provisioning.htm)

## okta—G 套件应用程序配置

进入 Okta > *应用*，点击*添加应用*，找到 G Suite 应用:

![](img/2b7056388190d272478d480a09a93216.png)

在 G Suite 管理页面进入*域名*，找到一个组织的*主域名*:

![](img/f8d2082347a88f54aef9680dfbf12bb3.png)

将其设置为以下设置(该域对配置不起任何作用，但在稍后使用 SAML SSO 时会用到):

![](img/3f59515754d46b0bb35eaa1e780880a3.png)

将*应用用户名格式*设置为*电子邮件*，此处的其他内容可以保留默认值—登录将在下次配置:

![](img/e1411971ade41ae01f7dd03b03677e4b.png)

保存，切换到*设置*页签:

![](img/6391a9aeb0d1dd4864215de19f07d369.png)

## g 套件供应配置

g 套件必须启用 API，查看[文档> > >](https://support.aodocs.com/hc/en-us/articles/205158773-Enable-G-Suite-APIs-in-your-domain) 。

在 Okta —点击*配置 API 集成*:

![](img/f48b02f6a72bbf27dfe5a6be69baafb1.png)

点击*用 G Suite* 认证，登录一个需要的账号:

![](img/29f559c2c78181f3f7988bef52833221.png)

允许访问:

![](img/6fe52a4d90dc269a92a5d262eaf97e17.png)

就绪:

![](img/e153fa1f8220c747ade1fd96a5cbb8f9.png)

点击*保存*。

## 用户从 G Suite 导入到 Okta

在 Okta 转到*设置>到 Okta* :

![](img/c2b7a57ad4aaa206fe90ea45b69fab78.png)

在*用户创建&匹配中，*您配置 Okta 将如何比较来自 G Suite 账户和 Okta 本地数据库的用户( [*Okta 通用目录*](https://www.okta.com/products/universal-directory/) )。

在目前的情况下，用户的电子邮件将被 G Suite 和 Okta 使用相同的值。

运行导入的时间—切换到*导入*选项卡:

![](img/e8b2a012e3e6dda09cf9370199a6b88a.png)

点击*立即导入*:

![](img/a57984f9afc40155b5e20f5b3cc1dc9c.png)

等待 5-10 分钟，取决于您的 G Suite 帐户大小:

![](img/eeefc29a91bb20776daf5a827ebaa8a9.png)

我们的 G 套件中的所有用户现在都可以在 Okta 的帐户中创建:

![](img/c55371f79a8592404561c8b7aa749078.png)

一旦你分配了其中的任何一个，Okta 通用目录中就会创建一个帐户。

例如，这里有一个 *Arseny* 用户——它已经出现在 Okta 帐户和 G 套件中，因此 Okta 将跳过它:

![](img/dd06d5d45df3c74a192b3d5c0c3885fc.png)

但是第二个 *Arseny* 将被创建，因为这是另一个具有不同电子邮件的人，匹配不适用。

从 G Suite 中选择一个要在 Okta 中创建的用户:

![](img/c54ba6fd5901662df2f7dfba6826020d.png)

您可以在这里选择一项操作:

1.  在 Okta 创建一个全新的帐户(默认操作)
2.  将它附加到已经存在的 Okta 用户
3.  无视就好

点击*确认作业*:

![](img/61682b67bc2076ce797b9599eb058aea.png)

Okta 现在创建了一个新用户:

![](img/b489a3cd38ba44612ba3be554dfcfc44.png)

## 用户从 Okta 导出到 G 套件

现在让我们尝试设置后端供应—如果现在在 G Suite 中找到了 Okta 的用户，那么必须在 G Suite 中创建该用户。

进入*设置>应用*，点击*编辑*，启用必要选项:

![](img/ddf494865ea0a638900ebc38c9d60f2a.png)

出于测试目的，您可以只启用*创建用户*，以避免意外删除 G Suite 中已经存在的用户(如果有的话)。

点击*保存*。

在 Okta 的数据库中创建一个新用户—转到*目录>人员>添加人员*:

![](img/70568bd098586de25176f15241ea66be.png)

***注意:*** *必须已经在 G 套件账户*中配置了邮件域

返回 G Suite 应用程序，切换到 *Assignments* 选项卡，分配这个新用户:

![](img/1e4ed0696a98c1bba2867232195a96e9.png)

在 hist profile 中为 G Suite 设置值，至少需要配置*组织单元*(当然如果使用 OU 的话):

![](img/45be59723da15bc883d2f2a2202ce891.png)

检查 Okta 的日志:

![](img/21dfefb10e1aa2883efd748f171fb506.png)

和 G 套件帐户:

![](img/13801883bcfebd053ba9ba0bbbda63d2.png)

## 通知

最后一件事是，如果 Okta 从 G Suite 进行了导入，那么在 Okta 中创建新用户时配置通知。

进入*设置>账户*，在*管理邮件通知*模块配置要发送的邮件:

![](img/d489b305986c2aa54e37b5ac1de87473.png)

完成了。

*最初发布于* [*RTFM: Linux、DevOps 和系统管理*](https://rtfm.co.ua/en/okta-g-suite-integration-provisioning-and-users-import-and-export/) *。*