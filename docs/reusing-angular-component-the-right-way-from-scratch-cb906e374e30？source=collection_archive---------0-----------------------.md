# 重用角度组件—正确的方法(从头开始)

> 原文：<https://itnext.io/reusing-angular-component-the-right-way-from-scratch-cb906e374e30?source=collection_archive---------0----------------------->

![](img/e5bc840837b78f7c42e0a6800decad42.png)

Angular 允许我们创建和重用组件，但是通常，我们做得不对。我见过很多次开发者只是把一个项目的组件复制到另一个项目，我们都知道 ctrl+c，ctrl+v 并不是一个很好的复用东西的方法，因为组件会在每个项目上单独进化。

为了避免这种情况，我们有 npm 存储库，允许我们发布一个独立的组件，并将其作为项目的依赖项。有一个公共版本的 npm 库(每个人都可以使用发布的组件)，但是你也可以使用私有的 npm 库。这有助于保护您公司的数据。

一个好主意是将一个组件看作一个项目，而不是“仅仅是一个简单的组件”。我这样说是因为组件应该有它们自己的版本、测试，并与应用程序分离。实际上，为什么不直接创建一个 git 组件库，然后我们为它设置一个 CI/CD 管道呢？这就是我们在这篇文章中要做的。

在这个例子中，我将使用 **Gitlab** ，我将使用[Nexus创建一个私有的 npm 库，因为它适合大多数项目，因为它支持其他注册表，如 Maven、Nuget、Docker 等。为了制作到组件的 CI/CD 管道，我将使用 **Jenkins** 。一切都将在 localhost 中完成，但可以使用服务器轻松复制。](https://www.sonatype.com/product-nexus-repository)

# 在这篇文章中我们将做什么？

目标是为角度组件设置自动化 CI/CD。您可以根据需要定制您的管道，一切由您决定。在这篇文章中，我只想用最简单的方法。

上图说明了流程:

![](img/6935c4fca8c8b0304f2f2e403358af8e.png)

## 工作流程

1.  有人创建了组件的新版本(源代码已更改)
2.  Gitlab 自动对 Jenkins 说(通过 webhook):“嘿，我有一个组件的新版本，你能在 nexus 上构建并发布一个新版本吗？”
3.  所以詹金斯向 Gitlab 询问源代码。Gitlab 表示，只会提供源代码和凭证🔒。但是 Jenkins 有资格并且得到了它(关于 Jenkins 的 SCM 部分)。
4.  Jenkins 自动开始构建过程，然后将这个新版本发送给 Nexus (npm publish)
5.  如果一切正常，詹金斯对 gitlab 说:一切正常。

# 要求

要完成本指南，您必须安装以下工具:

*   码头工人
*   Nodejs
*   角度 CLI
*   饭桶

## 实际上，我们将通过 docker 运行这些产品:

*   詹金斯
*   Gitlab
*   关系

# 1.装置

首先我们在 localhost 上运行 Nexus，Jenkins 和 Gitlab。为此，请运行以下命令:

## 1.1.联系:

```
docker pull sonatype/nexus3docker run -d -p 8081:8081 --name nexus sonatype/nexus3
```

在新容器中启动服务可能需要一些时间(2-3 分钟)。您可以跟踪日志以确定 Nexus 是否就绪:

```
docker logs -f nexus
```

![](img/17798415bff3d543a96c058aea1cd477.png)

运行此程序后，您将转到 [http://localhost:8081](http://localhost:8081) ，并且必须看到此屏幕:

现在我们需要获得 Nexus 的管理员密码。为此，我们必须执行以下命令:

```
docker exec -i -t nexus /bin/bashcat /nexus-data/admin.password
```

最后一个命令会给你密码

然后单击登录，然后使用如下图所示的凭据:

![](img/fca9c82a598be04f299591913470644e.png)

Nexus 中的身份验证

用户:管理员
密码:(在上一步中提供)

因此，将出现一个向导弹出窗口，您必须按照自己的方式进行配置。

## 1.2.詹金斯:

```
docker pull jenkins/jenkins:ltsdocker run -p 8080:8080 -p 50000:50000 -d — name jenkins jenkins/jenkins:lts
```

![](img/c035f47b3a7132f52f94446b74150e5a.png)

运行此程序后，您将转到 [http://localhost:8080](http://localhost:8080) ，并且必须看到此屏幕:

默认情况下，jenkins 会在首次初始化时要求输入管理员密码。要获取此密码，请运行上面的命令

```
docker exec -i -t jenkins /bin/bashcat /var/jenkins_home/secrets/initialAdminPassword
```

复制并粘贴屏幕上显示的代码，然后您将看到:

在下一个屏幕中，点击“安装推荐的插件”

![](img/ea0a9c5535485364f50d6d57dbdbc8b4.png)

在此创建您的 jenkins 管理员用户。

因此，创建您的管理员并登录，您将看到如下屏幕:

![](img/d082e0c5e5ff8afa5978b8e5e7dfc00d.png)

詹金斯的初始页面

## 1.3.Gitlab:

```
docker pull gitlab/gitlab-cesudo docker run -detach \
 -hostname localhost \
 -publish 443:443 -publish 80:80 -publish 22:22 \
 -name gitlab \
 -restart always \
gitlab/gitlab-ce:latest
```

在新容器中启动服务可能需要一些时间(2-5 分钟)。您可以跟踪日志以确定 Gitlab 是否准备好了:

```
docker logs -f gitlab
```

**注意:当 gitlab 准备就绪时，您将看到一个更改 root 用户密码的屏幕。不要忘记此密码，它将在本指南的后续步骤中使用。**

![](img/09226cecd766b4788f2d374d49585c4b.png)

Gitlab 首屏

点击注册并创建您的帐户。

登录，这将是屏幕。

![](img/a8d34f59696563b85e24b06c26eb6199.png)

现在，您已经具备了开始配置 CI/CD 管道的所有要求。所以，我们开始吧。

# 2.创建项目

现在我们需要创建一个包含这个包的 angular 项目📦。

因此，让我们运行以下命令来创建我们的 angular 应用程序:

```
ng new <my-app-name>
```

这将创建一个 angular 应用程序，**，但实际上它还不是库。**这一点很重要，因为**角度应用**和**角度库**是两回事。本文的主题是库(或者我们在这里称之为组件/模块)。应用程序需要看到组件运行，但是组件(库)被隔离在其他地方。

生成库的命令是

```
ng generate library <my-lib-name>
```

将在“projects/ <my-lib-name>”上创建一个新目录，这是**您的组件代码。**在实践中，您将在此目录下编写组件。</my-lib-name>

请注意，您的 lib 文件夹中有一个 package.json。这意味着你实际上是在编写一个孤立的 nodejs 应用程序，你需要在组件中设置的所有东西(比如依赖关系、版本控制等)都必须在那里编辑。

要查看关于如何编写组件的更多细节，并查看它在项目中的运行情况，我建议阅读本文的**部分 1** :

[](https://medium.com/@esanjiv/complete-beginner-guide-to-publish-an-angular-library-to-npm-d42343801660) [## 向 npm 发布 Angular 库的完整初学者指南。

### 角度> 6.0.0 和角度> 6.0.0 的逐步指南

medium.com](https://medium.com/@esanjiv/complete-beginner-guide-to-publish-an-angular-library-to-npm-d42343801660) 

现在，是时候在 gitlab 发表了。发布整个 angular 应用程序是很重要的，因为这使得贡献者测试组件变得容易。

要在 github 上发布，运行以下命令:

```
git init
git remote add origin <my-git-url>
git add .
git commit -m "Initial commit"
git push -u origin master
```

# 3.配置 Github、Gitlab 和 Jenkins

一旦用 angular 项目创建了存储库，就需要将 Jenkins 与 Gitlab 连接起来。所以首先你需要在 Jenkins 中安装 Gitlab 插件。因此，点击“管理詹金斯”，然后转到“管理插件”。然后找到 GitLab 插件并安装它。

![](img/4d68e94d11d94805cbc38dc47e4857d0.png)

现在我们需要配置 Jenkins 和 Gitlab。

## 3.1.配置 Jenkins 和 Gitlab

我们需要在詹金斯和 Gitlab 之间建立双向通信。但是为什么呢？

1.  需要 Gitlab webhook 通知 Jenkins 关于存储库的变化(Gitlab => Jenkins)。
2.  Jenkins 需要能够克隆存储库(Jenkins => Gitlab)。

## **3.2。将 Gitlab Webhook 预配置给 Jenkins**

我们之前需要这个额外的步骤，**因为在本文中我们通过 docker** 在 localhost 上运行一切，而 gitlab 的 webhook 默认不允许设置本地地址。**如果您没有在本地主机上运行，请跳过这一步。**

要使 gitlab 允许 webhook 上的本地地址，您必须使用 root 凭证登录。

![](img/2179aba5f1621377406bf942614f9d77.png)

登录后，您必须访问管理区( [http://localhost/admin](http://localhost/admin) )

现在，您需要进入设置= >网络

![](img/38eb6892c207df141666b55e049f48fc.png)

然后单击“出站请求”选项卡上的“扩展”，然后选中“允许从 web 挂钩和服务到本地网络的请求”

现在，您需要将 Jenkins 的 IP 添加到白名单中。要获取 Jenkins 的 IP，请运行以下命令:

```
docker inspect jenkins | grep IPAddress
```

毕竟你的屏幕肯定是那样的:

![](img/5cf5ac960677083cdadf78775973f21e.png)

允许本地地址上的 webhook

因此，点击保存更改，一切都准备好了。

现在，我们需要配置詹金斯。

点击“管理詹金斯”，然后进入“配置系统”。现在我们需要在“Jenkins URL”配置上设置 Jenkins 的 IP:

![](img/cbed50f7a231e010bf6bdf2889f7a093.png)

设置 Jenkins URL

## 3.3.在 Jenkins 上配置 Gitlab。

在 Gitlab 上，你需要生成一个供 Jenkins 使用的令牌。为此，您必须进入“用户设置”= >“访问令牌”。为这个令牌命名(我使用了“jenkins”)，并选择范围“API”(“Expire at”可以为空)，屏幕截图如下:

![](img/dfd26216b68d61ff7b8a341bd22a27fe.png)

生成 Gitlab 令牌

**注意:在实际场景中，建议在 gitlab 上创建一个新用户，以便在 jenkins 上使用。在这篇文章中，我使用自己的帐户，这是不推荐的。**

点击创建个人访问令牌，它将“生成一个访问令牌”。

**注意:请记住，如果您离开此页面，您将无法再次看到此令牌。**

![](img/307ff2c2ea8022ddb393252be4b725d5.png)

现在我们需要转到 Jenkins 并在凭据上添加此令牌。要做到这一点，访问“凭证”，然后去范围詹金斯。

![](img/14a8f980d1bf8a4300d312c6d38e71c0.png)

然后，将鼠标放在上面，会出现一个插入符号，单击它，然后单击“添加凭证”。

![](img/cd2bc2cd5120f1e8fc578bb81d2cf289.png)

因此，选择“GitLab API token ”,将上一步生成的令牌放入“API token ”,并在“ID”和“Description”中填写您想要的内容，然后单击“Ok”。

![](img/a72ea079a90edeb270b276e25a026bb9.png)

现在，转到“管理詹金斯”，然后“配置詹金斯”。在此页面中查找“gitlab”部分。像这样配置:

![](img/9031a814b4102851941cadb12df8fcfd.png)

**注意:要发现 Gitlab 主机 URL，请运行以下命令:**

```
docker inspect gitlab | grep IPAddress
```

## 3.4.在詹金斯身上创造新的工作

之后，我们需要创建一个关于詹金斯的工作。为此，请登录 Jenkins 并点击“创建新工作”。

输入名称，选择自由式项目，然后单击确定。

![](img/92bd311c6af50a43c9ae5696a954f078.png)

下一页是设置 Gitlab 库和 webhook。

*   在存储库 URL 上放置 Gitlab IP，而不是 localhost(当您使用 docker 运行 local 时)
*   在凭证中，点击“添加”,然后提供您的 Gitlab 帐户的用户名/密码。
*   在“分支说明符”中使用“主”
*   在“构建触发器”部分，选中“将变更推送到 GitLab 时构建”选项。这将允许詹金斯建立一个包后，在主分支推动

![](img/e5c6947b5d24423931b6ee012f775397.png)

构建配置示例

现在复制 Gitlab webhook url(在我的例子中是[http://172 . 17 . 0 . 3:8080/project/my-component-pipeline](http://172.17.0.3:8080/project/my-component-pipeline))，让我们在 Gitlab 中设置 webhook。

现在，我们需要从 Jenkins 生成一个令牌，以便 Gitlab Webhook 工作。为此，单击您的姓名，然后配置。所以在“API 令牌部分”。

![](img/b35e5795a1ae7a1e5ba56d40cea7ce30.png)

之后，您需要创建一个遵循以下格式的 URL:

> http(s)://登录:token @ Jenkins _ URL:Jenkins _ port/project/job

**注意:这个 URL 是你在上一步复制的 Gitlab webhook url。您只需在 ip 地址前添加“login:token@”(其中 login 是您的 jenkins 登录名，token 是您在此步骤中生成的值)。**

在我的例子中，我制作了这个 URL:

> [http://lucassklp:11390 b 872 e 4 BF 0 AE 3d 661664 f 724 f 36 c 70 @ 172 . 17 . 0 . 3:8080/project/my-component-pipeline](http://lucassklp:11390b872e4bf0ae3d661664f724f36c70@172.17.0.3:8080/project/my-component-pipeline)

## 3.5.配置 Gitlab webhook

现在我们需要配置 gitlab webhook。要做到这一点，进入你的项目，然后去“设置”，然后“集成”。用您生成的 URL 填充“URL”字段。

![](img/7b4eafafc74086bec0dbad7cf38e88ac.png)

Gitlab 中的 Webhook 配置

添加 webhook 后，您可以对其进行测试。预期的结果是一切正常。在“项目挂钩”中，通过发送 push 事件来测试它。钩子必须返回 HTTP 200:

![](img/1ec07cecd4ed6df78340841fbd141cca.png)

然后最后！！！！

![](img/58aa52b4157c70b3d416bb4ac326fdcb.png)

构建从 gitlab 推送开始

## 3.6.配置构建过程

Jenkins 与 Gitlab 集成在一起，现在每次在 master branch 上进行推送时，它都能够获得源代码。

但是我们需要构建组件。为此，我们需要在 Jenkins 上安装 [nodejs 插件](https://plugins.jenkins.io/nodejs/)。

安装插件后，进入“管理 Jenkins”，然后进入“全局工具配置”。

因此，添加具有所需版本的 nodejs 的新安装，如下图所示:

![](img/0aa51ed2e47b8a72c6b8c4cbc1634db9.png)

Jenkins 上的 Nodejs 插件配置

现在，转到您的管道并设置您在“构建环境”中创建的 nodejs 安装。

![](img/5887ec66fc0e5253dcf4ca678b91a95d.png)

组件构建环境

**注意:为了完成这项工作，你必须在你的包中添加下面一行**

```
“scripts”: {
   “build-component”: “ng build <my-lib-name>”
}
```

## 3.7.将 Jenkins 配置到 Nexus

首先我们需要配置 Nexus。

正如本文开头所介绍的，Nexus 提供了对许多存储库的支持，但是我们现在感兴趣的是 npm 存储库。

我们需要设置三个 Nexus 的存储库(我不会解释每个组件的细节，但会给你一个简介):

1.  托管-是放置您的包的“容器”。
2.  代理——如果在存储库中没有找到一个包，那么我们设置一个代理来在其他存储库中找到这个包。
3.  组——将多个存储库聚合成一个存储库。

## 3.7.1.创建存储库

**注意:你需要与管理员认证这样做。**

点击“齿轮”

![](img/a9dae46360fc040646192e1398410c5d.png)

创建存储库—步骤 1

![](img/1617a16fc57485f4a0e3ab531bdb6e87.png)

创建存储库—步骤 2

## 3.7.2.创建托管的

我们需要在 nexus 中存储我们的组件，所以我们需要创建一个托管存储库。

为此，重复第 3.7.1 节**中定义的步骤，然后选择 **npm(托管)。****

![](img/e789232044e2162e987b016584d06ec8.png)

创建托管存储库

根据需要填写名称(在本指南中，我将使用 npm-private ),然后单击“创建存储库”

## 3.7.3.创建代理存储库

现在，我们需要设置代理。为此，在选择 **npm(代理)时，重复**第 3.7.1 节中定义的步骤。****

![](img/8c41d250b3f961c0ae32692f0886c748.png)

在 nexus 上创建 npm 代理存储库

用您想要的任何东西填充名称(我将使用 npm-proxy ),远程存储将使用公共 npm 注册表。URL 地址是:

[https://registry.npmjs.org](https://registry.npmjs.org)

## 3.7.4.创建群组

组是存储库的集合。我们需要就内部国家预防机制和国家预防机制代理达成一致。为此，在选择 **npm(组)时，重复**第 3.7.1 节中定义的步骤。****

然后设置组名，并移动存储库。

![](img/73fe5f36096ce2b649ecd524ad347525.png)

在 nexus 上创建群组

## 3.7.5.在项目上设置默认存储库

首先，我们需要激活“npm 承载令牌领域”。该领域允许拥有先前生成的不记名令牌的用户发布 *npm* 包。

![](img/69a9b702384b2fb3fcdffb88dcd108f5.png)

之后，我们被允许使用 nexus 帐户发布来自 Jenkins 的包。

在这一点上，我们有一个接受新包的存储库，但我们不会从 Jenkins 发布它。那么，让我们来设置一下。

我们需要告诉 npm 默认的存储库来获取和发布包📦。这可以通过使用。npmrc 文件。

```
registry=http://<nexus-ip>:<nexus-port>/repository/<npm-group-repository>/
email=my-email@mail.com
_auth=base64(“<nexus-user>:<nexus-pass>”)
```

**注 1:此文件必须放在您的项目/ < my-lib-name >
注 2:这是您的群组库
注 3:不建议使用管理员帐户。请创建一个 Jenkins 帐户。**

现在我们可以获得依赖项，但是我们不能发布，因为托管存储库没有设置。为此，将以下节点添加到您的“项目/ <my-lib-name>/package.json”中:</my-lib-name>

```
“publishConfig”: {
   “registry”: “<nexus-ip>:<nexus-port>/repository/<npm-hosted-repository>/"
},
```

最后，我们需要更新 Jenkins 上的脚本，以便在 Nexus 上发布软件包🚀。

![](img/f9cff596dd9e6fcfa90ed33f578f16fb.png)

这是脚本的最终版本

然后，当您进行下一次提交时，您将在 nexus 上看到新版本的库，如示例所示🎉

![](img/d89207c84891f5588c07eff458d6917f.png)

组件发生变化后，将发布新版本🎉

# 4.结论

我们已经完成了重用角度组件的所有设置。现在，您可以跨项目重用您公司中的组件。尽管这篇文章很长，我还是错过了一些东西(比如开发过程、良好实践、自动测试等等)，但是我想以后再写这些要点。

就是这样。感谢你们的阅读，有任何问题，请告诉我=)