# 后端和前端的代码生成。何时、为何和如何？

> 原文：<https://itnext.io/code-generation-for-backend-and-frontend-when-why-and-how-a14d1cbbc47b?source=collection_archive---------6----------------------->

![](img/cb0adf76a302b11011b6c9f16c511dad.png)

我想在这篇文章中展示我们公司如何生成后端(和一点前端)代码，为什么我们需要代码生成，以及我们使用的一些代码生成技术。

我们到底产生了什么？这一点都不重要。
*我们如何做到这一点*——人们可能会对在他们的项目中使用 Gradle 以及做类似事情的方式感兴趣。代码生成的动机和生成的来源是什么——这些都是值得讨论的有趣话题..

我们在这里描述 3 种对象，它们是前端-后端交互的所有代码生成项目的源，有时是完整的后端实现。

1.消息——以序列化为 JSON 的形式参与前端/后端交互的对象。

2.端点—由前端调用的 URI，带有对其 HTTP 方法、查询参数、主体类型和 responserequest 的描述

3.实体——持久存在于后端并需要生成 DAO 层的消息，以及用于创建/更新/列出/删除的标准端点

我不是前端专家，但有些事情我知道:

1.在我们公司，frontend 是用 Typescript 编写的，所以我也想为消息生成 Typescript 类

2.对后端的许多需求(至少在控制器层)来自前端人员。

# 后端代码要求

下面是前端的后端要求:

1.  类似 REST 的后端 API
2.  统一响应— JSON，有效载荷在*“数据”*字段中

3.统一错误响应— JSON，消息在*“error”*字段，在开发环境堆栈跟踪中在*“stack trace”*字段

4.*【正确】* HTTP 代码—如果找不到实体，则为 404，如果 json 无效，则为 400，等等。

我这边的后端要求:

1.在一个地方处理错误

2.可以在任何地方中断业务逻辑流，并告诉前端我想发送什么 HTTP 代码和消息

3.有些业务逻辑我想用阻塞方式编写(例如，如果我使用阻塞库)，有些—用异步方式。但是这两者在我的异步框架中都应该工作得很好。

4.后端开发者不要去想请求和响应，HTTP 代码，控制器(如果我用 Spring)或者 Vertx 路由(如果我用 Vertx)，关于事件总线等等。他们应该写一个商业逻辑。

我还想为 Typescript 和 Kotlin 生成并行类，这样我将确保前端发送给后端的正是后端期望接收的对象。没有“未知属性”错误。

# 下面是我们将生成的内容。

我们将以一些假设的 web 应用程序为例，它可以保存和编辑书籍。一种图书馆。

后端技术— Kotlin、Vert.x、coroutines。类似于我在文章[“vert . x 中异步编程的三个范例”](/three-paradigms-of-asynchronous-programming-in-vertx-b01ab24d0927)中展示的东西

为了更有趣，我们将使用 Spring JPA 创建 DAO 层。

我不是说把 Spring 和 Vert.x 混在一起是个好主意(虽然我这么做！)，我只是拿 Spring 来展示实体的代码生成。

# 项目结构。

现在我们将为这一代人创建一个项目。

它将由 4 个模块组成。现在我将在同一个 git 存储库中创建它们，但在现实生活中，每个模块都将被放在自己的 git repo 中，因为它们会在不同的时间发生变化，并且会有不同的版本。

因此，第一个项目包含我们的端点、消息等的注释。

我们将称之为*【元信息】*。

另外两个项目——*codegen*和 *api* 依赖于 *metainfo* 项目。

*api* 包含端点和消息的描述——参与前端/后端交互的类

*codegen* 是代码生成项目(但不是生成代码的项目！)—它包含从 *api* 项目类和代码生成器收集信息的代码。

生成器将通过命令行参数接收所有细节——哪个包采用端点，将生成的文件写入硬盘上的哪个目录，什么是 *Velocity* 模板文件名等等。

