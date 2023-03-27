# Redux-Observable 会解决你的状态问题。

> 原文：<https://itnext.io/redux-observable-can-solve-your-state-problems-15b23a9649d7?source=collection_archive---------1----------------------->

![](img/0c3564055976fffcf39757ee7c29f4c8.png)

## 解决常见的异步状态问题

每个使用 Redux 的人最终都会发现自己需要一种方法来处理异步动作。有很多选择，最大的名字是 [Redux-Thunk](https://www.npmjs.com/package/redux-thunk) ，其次是 [Redux-Saga](https://www.npmjs.com/package/redux-saga) 和 [Redux-Observable](https://www.npmjs.com/package/redux-observable) 。

Redux-Observable 是一个非常强大的使用 RxJS 的 Redux 中间件。使用 **Redux-Observable 需要知道或学习 RxJS** ，这本身就是一个大麻烦。当我第一次开始使用这些工具时，花了相当多的时间和大量的实验来找出一个好的过程。

我没有强迫其他人经历同样的事情，而是创造了一堆有用的入门史诗，你可以用在你的项目中。虽然这些将是**更简单的例子**(相对陈述)，但是你应该能够**弄清楚 RxJS 的基础**以及它如何与 Redux-Observable 联系起来供你自己使用。

# 什么是可重复观测的

Redux-Observable 抽象出与 Redux store 的连接，并为您处理订阅，因此您不必担心监听动作或自己调度它们。

在一个非常基本的层面上，当它同时订阅你所有的史诗(可观察的管道)时，它调用 Redux 的`dispatch`函数，就像这样:

```
allYourObservablesTogether
.subscribe(store.dispatch)
```

因为 Redux-Observable 只调度动作，所以如何处理副作用可能不是很明显。

# 副作用

先说副作用。有时你需要连接到第三方库，比如聊天客户端、登录平台或分析服务。为此，您几乎总是要调用他们的非 Redux 库。由于 Redux 默认情况下只调用通过 reducers 影响状态的动作，您可能想知道如何将第三方库与 Redux 结合起来。这就是 Redux-Observable 及其史诗的用武之地。

对于这个例子，我将与第三方服务通信，并告诉它只有在我存储了用户信息之后才开始登录；例如，在成功登录后。

我们从通过 Redux-Observable 赋予我们史诗的`action$`可观测性开始。我们想使用`ofType`来检查进来的动作是否有匹配的类型。

`ofType`引擎盖下，真的是:

```
filter(({ type }) => (
  type === 'USER::UPDATE_USER_INFO'
))
```

`ofType`就像在状态减速器中检查`action.type`。发生的任何行为都将通过这个`ofType`检查，要么继续，要么停止。

因为日志服务需要我们的用户数据以某种方式格式化，我们将它映射到一个新的对象，然后使用`tap`将它传递给日志服务。

`tap`是在你想做副作用比如`console.log`或者挂钩第三方工具比如`updateLoggingService`的时候使用的。它与 Promise 的`.then()`完全相同，只是它不会改变管道的状态。

这是一个相当于`tap`的承诺:

```
Promise
.resolve(value)
.then(value => {
  updateLoggingService(value)

  return value
})
```

当你想调试你的管道时，它也是很棒的。在创作一部新的史诗时，这是无价的。

因为所有史诗都必须发出一个动作，而我们没有这样做，所以我们用`ignoreElements`结束我们的管道。我们已经把`ignoreElements`放在了我们的可观察对象的末端，这样它一通过`tap`就会停止管道。不会发出任何动作。

# 执行一次

除了副作用之外，有时您可能希望一个操作只执行一次。在本例中，这将是您想要开始现场聊天的时间。

这个 epic 看起来和我们之前的非常相似，除了它只使用`take(1)`启动一次聊天客户端。这意味着，无论这个动作被调用多少次，它只执行第一次。

可能有一个单独的动作用用户信息更新聊天客户机，就像我们在前面的例子中一样，但是这个 epic 只关心启动聊天客户机。因为我们正在调用第三方服务的函数，所以我们真的只想调用一次。再次调用它会导致第三方库出错。

# 立即执行行动

当你的应用程序启动时，你可能想要立即执行一个动作，而不是等待一个动作的到来。将 epic 与 Redux-Observable 的初始化紧密耦合很可能是个坏主意，但是它消除了调用一个动作来执行 epic 的需要。

一个简单的例子是将应用程序的当前版本记录到控制台，这样您就可以确保应用程序的部署版本与您预期的版本相匹配。

我提供了 3 个单独的例子，但是您可能只想在您的应用程序中使用一个。

注意我们没有使用`ofType`。没有发出 Redux 操作来触发应用程序版本日志记录。虽然这一开始看起来是个好主意，但这也意味着我们无法控制它的执行。我建议不要像这样立即行动，除非你真的有需要。不管怎样，我们可以用这个例子来学习更多的 RxJS 模式。

在版本 1 之前，Redux-Observable 没有正确处理立即执行，需要使用`timer(0)`。`timer`功能需要一段时间，并一直等到这段时间过去后，才通过管道发出一次`0`。明确地说，不管你向`timer`函数传递了什么，它都会通过管道传递`0`。

在引擎盖下，它看起来像这样:

在另外两个例子中，我使用了`of`和`EMPTY`。`of`会散发出它所赋予的任何东西。如果你给它传递更多的参数，这些参数将会一个接一个地发出。在这种情况下，我们不在乎我们发出什么，只要我们发出什么。我们甚至可以发出`null`，但是我们选择发出`appVersion`是为了让我们的代码更干净一点。

`EMPTY`是一个常量可观测值，订阅后立即完成。这就是为什么我们可以使用`defaultIfEmpty`。所发生的是空调用`observer.complete()`，然后`defaultIfEmpty`中的值沿着管道向下传递，只要它在你的管道中。这就是为什么我们不直接调用`console.info(appVersion)`的原因。

# 更新状态

可能你会遇到的 Redux-Observable 的最基本的例子是更新状态。这并不完全显而易见，但是用 epic 更新 Redux 的状态需要监听一个动作并执行另一个动作。

在这个例子中，当用户登录时，我们有一个认证平台分派一个`UPDATE_USER_INFO`动作。有几种方法可以处理这个问题，比如直接在我们的 reducer 中进行所有的转换，但是我们可以让我们的 epic 来处理它。

首先，我们`map`(转换)我们的数据以匹配我们将如何存储它，然后我们`map`我们的 reducer 监听存储的动作。

现在，我们的`userInfoReducer`可以这么简单:

因为在 epics 中使用可观察的管道比使用 reducer 可以做更多的事情，所以使用超级简单、可重用的 reducer 并利用 epics 进行所有数据转换和业务逻辑是一个很好的过程。这样，你可以专注于 Redux-Observable 中的复杂逻辑，而不是复杂化你的 reducers。

# 用行动利用商店

很多时候，您会希望利用状态中的某些东西来执行其他操作。这就是我们得到 Redux-Observable 的真正力量的地方。监听动作是一回事，将应用程序的全部状态放在一个地方并使用它来执行其他动作是另一回事。

我有过不少这样的情况，我真的想要一些州外的东西，但也许它还不在那里。也许我想等到商店里有东西的时候再继续我的史诗。有几种方法可以解决这个问题。我将回顾几个简单的例子并解释为什么你会使用它们，但是我将把复杂的留到本文的最后。

在第一个例子中，我们在做任何事情之前等待`UPDATE_AUTH_INFO`动作被分派。我们的最终目标是在我们通过身份验证后发送`closeModal`动作。

像我们其他的观察对象一样，我们在倾听一个动作，然后进行映射。在这种情况下，我们甚至不关心`UPDATE_AUTH_INFO`的有效载荷是什么，因为我们映射到封闭的`state$.value`。`.value`道具并不存在于大多数可观测物体上，但是`state$`是一个`BehaviorSubject`，这意味着你可以在任何时候抓取当前值。

接下来，我们从我们的状态中获取当前的`authInfo`对象，并从`pluck`中取出`isAuthenticated`道具。这与做以下事情是一样的:

```
map(authInfo => (
  authInfo
  .isAuthenticated
))
```

接下来是我使用的一个技巧，用更少的代码来验证值。通过将`Boolean`构造函数传递给`filter`，只有真值才会继续。注意`0`和`''`是假的。

一旦我们从我们的状态中确定用户通过了身份验证，我们就可以执行一个新的操作符:`mapTo`。这就像一个 map，但是它存储它在运行时接收的值，而不是像 map 一样在管道中执行一个函数。

从这里，我们像往常一样继续，将我们的管道值映射到一个将由 Redux-Observable 调度的动作。

# 直接监听状态更新

有时候你会想直接听 state$的。这可能非常有效，并且解耦了您需要知道哪个动作更新状态的哪个部分的 epics。

在这个例子中，我们通过`distinctUntilChanged`操作符依赖 Redux 存储的不变性。该操作符保留以前的状态，并将其与管道中的当前值进行比较。如果它们匹配，它就在那里停止管道。如果值已更改，则继续通过管道。

# JavaScript 事件

有时您需要挂钩 JavaScript 事件，比如监听对`localStorage`的更改、来自`iframe`的通信或者与第三方库交互。这个问题经常出现，因为第三方几乎从不使用 observables。

我们将监听`window`大小的变化，因为我们将创建一个`@media`查询的定制实现。

我们使用`window`创建一个`fromEvent`可观察对象，以及我们正在监听的事件类型:`resize`。这和做一个`window.addEventListener`是一样的，除了它被很好地包装成一个可观察对象。

您甚至可以收听其他类型的事件，如`on`:

```
someThirdPartyLibrary
.on('ready', () => {})
```

使用`fromEvent`时，看起来是这样的:

```
fromEvent(
  someThirdPartyLibrary,
  ‘ready’,
)
.pipe(
  // This is your callback pipeline
)
```

一旦这些响应来自我们的`resize`事件，我们将把它们限制在每秒 60 帧。这意味着调整大小通知只有在每秒 60 帧的时间段内才会出现。这也意味着我们每秒最多只有 60 个调整大小事件。

作为澄清，我们确实应该在 RxJS 中使用`requestAnimationFrame`调度程序，但是我们在这个例子中使用了我们自己的调度程序，因为解释起来要简单得多。

# AJAX 调用

如果您直接挂钩到 API 端点，而不是通过 GraphQL 服务器，那么您可能希望通过 Redux-Observable 来管理来自 AJAX 调用的状态更新。

这就是依赖的由来。与其直接拉入`ajax`，不如通过中间件函数注入到你的 epic 中。

这使得调试更加容易。如果你像我一样渲染服务器端，你会想知道 RxJS 的`ajax`函数只在你有 DOM 时才起作用，因为它在幕后做了一个`XMLHttpRequest`。你也可以把它换成类似`axios`或`fetch`的东西。

RxJS 中有一个操作员将与`ajax`协同工作，那就是`switchMap`。当使用`switchMap`时，它完成先前的`ajax`可观察，取消请求。这对于异步操作来说非常好，因为这意味着您可以轻松地最小化竞争条件。

它看起来有点像这样:

依赖性是我们史诗创造者的第三个论点。您的访问令牌或 id 令牌中应该存储有权限；但是，您可能需要使用相关的 API 来检查您的用户能够做什么，以确保 UI 不允许他们拥有比平时更多的权限。在这种情况下，我们向 API 请求所有用户权限。

有了这部史诗，还有更多的事情要做。在我们进行 AJAX 调用之前，我们需要使用`state$.value`从商店中获取`accessToken`。

现在我们看到一个新的操作符:`switchMap`。这个操作符，以及其他一些类似的操作符，是迄今为止最重要的操作符，因为它们允许您在一个管道中将可观察对象链接在一起。在我们的例子中，当您调用`ajax`函数时，它将返回一个可观察值。

通常，我们可以在调用`ajax`后立即返回，并将我们的操作符添加到主管道中。相反，我们将直接从`ajax`中分离出来。虽然它在这个例子中没有意义，但它与我们最终如何处理错误有关。

在我们成功的 AJAX 调用之后，我们将包含权限列表的属性映射到动作。

# `Cancelling AJAX Calls When Using mergeMap`

当一次获取所有权限时，你可能只需要一个史诗就可以了。基于您的架构，您可能正在使用 React，并且每个组件可能知道它需要哪些权限。在这种情况下，特别是如果你的应用程序有大量的权限，你可以在需要的时候获取一个权限，而不是获取整个列表。

你不想为每一个可能的许可创建一个单独的史诗。相反，你会想要创造一个利用`mergeMap`的通用史诗。为了避免这些情况下的竞争条件，您将需要`takeUntil`操作符。

让我们从头开始。我添加了一个助手函数，根据给定的`permissionType`过滤出权限动作。这将在史诗的后期派上用场。

这部史诗的大部分内容都是一样的，除了我们现在用我们的动作传入一个`permissionType`,并需要将它与状态外的一个值相结合。我使用了之前的一个可观察对象:`of`，它接受传递的值并通过管道传递。这样就可以`pipe`掉`of`可观察，抢接入令牌，像以前一样`switchMap`到`ajax`。

当您使用`mergeMap`而不是`switchMap`时，它将创建一个新的可观察对象并订阅它。如果不小心的话，这个操作符可以单独造成内存泄漏。幸运的是，`ajax`只触发一次并在之后完成，但是如果你想要和我们之前使用`switchMap`时一样的取消行为，我们将会使用`takeUntil`。

`takeUntil`给定一个可观察值，从不通过管道推送任何东西。当给定的可观察对象发射时，它以与`take`相同的方式完成当前的可观察对象。

在我们的`takeUntil`中，我们正在监听另一个`FETCH_PERMISSION`动作，它有一个匹配的`permissionType`。如果是这样，我们将完成取消 AJAX 调用的 observable。这允许另一个开始，而不会导致可能的竞争情况。

# 延迟 AJAX 请求

您可能希望限制一次发出的 AJAX 请求的数量。例如，您可能正在上传文件，并且希望两个一组地上传，而不是一次上传十个文件。

使用`mergeMap`，它几乎肯定会一次发送所有十个文件。另一方面，使用`concatMap`会将其限制为一次一个项目。如果您想将它限制为一次两个项目，请使用带有第二个参数`2`的`mergeMap`。如果你把所有东西都放在同一个史诗中，这真的很难理解。

更好的方法是像这样使用`map`和`mergeAll`:

这个例子非常复杂，它包含了很多我们还没有看到的概念。例如，将`permissionTypes`包装在`from`中实际上是通过管道将每个`permissionType`作为管道中的一个单独的值。我们将这些映射到一个全新的可观察对象中，我们`mergeAll(2)`将它保持为一次两个项目被合并映射。这限制了我们在 API 上的负载，并确保我们的应用程序可以轻松地处理越来越多的权限。除了这些变化，它几乎和以前一样的史诗。

一个主要的区别是我们如何处理`takeUntil`。当`FETCH_PERMISSIONS`被执行时，我们现在需要遍历每个`permissionType`并在取消之前查看是否有与我们的动作调用匹配的。

# 等待合并映射的可观测量

现在，如果您想等到所有这些合并映射的可观测量完成，但同时启动`storePermission`动作，会怎么样呢？为此，我们将使用`toArray`和`multicast`。

我实际上已经在一个实际的 epic 应用程序中使用了这个功能，它控制文件上传，所以它有它的用处。

这个`multicast`解决方案相当神奇，但也很难理解。如果你想改变它的行为，你真的需要意识到它在做什么，但是在这篇文章中，我将只在一个高层次上介绍当前的实现。

`Subject`有一些特殊之处，它既是一个可观察对象，又是一个观察者。因此，我们正在创建一个单独的`Subject`可观察对象，它将在管道中的这一点接收相同的输出，并连接到源可观察对象(在本例中为`from(permissionTypes)`)。我们将把源可观察对象与其自身合并，并把合并后的可观察对象分离出来。

我们将这个`multicast`操作符放在了`mergeAll`之后，因此它将接收每个`permissionType`发出的响应。一旦所有权限类型都通过了管道的这个区域，`toArray`将发出它收集的 AJAX 响应数组。因为我们不关心这些响应，因为我们已经单独处理了它们，所以我们映射到`finishedStoringPermissions`动作。

这里有一个简单得多的例子，您可以将它复制粘贴到任何 JavaScript 领域:

注:`range(1, 4)`与`from([1, 2, 3, 4])`相同。

# 捕捉 AJAX 错误

我们如何处理 Redux-Observable 中的错误？就像我们对待 RxJS 一样…有点。

我们使用`catchError`来捕捉任何不成功的 AJAX 响应。否则，它们会被 Redux-Observable 吞噬，你永远也不会知道。RxJS v6 解决了吞咽错误，但是您可能还是想处理这个错误。

在这个例子中，我们打开了错误模式，并将错误和堆栈跟踪一起传递给它。再次注意`of`可观察值的使用。`catchError`采用返回可观察值的函数。不这样做将导致另一个错误，所以如果你使用`catchError`，请确保你将启动一个动作。

我们还必须把`catchError`放在`ajax`上，因为否则它不会被捕捉到。这就是我们为什么直接关闭`ajax`的原因，因为`catchError`返回的动作仍然通过管道，就好像`catchError`从未被调用过一样。

# 一般错误捕获

像 AJAX 错误一样，您会希望将`catchError`放在相当多的史诗中。在本例中，我们将通过调用一个错误操作将它们发送到我们的日志记录软件。这样，这些错误也可以在 Redux Devtools 中找到。我们还可以在代码库中的其他地方创建一个 epic，简单地监听这个错误动作，并使用副作用处理与第三方日志服务的集成。

如果这个例子是在服务器端执行的，`window`就是`undefined`；因此，你通常会中断你的应用程序，但是使用 Redux-Observable，你只需点击`catchError`并继续运行，就像什么都没发生一样。

在这个例子中，我们有目的地确保仍然调用`storeCurrentUrl`，但是因为我们没有`window`，所以我们将传递一个空字符串。

不仅如此，我们还同时调用`logError`来通知我们的日志记录服务，因为这个有用的可观察对象`merge`。正如您可能已经收集到的，`merge`允许我们同时执行多个动作，当我们把它们作为单独的参数传入时。

因为它将要出现，你应该知道还有一个`merge`操作符。我会说你可能会完全避免使用操作符版本，但是由于 RxJS 的`finalize`操作符在处理 Redux-Observable 时存在缺陷，你最终会需要它。

在一般的`catchError`例子中，我简单地看了一下`merge`，但是它有更多的用途。通常，在进行 AJAX 调用之前，我会启动一个加载指示器。AJAX 调用完成后，我将触发更多的动作，比如存储数据和停止加载指示器。

我们的例子越来越复杂，所以我们保持同样的权限史诗。

使用`merge`，我们能够在 AJAX 调用之前启动加载指示器，并在调用之后立即停止。遗憾的是，使用 Redux-Observable 没有真正的方法来听出成功和不成功。这是因为 RxJS 中的`finalize`方法像`tap`。当可观察对象完成或出错时，它调用一个动作，实际上它不执行任何事情。所以当你需要一个副作用的时候你可以用它，但是当你想在场景一或场景二中调用一个动作的时候就不用了。

为了解决这个问题，我添加了操作员版本的`merge`，我将把它别名为`mergeOperator`，这样就很清楚了:

在这个例子中，`mergeOperator`的工作方式类似于`finalize`函数，我们不必复制我们的`loaded`动作。

# 听两个观察

我们的应用程序有一个错误通知，在 10 秒钟后自动删除，但也允许用户在计时器超时前手动隐藏错误。不知何故，我们必须找到一种方法来同时倾听两个可观察的声音，并让它们`race`来看看哪一个先发生。

我们使用和以前一样的概念，但是现在有了一个新的可观察的东西:`race`。这个可观测值发出两个可观测值之间的第一个值。

令人厌倦的事情，并不是比赛一开始就停止。虽然类似于`Promise.race()`，但它实际上一直在监听。当一个可观测物体发射时，它让该可观测物体的发射通过，并等待直到另一个也发射。一旦它发出信号，它就返回监听，看两者中哪一个会赢得下一场比赛。只要`race`仍处于活动状态，这种情况就会持续。

为了缓解无限等待的问题，我将`take(1)`放在了`HIDE_ERROR_NOTIFICATION`动作监听器上。`timer(10000)`完成后它会放出，这样我就不用上`take(1)`了。

最后一个非常奇怪的部分是`filter`。这是为了确保我们只调用一次`hideErrorNotification`动作。如果定时器先用完，将调用`hideErrorNotification`。既然在一个`race`里面，就不会发出什么东西。但是如果用户隐藏了通知，那么`hideErrorNotification`已经被调用过了，我们不想再调用第二次。这就是它被过滤掉的原因。

# 奔向美国

如果我们正在加载需要一个`accessToken`来分派动作的组件，那么很有可能这些组件会在状态中的访问令牌可用之前加载。在这个特殊的例子中，我们不想在没有`accessToken`的情况下进行 AJAX 调用。

下面是**易错**的例子:

这个特定的可观察对象可能使用`undefined`作为`accessToken`进行 AJAX 调用。你可能会想我们可以用种族来解决这个问题；我们可以，但没有必要。

为了确保您的状态中有一个`accessToken`，您需要**听** `**state$**`:

通过监听`state$`并检查来自`accessTokenSelector`的真值，我们确保只有在有`accessToken`时才进行这个 AJAX 调用。

这只起作用，因为`state$`是一种特殊的被称为`BehaviorSubject`的可观察对象，这意味着`state$`总是有一个值，而`action$`是无状态的，只能监听动作。这就是为什么`state$.value`存在，也是为什么听`state$`给你最新的值。

在另一种情况下，您可能不想等待`accessToken`可用。相反，如果没有可用的`accessToken`，你可以在`accessTokenSelector`之后写一些逻辑来显示错误。

# 结论

我希望你从阅读这篇文章中有所收获。我将它视为 Redux-Observable 文档的扩展，以及我自己在生产应用程序中使用 Redux-Observable 和 RxJS 的 9 个月经验的积累。

我所有使用 Redux-Observable 的应用程序都是用这些相同的构件构建的。我用 Redux-Observable 实现了很多解决方案，但这几个应该足够开始了！

# 更多阅读

如果您对更多与 Redux 和 RxJS 相关的主题感兴趣，您应该看看我的其他文章:

*   使用 Redux 的秘密:createNamespaceReducer
*   [在 React 组件中使用 Redux 还原剂](https://medium.com/@Sawtaytoes/using-redux-reducers-in-react-components-4e92985dd9cb)
*   [为什么你不需要 React-Redux 的“连接”](https://medium.com/@Sawtaytoes/why-you-shouldnt-need-connect-from-react-redux-498876de9e4e)
*   [RxJS 和可观察的 Flic 按钮](https://medium.com/flicblog/flic-buttons-and-the-observable-customization-using-rxjs-2214bc53d407)
*   函数式编程的表情爱好者指南:第一部分