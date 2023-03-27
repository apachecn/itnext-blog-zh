# JAMstack 基础:如何用 Contentful 创建一个 Gatsby Starter 并部署到 Netlify

> 原文：<https://itnext.io/jamstack-basics-how-to-create-a-gatsby-starter-with-contentful-and-deploy-to-netlify-846354cc74bc?source=collection_archive---------1----------------------->

JAMstack 已经激发了一些我们见过的最伟大的网络开发工具。发布惊人的快速、安全和可访问的网站从未如此容易或如此自由。我仍然很难相信我自己的[个人网站](https://cwlsn.com)现在是免费的，而不是每月 15 美元的 VPS。如果你能把一个旧的架构转换成我们今天要讨论的，你可以把这些节省下来的钱投资到你的个人发展上，参加一个课程或教育会员。

![](img/155c492e3376217bcfebaa653a9ae2cb.png)

照片由[潘卡杰·帕特尔](https://unsplash.com/@pankajpatel?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

下面是你可以从这篇文章中学到的一些东西:

*   这些新技术是什么，它们有什么作用？
*   如何建立一个盖茨比启动器，或者如何从我建立的那个开始。
*   如何设置 Contentful 并创建您的第一个数据。
*   如何链接 Contentful 和 Gatsby，使用 GraphQL 获取这些数据并显示，如何使用 Gatsby 的 GraphQL playground 测试查询。
*   如何部署到 Netlify，将 Contentful 连接到 Netlify 以在数据更改时触发构建。

初学 React 经验是用 Gatsby 开发的必备。

# 盖茨比，心满意足，无忧无虑

这些工具处于这一运动的顶端，为各自领域的开发人员提供一流的服务。每一种都有不同的选择，做着同样伟大的事情，但是当我为自己的项目寻找相关性时，我喜欢用最好的。

[**盖茨比**](https://www.gatsbyjs.org/) 是一个静态站点生成器。你在 React 中建立你的站点，用 GraphQL 查询数据(这比听起来容易)，Gatsby 把它们全部转换成静态文件提供给浏览器。Gatsby 是非常可配置的，有很多插件可以让你做更高级的事情，但是默认情况下，它已经为你做了很多。

[**Contentful**](https://www.contentful.com/) 是个无头 CMS。这就像 WordPress，但是没有等待用户的站点。它允许您定义想要创建的内容类型，然后为您提供创建内容的表单。然后，它通过一个 API 提供您的内容，我们的静态站点生成器将在构建时与之对话。

[**Netlify**](https://www.netlify.com/) 将部署我们的网站并在网上免费提供服务。我们需要做的就是登录 GitHub，从我们的 repo 创建一个新的站点，并做一些简单的配置工作。由于 Gatsby starters 通常是托管在 GitHub 上的，我们甚至可以创建一个按钮来提前完成所有的配置。在你的站点成功部署后，Netlify 提供域名注册(或者你可以使用你自己的)和免费的 HTTPS 支持。

我们今天将使用这些工具的方式非常简单，几乎没有触及到什么是可能的，但是将您的第一个 JAMstack 站点放在您的腰带之下是迷上这个设置的关键。这些工具为我们做了这么多工作，我们可以专注于构建很酷的东西，让 DevOps 自己处理。

# 创建一个盖茨比启动器

开始一个盖茨比项目最简单的方法是使用一个启动器。在撰写本文时，官方的 [Gatsby Starter Library](https://www.gatsbyjs.org/starters/) 中几乎有 70 个，每个都提供不同的特性、插件和附加功能。我喜欢创建自己的样板文件(比如我的 [React 组件库样板文件](https://rinsejs.io/))，所以让我们看看 Gatsby 应用程序的最低要求。

*   React、ReactDOM、Gatsby 的基线依赖关系
*   `src/pages/index.js`导出 React 组件

就是这样！许多初学者会在第二页演示 Gatsby 附带的`Link`组件，但是我们现在可以跳过它。有一个回购有这个基本的启动程序，你可以在[盖茨比的 GitHub](https://github.com/gatsbyjs/gatsby-starter-hello-world) 上看到它。我们现在不需要修改这个，在我们深入研究数据和插件之前，我们需要获取一些东西！

如果您想玩“Hello World”示例，您可以进行 Gatsby 设置:

```
$ npm i -G gatsby-cli # lets us call Gatsby commands
$ gatsby new my-fun-site [https://github.com/gatsbyjs/gatsby-starter-hello-world](https://github.com/gatsbyjs/gatsby-starter-hello-world)
$ cd my-fun-site
$ yarn install # or npm i
$ gatsby develop
```

看看`src/pages/index.js`，做些改变。它将在您的浏览器中实时更新！

# 创造令人满意的…内容

当你最初注册 Contentful 的时候，他们会给你一个相当有指导意义的教程，所以我会简单介绍一下我们在这里做的事情的要求。

1.  **创建空间**。我不喜欢使用给定的示例空间，所以我删除了它，并创建了一个名为“Starter Site”的空间，我将在这里使用它。
2.  **创建内容模型**。你可以把它想象成一篇博文。一个博客文章内容模型会有一个标题、内容、标签和你喜欢的任何其他字段。在您的内容模型中，您定义这些字段以及它们在编辑它们的表单中的外观。对于第一个内容模型，我将创建一个名为“主页”的模型。Contentful 会把你的标题变成一个 API 标识符，就像`homePage`一样，后面会很重要。我喜欢坚持他们产生的东西。
3.  **在内容模型**中创建一些字段。有很多可供选择的，但是对于这个例子，我们将选择四个字段，以便在扩展 Gatsby starter 时获得一些多样性。我将采用每个的默认设置。字段分别是:
    -标题(短文本)
    -内容(长文)
    -日期(日期&时间)
    -图片(单媒体)
    别忘了保存！
4.  **补充一些实际内容**。在内容选项卡中，应该有一个“添加主页”按钮，如果你只是像我一样做了单一的内容类型。您会注意到编辑器非常丰富，长文本字段提供了一个 Markdown 编辑器。疯狂阅读这里的内容，确保选择一个好的图片，并在编辑器中尝试一些降价功能。如果需要内容，我推荐一些 [Cat Ipsum](http://www.catipsum.com/index.php) 。
5.  **发布**！Contentful 将保存您的内容。

现在我们已经有了一些生活在 Contentful 中的基本内容，让我们扩展一下基本的 Gatsby starter，把它从云中取出来放到我们的本地浏览器中。

# 把盖茨比接到 Contentful，让我们得到 GraphQL

成为优秀开发人员的第 56 个关键是永远不要错过流行文化的双关语。现在我们有了生活在别人电脑里的内容，一个运行在我们浏览器里的盖茨比网站，我们可以把两者连接起来。在你的 Contentful 的空间仪表盘中，会有一个写着“使用 API”的按钮。该页面将显示您的空间 ID 和访问令牌。这就是我们获取内容所需要的一切，插件将处理其他一切。

在 Gatsby 项目的根目录下，创建一个名为`.env`的文件，并用内容丰富的键填充它:

```
spaceId=**********
accessToken=***************************
```

现在，在你继续之前，**确保你的** `**.gitignore**` **文件有** `**.env**` **，这样你就不会提交这个文件**。对于生产密钥，我们将在后面进行配置。目前，我们只是在本地获取内容。现在我们将开始为 Gatsby 添加一些配置。说到 Git，因为我们只创建了一个 Gatsby 站点，而不是一个存储库，所以让我们花点时间运行一下`git init`。

# 快速回顾和状态检查

到目前为止，我们已经完成了以下步骤:

1.  全球安装在我们的机器上，从 Gatsby Hello World starter 创建了一个新项目，并安装和运行了它。
2.  注册了 Contentful 的免费计划，做了一些内容。
3.  将我们的钥匙添加到`.env`。
4.  跑去`git init`创建一个回购。

现在，运行`git status`并验证`.env`没有被跟踪。我们应该在状态报告中看到的是:

```
On branch masterNo commits yetUntracked files:
  (use "git add <file>..." to include in what will be committed).gitignore
 LICENSE
 README.md
 package-lock.json
 package.json
 src/nothing added to commit but untracked files present (use "git add" to track)
```

# 安装 Gatsby 插件并与 Contentful 交谈

现在我们可以进入盖茨比的第一个阶段了。要添加配置，我们可以在项目根目录下创建一个名为`gatsby-config.js`的文件，在该文件的`module.export`对象中，我们可以指定插件。我们需要添加一些简单的依赖项来继续:

```
npm i -S dotenv gatsby-source-contentful
```

在我们的`gatsby-config.js`文件中，我们将为我们的开发环境指定插件和凭证，如下所示:

现在，如果一切顺利，您应该能够使用 GraphQL 进行查询，与您的内容丰富的 API 进行对话。如果您的终端运行的是 Gatsby 服务器，您可以尝试在操场中查看[这个查询。您可以看到返回的对象的结构很像 GraphQL 查询。如果你没有跟着编码，这里是查询:](http://localhost:8000/___graphql?query=%7B%0A%20%20contentfulHomePage%20%7B%0A%20%20%20%20title%0A%20%20%20%20date%0A%20%20%20%20content%20%7B%0A%20%20%20%20%20%20content%0A%20%20%20%20%7D%0A%20%20%20%20image%20%7B%0A%20%20%20%20%20%20file%20%7B%0A%20%20%20%20%20%20%20%20url%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A)

```
{
  contentfulHomePage {
    title
    date
    content {
      content
    }
    image {
      file {
        url
      }
    }
  }
}
```

第一次这么做的时候，我完全不知道`contentfulHomePage`键是从哪里来的。我知道我想要其中的`homePage`部分，但是谢天谢地，Gatsby 中的 GraphQL playground 已经自动完成了你可以访问的所有内容。按(Mac) Ctrl + Space 并键入`home`并注意显示的选项。从那里我知道了我的字段的键，`title, date, content, image`并使用 CMD +单击字段名来探索它们的属性。让所有这些信息唾手可得对于发现 Contentful 为您的 Gatsby 站点提供了什么非常有用。

现在我们可以在操场上看到一些返回的数据，让我们更改位于`src/pages/index.js`的文件来显示这些数据。为了在屏幕上显示这些内容，我们将使用 Gatsby 的`StaticQuery`组件来发送这个查询，并通过`render`道具提供我们的 JSX 内容。我将呈现的内容包装在 React 片段中，并使用一些析构来公开值。

您可能会注意到`content`显示为 raw Markdown。为此当然有一个 Gatsby 插件，它稍微改变了查询。因为我们很快就要把这些内容放到互联网上了，所以我会把它添加进去，你可以在这个 diff 的源代码[中查看。我们把这个放到网上吧！](https://github.com/cwlsn/gatsby-simple-contentful-starter/commit/f77d7cc432fe8bf29094cbee772f407fb133aa3d)

# 部署到网络

如果你还没有，使用你的 GitHub 帐户注册 Netlify。您应该会在仪表板上看到一个按钮，上面写着“来自 Git 的新站点”。点击它！按照步骤允许 Netlify 访问您的回购。您可以选择特定的，或允许它访问所有当前和未来的回购。

当您选择您的 repo 时，它可能会为您填充配置选项，因为 Netlify 已经确定您正在与 Gatsby 合作。让它展开，让它发挥魔力一分钟左右。

但是等等…构建失败了，因为我们还没有告诉 Netlify 我们的环境变量。我们为本地工作创建的`.env`文件没有得到部署，所以在 Netlify 中跟踪站点设置>构建&部署>构建环境变量。将您的`spaceId`和`accessToken`与之前的值相加。

然后，返回部署选项卡，单击“触发部署”按钮，并将缓存复选框留空。如果您一直这样做，您应该有一个很长的 Netlify 子域，其中包含您的由 Gatsby 生成的内容丰富的代码！

# 自动获取内容更新

还有一件事！由于 Gatsby 网站是静态的，这意味着你的内容更新不会自动显示。当您进行内容更新时，您需要触发一个构建来重新查询和重建所有内容。幸运的是，或者可以预见的是，这很容易设置。

回到我们在 Netlify 中添加环境变量的同一部分，这是一个生成 Web Hook URL 的部分。创建一个新的构建钩子，我称之为“Contentful ”,并将结果 URL 复制到你的剪贴板。确保不要接受他们提供的全部`CURL`命令。

现在，在网站挂钩设置下的 Contentful 中，你会注意到右边有一堆模板。单击 Netlify 模板，粘贴您的 URL 并保存。

# 现在我们有什么？

Netlify 使用 Contentful 提供的数据部署并托管了一个 Gatsby 站点。当您更新数据时，将会触发一个新的构建来保持您的站点最新。

这听起来可能很多，但是如果您能够跟随这里并在 Netlify 上获得一些东西，您就是一个认证的已发布 JAMstack 成员！欢迎来到网络开发的未来。

# 后续步骤

现在你已经把这些都连接起来了，你可以得到数据了，但是我们没有添加太多的样式或者其他东西。就我个人而言，我将使用一些有趣的附加功能来改进我用这个过程开发的 starter，并将其提交给 Gatsby Starter 库。让我在 Twitter 上知道[这个过程对你来说怎么样！](https://twitter.com/_cwlsn)