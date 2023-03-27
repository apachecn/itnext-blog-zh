# React 项目架构

> 原文：<https://itnext.io/react-project-architecture-641da390ebe7?source=collection_archive---------0----------------------->

![](img/ac9d7df60e011f7598e2030dd5fe137c.png)

我已经用 React 开发应用程序很长时间了，我越来越喜欢它。React 是一个非常棒的用于创建应用程序架构和计划的库。它提供了应用基本软件原则的机会(比如 [SOC](https://en.wikipedia.org/wiki/Separation_of_concerns) ，比如 [SOLID](https://en.wikipedia.org/wiki/SOLID) ..)并保持代码库的整洁，即使我们的项目规模在增长。尤其是在钩子之后，它变得如此美味！

在这篇文章中，我想谈谈如何用 React 创建项目结构和架构。您可以将其视为最佳实践和 React 基础知识的结合。当然，它们不是“规则”或其他东西，你可以继续你想做的，我只是想在心里点燃一些悲伤:)

这将是一篇有点长的文章，但我认为它会有所帮助。此外；我将给出 React Native 的例子，但是你也可以在 web 上使用完全相同的结构 ReactJS。

如果你准备好了，我们走吧！🤟

![](img/c7f743293fa0f68019fd3d970a2ed202.png)

gettyimages.com

# 航行

导航是应用程序的**支柱**。你保持它的整洁和平衡，当新的需求、新的页面出现时就更容易集成，花在“我将在哪里以及如何实现新的变化”上的时间就更少问题。

当你开发一个应用程序时，所有的项目架构都在设计阶段显露出来。所有的问题都像；会有哪些屏幕？它能达到什么目的？如何在应用程序中对页面进行分组？找到他们的答案。至此，您可以创建导航架构了。你可以通过查看屏幕设计来创建整个架构。

如果您的应用程序有不同用途的屏幕，您可以将它们收集到一个单独的堆栈架构中。例如，如果应用程序有主模块，如**配置文件、消息、时间线**；

```
- App
   - ProfileStack
   - MessageStack
   - TimeLineStack
...
...
...- ProfileStack
   - ProfilePage
   - UpdatePreferencesPage
   - AddNewPhotoPage- MessageStack
   - InboxPage
   - NewMessagePage
   - TrashCanPage- TimelineStack
   - TimelinePage
   - PostPage
   - CommentsPage
   - LikesPage
```

你可以创建一个类似这样的结构。

主导航器有**配置文件**、**消息**和**时间线**堆栈。这样，我们的应用程序的主要模块是确定的，它们有单独的子屏幕。

比如说； **MessageStack** 模块**仅与**消息部分相关，明天，如果它需要任何新的屏幕，只更新该部分即可。我们可以从任何屏幕导航到任何地方。反应导航给了我们无限的自由，只是我们应该做好计划。

嵌套堆叠没有限制。具有相似上下文的模块可以聚集到相同的堆栈结构中。比如说；如果设置部分中的通知页包含 4 页中的第 3 页；您可以将它们收集在同一个堆栈中。因为在 **SettingsStack** 上看到带有 **NotificationPreferences** 、 **NotificationDetail** 、 **BlockedAppNotifications** 名称的页面并不是什么好事。听起来他们需要**通知**堆栈。此外，像这样放置它们，意味着我们将用相同的导航思想实现每个新页面。毕竟我们应该坚持某种开发方法，对吗？明天 10 个分页模块来了怎么办？

一个项目因为没有遵循一定的开发方式或者遵循错误的开发方式而夭折。

![](img/9b9fa54d3f8d07a8cc4046d356f655c3.png)

avantelectronics.co.uk

# 成分

当你开发一个模块时，**复杂感**结构或**开放性** **可重用性**结构应该被设计成单独的组件。

使用 React 开发页面或模块时，请始终考虑**划分**。React 给了你这个机会，你应该尽可能地利用它。您当前的组件今天看起来可能很简单，您可能没有想到要划分它，但是在您之后开发它的人，如果继续这样开发它，并且如果该组件增长到 200-300 loc*(代码行)*，修改它将比开发它花费更多的时间。

就像厕所一样，你应该把它留在那里，就像你想找到它一样。

# 那么，什么时候应该分组件呢？

在创建一个应用程序的设计时，选择一个固定的设计原则来吸引眼球。按钮，输入，模态总是有一致的设计，看起来彼此相似。你会看到一个按钮十种不同变化，而不是十种不同的按钮设计。这就是一致性，它在用户的视觉记忆中创建应用程序的签名，而你*(实际上，你应该)*在这些查看设计的时候创建你的一致的组件结构。

比如说；如果有经常使用的按钮设计，您可以创建它的变体并将其存储在**通用组件目录**中。你也可以在同一个目录中存储那些在其他地方没有使用但闻起来像**可重复使用**的组件。

但是，如果有一个组件只使用一个屏幕，最好将它存储在与相关屏幕相同的目录中。我们来举个例子；

如果图形和表格组件在分析屏幕上只使用**并且只使用**，如果分析逻辑会将**完全粘在**上，那么最好将它保存在同一个目录中。因为相互需要的模块应该彼此靠近。但是在这个例子中，列表模式和按钮组件可以存储在通用组件中，并从那里进行调用。他们因此而创造。

然后，我们的文件目录会是这样的:

```
- components
   - Button
      - Button.tsx
      - Button.style.ts
      - Button.test.tsx
      - Button.stories.tsx
      - index.ts
   - ListModal
      - ListModal.tsx
      - ListModal.style.ts
      - ListModal.test.tsx
      - ListModal.stories.tsx
      - index.ts
...
...
- pages
   - Analyze
      - components
         - AnalyzeGraph
            - AnalyzeGraph.tsx
            - AnalyzeGraph.style.ts
            - AnalyzeGraph.test.tsx
            - AnalyzeGraph.stories.tsx
            - index.ts
         - AnalyzeDataTable
            - AnalyzeDataTable.tsx
            - AnalyzeDataTable.style.ts
            - AnalyzeDataTable.test.tsx
            - AnalyzeDataTable.stories.tsx
            - index.ts
      - Analyze.tsx
      - Analyze.style.tsx
      - index.ts
```

那个。与分析模块相关的组件和**只为其服务**位于该模块附近。

*注意:在命名时，我认为将相关的模块名作为前缀是更好的选择。因为您可能需要在完全不同的模块上使用另一个图形和表格组件，并且如果您只给出* ***数据表*** *作为名称，您可能会有十个不同的数据表组件，并且您可能很难找到哪个组件在哪个模块上使用。*

# 第二种方式:造型阶段

编写干净代码的最主要的基本原则是给变量和值起一个正确的名字。风格也是我们的价值观，他们应该正确命名。在为组件编写样式时，您给出的正确名称越多，您编写的可维护代码就越多。因为以后会继续开发它的人，会很容易找到哪种风格属于哪里。

如果在命名样式时频繁使用相同的前缀，则应该将该部分视为另一个组件。

所以如果你的 **UserBanner.style.ts** 文件看起来像这样；

```
contanier: {...},
title: {...},
inner_container: {...},
avatar_container: {...},
avatar_badge_header: {...},
avatar_title: {...},
input_label:  {...},
```

你可能觉得你需要一个像 **Avatar.tsx** 这样的组件。因为如果在造型阶段有一个分组，那就意味着一个成长的结构即将到来。没有必要重复 3 或 5 次来将一个结构视为另一个组件。你可以一边编码一边跟着它做推论。

此外；没有规则规定所有组件都应该有逻辑。模块划分得越多，就越能控制它，也就越能编写测试。

让它成为🧳的一个小提示

![](img/b17d9fdfd8017f1ddff711af9f64eee8.png)

atle.artstation.comd

# 钩住

那些**在生命周期**中起到 **作用的**代表一个工作逻辑**的结构，应该抽象成一个钩子。**

为此，他们需要有自己的逻辑，就像定义一样，他们应该在生命周期中。

其主要原因是减轻了总体结构的工作重量，创建了可重用的工作部件。正如我们创建定制组件来降低代码复杂性一样；可以用同样的方法创建自定义挂钩。重要的是要确定创建的结构和它的正确工作。

# 我们如何理解我们需要一个定制的钩子？

让我们用一个例子来解释它；

我认为你需要一个关于项目范围的搜索结构。你需要一个**搜索框**组件，它将能够从任何地方使用，并使用 [fuse.js](https://fusejs.io/) 包进行搜索操作。首先，让我们对两个示例组件实现搜索结构。

*(我没有保留太长的代码，但你可以认为三个点部分是组件的一部分)*

当我们看我们的组件时，我们注意到的主要事情是实现了相同的搜索结构，并且可以清楚地看到代码重复。如果在一个结构上有如此多的代码重复，那就意味着有什么地方出错了。

除此之外；当有人打开任何文件，它会想看到只有**和只有**文件名相关的代码。当您打开 **CommentsScreen.tsx** 文件时，您希望只看到注释相关的代码，而不是任何其他分组的逻辑。是的，在示例中，我们的搜索结构与**产品**和**成员**组件以及它们为其工作相关。但是它们代表了一种**它们自己的逻辑**而且，它们可以被转换成可重用的结构。因此，我们需要定制的钩子或组件结构。

回到例子；很明显，状态被用于搜索动作，并且在生命周期中占有一席之地。当用户开始键入搜索输入时，存储在 ***searchKey*** 状态中的字符串以及更新主列表时也会被过滤。

# 那么我们怎样才能设计得更好呢？

我们可以在一个名为**使用搜索**的钩子上收集我们的搜索结构。我们应该创建这样一个钩子，它不依赖于任何模块，具有可重用的结构，可以在任何地方自由使用。

因为我们将使用 fuse.js 进行搜索，所以我们可以发送数据和搜索标准作为输入，我们可以返回搜索结果和搜索功能，这将在以后触发。

然后我们要创建的钩子是。

会是这个。

有了 TypeScript 支持，我们的钩子可以和类型一起使用。有了它，我们可以在使用时发送和接收任何类型的内容。钩子内部的工作流程和我们之前说过的一样，你会在检查代码时看到。

如果我们想在我们的组件上使用它；

现在可以看到，搜索结构是从组件中抽象出来的。代码复杂度降低了，无论何时我们需要一个搜索结构，我们都有一个定制的钩子。

有了它，我们创建了一个更加干净和可测试的结构。

顺便说一下，就像我说的；可以创建依赖于上下文或通用用途的挂钩，就像组件一样。在这个例子中，我们创建了通用定制钩子，但是我们也可以为特定的任务或上下文创建定制钩子。例如，对于特定页面上数据获取或操作，您可以创建自己的钩子，并从主组件中抽象出作业。

我是说；

```
- hooks
   - useSearch
      - useSearch.ts
      - useSearch.test.tsx
      - index.ts
...
...
- pages
   - Messages
      - hooks
         - useMessage
            - useMessage.ts
            - useMessage.test.tsx
            - index.ts
         - useReadStatus
            - useReadStatus.tsx
            - useReadStatus.test.tsx
            - index.ts
      - Messages.tsx
      - Messages.style.tsx
      - index.ts
```

而 **useSearch** 在项目规模上使用； **useMessage** 负责数据获取， **useReadStatus** 用于消息的订阅者读取状态。与组件上的逻辑相同。

那就是**钩子**🔗

> 在**干净的代码上**书；对于一个功能的单一责任需求有这样一个很好的描述。如果我们把它用于 React: **一个组件或者一个钩子应该做一件事，应该只做这件事，而且应该做好这件事。**

![](img/853b929ac1896f90e97359fd90834e58.png)

searchenginejournal.com

# 语境

您应该为不能直接通信但通过内容连接的模块创建**不同的上下文**结构。

上下文不应该被认为像**“整个项目的所有包装”**。当项目复杂性增加时；与逻辑有联系的结构在数量上也在增加，这些部分应该保持相互分离。语境在这些部分之间起着沟通的作用。比如说；如果您需要在消息模块的组件和页面中进行通信；您可以创建 **MessagesContext** 结构，并通过将它包装到 **only** 消息传递模块来创建独立工作逻辑。在同一个应用程序中，如果你有**附近的**模块，你可以找到你周围的朋友，如果它有许多工作部分；您可以创建 **NearbyContext** 并从其他内容中抽象出来。

所以，如果我们需要一个全球性的结构，在任何地方都可以访问；我们不能用上下文来包装主应用程序吗？

你当然可以。

这就是为什么全球状态管理代表。

在这一点上，你应该小心的主要事情是**不要超载上下文**。你不应该仅仅用 AppContext 来包装应用程序，而把所有的状态都放进去，比如用户信息、风格主题和信息。因为您已经为它们创建了工作模块，可以清楚地看到它们是不同结构。

此外；在**任何**状态更新时，上下文更新与其连接的每个组件。

在例子中；您已经在 **AppContext** 上创建了**成员**和**消息**状态，并且您只监听 **Profile.tsx** 上的**成员**状态和 **MessageList.tsx** 组件上的**消息**状态。当你收到一条新消息并更新**消息的**状态时；**个人资料**页面也会进行**更新**。因为它监听了 **AppContext** ，并且有一个与*(实际上不是)*相关的上下文更新。你认为消息和概要模块之间真的有关系吗？为什么有新消息时，个人资料部分会有更新？这意味着不必要的刷新*(渲染，更新，无论你想怎么命名)*当它们像雪崩一样增长时，会导致很多性能问题。

因此，您应该为不同的工作内容创建不同的上下文，并确保整个逻辑结构的安全。甚至更有理由；当应用程序进入维护阶段时，关心任何模块更新的人应该能够容易地选择相关的上下文，并且毫不费力地理解架构。实际上就在这里，干净代码原则的最基本的教导再次发挥作用；我们刚刚提到的**右变量命名**。

当你以正确的方式命名你的上下文时，你的结构也会变得健康。因为看到**用户上下文**的人会知道它应该从这里获取或放置用户信息。它将知道不要从用户上下文中管理关于设置或消息传递的工作。正因为如此，干净代码原则是非常重要的原则。

此外，用户已经打开了一个关于上下文 API 的[问题](https://github.com/facebook/react/issues/15156)；从上下文中监听状态的组件应该只在订阅状态更新时进行刷新，就像 Redux 一样。[丹阿布拉莫夫的这个回答](https://github.com/facebook/react/issues/15156#issuecomment-474590693)其实是很好的概括了 Context API 的工作逻辑。

监听上下文**的组件应该** **必须** **需要那个上下文**。如果您看到一个从上下文中调用的不必要的状态；这意味着要么该状态在上下文中没有位置，要么您设置了错误上下文结构。都是关于你创造的建筑。

当使用上下文时，一定要确保你的组件确实需要你所调用的状态。你就不太可能犯错误。

举个小例子；

```
[ App.tsx ]<AppProvider> (member, memberPreferences, messages, language)
  <Navigation />
</AppProvider>
```

如果我们分开；

```
[ App.tsx ]<i18nProvider> (language)
  <MemberProvider> (member, memberPreferences)  
    <Navigation />
  </MemberProvider>
</i18nProvider>...
...
...[ MessageStack.tsx ]<MessagesProvider> (messages)
  <Stack.Navigator>
    <Stack.Screen .../>
    <Stack.Screen .../>
    <Stack.Screen .../>
  </Stack.Navigator>
</MessagesProvider>
```

那会好得多。正如你所猜测的，我们拆分了 **MessagesProvider** ，但是我们没有将它放入入口点。因为一般访问需要 **i18n** 和**成员**提供者，但是**消息**将只用于消息范围，它将只触发那部分的更新。所以我们可以期望消息上下文更新消息部分，对吗？

# 结论

嗯，我试着用我自己的方式解释一些 React 的命脉问题。我希望这是一篇对读者有益的好文章。

正如我在上面说过的，React 对于创建这种架构来说是一个非常棒的库。当你希望干干净净的时候，它会尽可能地给你提供机会。您可以使用高质量代码库创建有用且性能良好的 web/mobile 应用程序。

如果你有任何反馈，我很想听听。

下一篇文章再见，注意安全！✌

# [🎙](https://www.youtube.com/watch?v=QBK6xymmKHM)

> “干净的代码总是看起来像是由关心它的人写的”
> ― **迈克尔·费哲**

![](img/4abde93b9a7c4867d14834fa0f0b0c92.png)