所以 *metainfo* 和 *codegen* 是通用项目——你可以在不同的应用程序中使用它们，而 *api* 是特定于特定的 web 应用程序的。

这是我们生成代码的两个项目:

在*前端生成的*中，我们生成了 Typescript 类，它对应于 Kotlin 消息和实体类

以及*后端*——我们的 Vert.x 应用

为了使一个项目(gradle 模块)类对另一个可见，我们将使用一个 gradle 插件在本地 Maven 库中发布工件。

## Metainfo 项目:

标记生成源的注释—端点、消息、实体:

对于 Typescript 类，我们将定义注释，这些注释将标记消息字段，并将被添加到生成的 Typescript 类中:

下面是 metainfo 项目的一个[源代码。](https://gitlab.com/bernshtam-articles/kotlin-codegen-part-1/tree/master/metainfo)

## 项目 api:

注意 *build.gradle* 中的 *noArg* 和 *jpa* 插件，用于生成无参构造函数。

由于缺乏想象力，我们会对端点和实体进行一些疯狂的描述:

下面是一个 api 项目的[源代码。](https://gitlab.com/bernshtam-articles/kotlin-codegen-part-1/tree/master/api)

## Codegen 项目:

首先，我们将定义“*描述符*”—类，我们将通过从对“ *api* ”项目类的反映中接收到的信息来填充这些类:

下面是收集信息的代码:

此外，这个项目包含“主”类、实用程序类等等。你可以在库中看到所有的[。](https://gitlab.com/bernshtam-articles/kotlin-codegen-part-1/tree/master/codegen)

## 在项目*前端生成的*和*后端*中，我们做类似的事情:

1.编译阶段对 *api* 项目的依赖。

2.构建过程对 *codegen* 项目的依赖性。

3.生成模板在 buildSrc 目录中，该目录在 *gradle* 中用于类和资源，用于构建过程，但不用于编译或运行时。也就是说，我们可以在不重新编译 *codegen* 项目的情况下更改生成模板

4.前端生成的编译生成的类型脚本，并将其发布到 npm 包存储库。

5.在*后端*项目中，我们生成*路由器*，它们继承自非生成的抽象*路由器*，后者知道如何处理各种请求。此外，我们生成抽象的*verticals*，您应该通过业务逻辑实现继承它。此外，我们会生成许多样板代码——编解码器注册、事件总线中的地址常量等等——所有这些细节，程序员都不应该考虑。

下面是 [*前端生成*](https://gitlab.com/bernshtam-articles/kotlin-codegen-part-1/tree/master/frontend-generated) 和 [*后端*](https://gitlab.com/bernshtam-articles/kotlin-codegen-part-1/tree/master/backend) 的源代码。

在*前端生成的*中，注意将生成和编译的源代码发布到 npm repo 的插件。为了让它工作，把你的库的 IP 放到 *build.gradle* 中，把你的认证令牌放到*中。npmrc* 文件

下面是生成的 typescript 类的样子:

注意类验证器注释。

在后端项目中，我们还为 Spring 数据 JPA 生成了存储库。我们可以看出 verticle 中的消息处理方法是阻塞的(将使用 Vertx.executeBlocking 执行)还是异步的(使用协程)。

我们可以知道为一个实体生成的 verticle 将是抽象的，我们将覆盖 hooks，它在生成方法之前和之后被调用。

我们自动部署所有的垂直——只需获得所有类型为*垂直*的 beans。

这个框架很容易扩展——例如，我们可以在一个端点上放置一个用户角色列表，并生成一个代码，检查登录的用户是否具有允许的角色之一。

此外，通过改变模板，我们可以生成 akka-http，而不是 Vertx 和 Spring(甚至是 Visual Basic:)

另一个方向——生成更多的前端代码，或者 swagger 或 WSDL 或任何你想要的东西。

那些项目的所有源代码都是[这里](https://gitlab.com/bernshtam-articles/kotlin-codegen-part-1)。

*感谢前端团队伊尔达的帮助。*