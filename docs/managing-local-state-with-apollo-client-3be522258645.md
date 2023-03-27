# 使用 Apollo 客户端管理本地状态

> 原文：<https://itnext.io/managing-local-state-with-apollo-client-3be522258645?source=collection_archive---------2----------------------->

# **序幕**

Web 开发最大的优点和缺点之一是它的模块化方法。一个关键的编程口头禅是选择一些东西(一个函数，一个包)来完成一项工作，并把它做好。这种方法的缺点可能意味着单个项目可能在几十种独立的技术和概念之间周旋，每种技术和概念都专注于特定的东西。

所以选择 Apollo 客户机来处理我的本地状态和远程数据似乎是显而易见的。既然我已经设置了 Apollo/GraphQL 从后端获取数据，为什么还要处理 Redux 的样板文件和习惯用法呢？

虽然本文将讨论如何设置 Apollo 来处理本地状态，但它并不是对该技术的介绍。(这个正统的 [howtographql](https://www.howtographql.com/) 教程就是一个很好的例子)。

注:已完成的回购可以在这里找到[。除了在你陷入困境时为你提供帮助之外，它还充当了一个与 Apollo-Client 一起设置的 React 应用程序，用于开箱即用的本地状态功能，带你上路。](https://github.com/andrico1234/apollo-local-state-starter)

# **文森特·维加&马沙·华莱士的妻子**

我们将从这里的[开始克隆相应的回购协议](https://github.com/andrico1234/apollo-state-blog-repo)。这个 repo 包含一个简单的 react 网站，有侧边栏、标题和正文。它本质上是相当静态的，没有动态内容(..还没)。在本教程结束时，我们将让 Apollo 管理网站的状态。点击侧边栏中的一个项目会改变网站的状态，进而更新标题以显示新数据。

如果你检查`package.json`，你会看到我们只有基本的，加上一些关于我们的包裹设置的额外的包裹。

克隆 repo 后，运行您的标准命令

```
> yarn
> yarn dev
```

安装所有的包并创建一个本地服务器。转到 localhost:1234，你将有希望看到演示网站的辉煌。它现在是静态的，所以点击什么也做不了。

我们首先要做的是让 Apollo 进入我们的项目，所以安装这些包。apollo-client 允许我们配置 apollo 的实例，react-apollo 是允许我们将其集成到 react 应用程序中的驱动程序。由于包裹的问题(我想)，我们也需要安装 graphql。

```
> yarn add apollo-client react-apollo graphql
```

所以创建一个新目录`src/apollo`。打开一个`index.js`文件，添加以下内容

src/阿波罗/index.js

这是初始化我们的 Apollo 客户端，然后我们将通过在我们的`src/index.js`文件中添加以下内容来包装我们的 React 应用程序。

src/index.js

现在，我们已经可以在应用程序中使用 Apollo 了。当我们重新启动我们的开发服务器时，一切都构建好了，但是当我们试图在浏览器中访问它时，我们得到了一个错误。控制台会告诉我们，我们需要为 Apollo 客户机指定链接和缓存属性，所以我们就这么做吧。

```
> yarn add apollo-link apollo-cache-inmemory apollo-link-state
```

前一行向我们的应用程序添加了新的 Apollo 依赖项，而下面的代码解决了我们遇到的控制台错误。因此，返回到`apollo/index.js`并更新它，使文件看起来像这样:

src/阿波罗/index.js

让我们创建一个缓存实例。缓存是 Apollo 的标准化数据存储，它将查询结果存储到一个扁平的数据结构中。(为什么这对我们有好处？).当我们进行 graphql 查询时，我们将从缓存中读取，当我们进行变异解析时，我们将向缓存中写入。

您可以看到我们还向我们的客户端对象添加了`link`。`ApolloLink.from()`方法让我们模块化地配置如何通过 HTTP 发送查询。我们可以用它来处理错误、授权并提供对我们后端的访问。为了本教程的缘故，我们不会做这些，但是我们将在这里设置我们的客户端状态。所以我们创建上面的`const stateLink`，并传入我们的缓存。稍后我们将在这里添加默认状态和解析器。

回到浏览器，你会看到我们可爱的静态网站显示其所有的辉煌。让我们为项目添加一些默认状态，并启动我们的第一个查询。

在 apollo 目录中创建一个名为`defaults`的新目录，并在其中添加一个`index.js`。该文件将包含以下内容:

src/默认值/index.js

我们创建一个对象作为我们站点的默认状态，apolloClientDemo 是我们在查询时想要访问的数据结构的名称。`__typename`是我们的缓存使用的强制标识符，currentPageName 是我们的 header 将用来显示当前页面名称的特定数据项。

我们需要将它添加到我们的`apollo/index.js`文件中

src/阿波罗/index.js

让我们澄清一下。通常`import default`是 JavaScript 关键字，你可以用它从文件中导入`export default`生成的代码，但是巧合的是，我们从`./defaults`中导出的对象的名字叫做`defaults`。将该导入行视为普通的 ol' named import。

这样一来，我们就可以进行查询了！

# **金表**

将以下包添加到您的项目中

```
> yarn add graphql-tag
```

创建一个新目录`src/graphql`，并在其中创建两个新文件`index.js`和`getPageName.js`。graphql 目录将容纳所有的查询和变化。我们将通过编写以下代码在`getPageName.js`中创建我们的查询:

src/graphql/getPageName.js

所以我们导出两个变量，查询和选项。如果您以前使用过 graphql，那么这个查询看起来会很熟悉。我们对 apolloClientDemo 数据结构进行查询，只检索 currentPageName。您会注意到我们已经在查询中添加了`@client`指令。这告诉 Apollo 查询我们的本地状态，而不是将请求发送到后端。

下面您会看到我们正在导出一些选项。这只是定义当我们将结果映射到道具时，我们希望数据看起来是什么样子。我们正在析构 graphql 响应，并将其发送到我们的视图，因此它看起来像这样:

```
props: {
  currentPageName: ‘Apollo Demo’,
}// and not thisprops: {
  data: {
    apolloClientDemo: {
      currentPageName: ‘Apollo Demo’,
    }
  }
}
```

转到`graphql/index.js`文件并导出查询，如下所示

src/graphql/index.js

同样，虽然这对于一个小的演示/项目来说不是完全必要的，但是如果你的应用程序变大了，这个文件会很方便。将您的查询从一个集中的位置导出，可以保持一切有序和可伸缩。

添加到您的 Header.js:

src/components/Header.js

这将动态显示当前页面的标题。现在，它将只显示默认值，因为我们还没有添加变异。但是你可以改变默认状态，网站会反映出来。

现在剩下要做的就是通过点击侧边栏条目来改变 Apollo 缓存中的数据。

![](img/35d0214d0feb513029a8ad76276c43d6.png)

一个令人耳目一新的形象，打破了文字。杰夫·谢尔登

# **儒勒、文森特、吉米&狼**

当处理突变时，事情变得有点复杂。我们不再只是从 Apollo 存储中检索数据，还会更新数据。突变的架构如下:

>**用户**点击侧边栏项目

>**将**变量发送至突变

>**用变量触发**突变

>**将**发送到阿波罗的实例中

>**找到**对应的解析器

>**将**逻辑应用于阿波罗商店

>**将**数据发送回割台

如果这很难记住，那就用这个用助记符生成器创造的方便的助记符:城市老年牧神庄严地摸索着不忠的阿斯兰。(简单……)

首先创建一个文件`graphql/updatePageName.js`。

src/graphql/updatePageName.js

并像我们处理查询一样导出它。

src/graphql/index.js

你会注意到一些关于突变的不同之处。首先，我们将关键字从 query 改为 mutation。这让 graphql 知道我们正在执行的动作的类型。我们还定义了查询的名称，并向传入的变量添加了类型。在这里，我们指定了我们将用来执行更改的解析器的名称。我们还传递变量并添加了`@client`指令。

与查询不同，我们不能只是将突变添加到我们的视图中，然后期望发生任何事情。我们将不得不回到我们的阿波罗目录，并添加我们的解决方案。所以继续创建一个新目录`apollo/resolvers`，以及文件`index.js`和`updatePageName.js`。在`updatePageName.js`内添加以下内容:

src/阿波罗/resolvers/updatePageName.js

天啊，这个文件里有很多有趣的事情。幸运的是，这一切都非常符合逻辑，并没有给我们之前所做的事情增加许多新概念。

默认情况下，当解析器被调用时，Apollo 会传入所有变量和缓存。第一个参数是一个简单的“_”，因为我们不需要使用它。第二个参数是变量对象，最后一个参数是缓存。

在对 Apollo 存储进行更改之前，我们需要检索它。因此，我们发出一个简单的请求，从存储中获取当前内容，并将其分配给 previousState。在数据变量内部，我们创建了一个新的对象，其中包含了我们想要添加到存储中的新信息，然后我们向其中写入数据。你可以看到我们已经将之前的状态扩展到了这个对象中。因此，只有我们明确想要更改的数据才会得到更新，其他的都保持原样，这就防止了 Apollo 不必要地更新那些数据没有更改的组件。

注意:虽然对于这个例子来说这不是完全必要的，但是当查询和突变处理大量数据时，它非常有用，所以为了可伸缩性，我保留了它。

同时在`resolvers/index.js`文件中…

src/apollo/resolvers/index.js

这是 Apollo 期望的对象形状，当我们在`apollo/index.js`中将解析器传递给 stateLink 时:

src/阿波罗/index.js

剩下要做的就是将变异添加到我们的侧栏组件中。

src/components/Sidebar.js

就像我们的解析器文件一样，这个文件中有很多内容，但没有什么是我们以前没有见过的。我们导入变异，使用 compose 方法将 graphQL 变异添加到我们的 props 中。然后我们在 handleClick 中触发它。我们通过`<li>`元素传入页面的新名称，如果一切正常，您应该能够运行您的开发服务器并单击侧栏项目。

# **后记**

万岁！希望一切顺利。如果你卡住了，然后检查回购[在这里](https://github.com/andrico1234/apollo-local-state-starter)。它包含了所有完成的代码。如果你想在下一个 React 应用中使用本地状态管理，那么你可以派生这个 repo 并从那里继续。

我还想在本教程中介绍更多内容，比如异步解析器(想想 Redux thunk)、类型检查/创建模式以及更多关于缓存的乐趣。

我真的希望这个教程对你有用，我也想大声说出[萨拉·维埃拉的 youtube 教程](https://www.youtube.com/watch?v=2RvRcnD8wHY)，因为它帮助我理解了阿波罗客户端。如果我没有做好我的工作，让你挠头，那么请点击链接。最后，请随时在社交媒体上联系我，我是一个超级音乐和技术迷，所以请和我聊聊极客。

## 感谢阅读！

如果你有兴趣邀请我参加会议、meetup 或作为演讲嘉宾参加任何活动，那么你可以在 [twitter](https://twitter.com/andricokaroulla?lang=en) 上给我发 DM！

我希望这篇文章教给你一些新的东西。我定期发帖子，所以如果你想了解我的最新消息，你可以关注我。记住，你按住拍手按钮的时间越长，你能发出的拍手声就越多。👏👏👏

## 你也可以看看我下面的其他文章:

[*用 React.lazy()*](http://Add a touch of Suspense to your web app with React.lazy()) 为你的 web app 增添一丝悬念

[*如何使用 Apollo 全新的查询组件管理本地状态*](https://medium.com/@andricokaroulla/updated-for-apollo-v2-1-managing-local-state-with-apollo-d1882f2fbb7)

[*不用等到节假日，现在就开始装修*](https://codeburst.io/no-need-to-wait-for-the-holidays-start-decorating-now-67b9dabd60d7)

[*用阿波罗和高阶组件管理本地状态*](/managing-local-state-with-apollo-client-3be522258645)

[*React 大会喝酒游戏*](https://medium.com/@andricokaroulla/the-react-conference-drinking-game-7a996bfbef3)

[*使用 Lerna、Travis 和现在的*](https://codeburst.io/develop-and-deploy-your-own-react-monorepo-app-in-under-2-hours-using-lerna-travis-and-now-2b140d647238) ，在 2 小时内开发和部署您自己的 React monorepo 应用程序