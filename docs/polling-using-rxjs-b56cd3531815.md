# 使用 RxJS 进行轮询

> 原文：<https://itnext.io/polling-using-rxjs-b56cd3531815?source=collection_archive---------1----------------------->

随着 observables 在 JavaScript 中越来越受欢迎，我们希望使用它们来完成我们的日常任务，并评估它们是否真的值得大肆宣传。您可能会发现自己正在做的一项任务是轮询后端，以了解一个更长时间运行的任务是否已经完成。

![](img/d4006110cdaeac75a3b111aeebadbe2b.png)

由[克里斯·阿布尼](https://unsplash.com/@chrisabney?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的溪流照片

我们将浏览这样一个场景的例子，并使用 RxJS 实现一个解决方案。在路上，我们将学习 RxJS 的一些基本操作符，一些技巧以及如何避免一两个陷阱。最后，我将展示一个真实的例子，向您展示如何在一个特定的场景中实现我们所学的内容。

你应该带着对 Streams / [Observables](http://reactivex.io/documentation/observable.html) 的基本理解以及扎实的 JavaScript 基础来享受这篇文章。在这篇文章的其余部分，我将把 Stream 和 Observable 视为同一事物的可互换词。虽然我们将涵盖许多基本的东西，但它们主要是 RxJS 细节，而不是关于流的基础知识。如果你正在寻找一个一般性的介绍，考虑一下要点标题“[你一直错过的反应式编程介绍](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)”。

这篇文章的代码是用 RxJS 6.2.0 测试的。

# 方案

假设我们有一个后端，它公开了一个端点/tasks/[taskId],您可以查询它来了解特定任务的状态。它返回这样一个对象:

```
{
  // Whether the task is still running
  processing: boolean;
  // A unique ID for this task
  taskId: string;
}
```

一旦我们开始轮询，我们希望每秒两次获得该任务的当前状态，并在处理一次后停止轮询=== false。

# 程序化解决方案

首先，我们来看看这个问题的程序性解决方案。

这里，每当后端仍在处理时，我们简单地调用一个新的超时。

# 使用 RxJS

现在我们将使用 RxJS 实现相同的行为。

首先，我们需要一些东西在每次 *x* 的时候发出一个事件。RxJS 为此提供了两个功能:

*   间隔
*   计时器

*间隔*在给定时间后发出第一个事件，然后以相同的间隔连续发出，而*计时器*在给定时间后开始每隔 *x* 次发出事件。对于每秒两次更新，我们可以从使用 timer(0，500)开始。这将从蝙蝠的右边开始触发事件，之后每秒两次。

让我们首先通过在控制台上记录一些东西来看看实际情况。

```
import { timer } from 'rxjs'timer(0, 500)
  .subscribe(() => console.log('polling'))
```

现在，您应该看到您的控制台每秒钟打印两次“轮询”。

> 注意从正确的包中导入(rxjs 或 rxjs/operators)。遗憾的是，RxJS 文档可能跟不上您正在使用的版本。

接下来，我们想把这些“滴答”变成对我们后端的请求。我们将使用上面的相同提取，但这次**将承诺转化为可观察到的**。幸运的是 RxJS 为此提供了便捷的功能，即从开始的*。使用它，我们现在可以创建一个可观察对象(或流),表示每个时钟周期对后端的请求，并继续使用它。*

```
import { timer, from } from 'rxjs'
import { map } from 'rxjs/operators'timer(0, 500)
  .pipe(from(fetch(`/tasks/${taskId}`)).pipe(map(response => response.json())))
```

**。管道**是 RxJS 指定转换将在流上发生的方式。通过将操作符提取到它们自己的导入中，RxJS 实现了比重载的 Observable 实现更好的树摇动，更多上下文参见[这个解释](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)。

> Pipe 是 RxJS 对应用于事件流的转换的包装。

这样的结果将是一个**流的流**。每个发射值本身都是可观测的。为了管理混乱，我们可以通过 *concatMap* 将所有的流合并成一个包含嵌套值的流。

```
import { timer, from } from 'rxjs'
import { map, concatMap } from 'rxjs/operators'timer(0, 500)
  .pipe(concatMap(() => from(fetch(`/tasks/${taskId}`))
    .pipe(map(response => response.json())))
  )
```

# 完成轮询

最后，我们真正关心的是获得一个事件，告诉我们后端处理已经完成，我们的轮询已经完成。我们可以通过过滤后端不再处理的事件并只取其中的第一个事件来实现这一点。通过使用 *take(1)* 我们指定我们只关心一个(第一个)告诉我们处理已经完成的事件。一旦后端处理完任务，这将停止我们的轮询。

```
import { timer, from } from 'rxjs'
import { map, concatMap, filter, take } from 'rxjs/operators'timer(0, 500)
  .pipe(concatMap(() => from(fetch(`/tasks/${taskId}`))
    .pipe(map(response => response.json())))
  )
  .pipe(filter(backendData => backendData.processing === false))
  .pipe(take(1))
```

# 把所有的放在一起

现在是时候把它们放在一起，并使用新的基于 RxJS 的代码从上面替换我们的函数了。最后一点是在我们的流末尾使用 *subscribe* 来处理我们的流发出的单个事件。

```
import { timer, from } from 'rxjs'
import { map, concatMap, filter, take } from 'rxjs/operators'pollUntilTaskFinished(taskId) {
  timer(0, 500)
    .pipe(concatMap(() => from(fetch(`/tasks/${taskId}`))
      .pipe(map(response => response.json())))
    )
    .pipe(filter(backendData => backendData.processing === false))
    .pipe(take(1))
    .subscribe(() => pollingFinishedFor(taskId))
}
```

你可能不希望在完成后调用函数，而是使用可观察对象的输出来呈现你的 UI。通过使用合并，将两个流合并在一起，我们可以将我们的轮询映射到两个状态，并将输出直接用于我们的 UI。

为了实现这一点，我们将把上面的流和一个初始值合并在一起，我们使用中的*把它变成一个流。*

```
import { timer, from, merge, of } from 'rxjs'
import { map, concatMap, filter, take } from 'rxjs/operators'const loadingEmoji = merge(
  of(true),
  timer(0, 500)
    .pipe(concatMap(() => from(fetch(`/tasks/${taskId}`))
      .pipe(map(response => response.json())))
    )
    .pipe(filter(backendData => backendData.processing === false))
)
    .pipe(take(2))
    .pipe(map(processing => processing ? '⏳' : '✅'));
```

在我们使用 *map* 将来自后端的响应映射到处理属性上之后，我们可以将结果映射到表情符号上以显示给我们的用户。

# 真实世界的例子

理论总是好的，但现实世界通常会对一个写得很好、内容丰富的教程提出不同的挑战。让我向您介绍一个问题的解决方案，这个问题是我们在构建关于使用 RxJS 进行轮询的知识时遇到的。

情况:我们有一个 Angular 应用程序，我们使用 [NGXS](https://ngxs.gitbook.io/ngxs) 作为状态管理器。与 Redux 类似，它使用动作来表示改变状态的事件。

事实证明，NGXS 提供了一个调度所有动作的流，作为我们可以挂钩的可观察对象。这是我们最终的解决方案，轮询后端处理每个*文档**添加到状态中的状态，并在后端完成处理后更新状态。*

*一些注意事项:*

*   ***环境**是一个角度环境，为我们的应用提供配置。*
*   *后端是一个提供连接到我们后端的服务。它插入了一些必需的头等等。*
*   *这使用了 TypeScript，因此 polledDocument: Document 描述了一个名为“polledDocument”的变量，该变量遵循类型“Document”。*

*这里一件棘手的事情是，我们需要为每个添加到我们状态的文档创建一个新的“轮询流”。起初，我们尝试将逻辑放入一个单独的级别，但最终我们只能在每次页面加载时轮询一个文档，因为 take(1)会阻止所有未来轮询的流。*

# *包扎*

*今天，我们使用 RxJS 构建了我们的第一个轮询逻辑，并了解了这个伟大的库。我们还看了一个真实世界的例子，看看它能让我们的代码变得多么有表现力。*

*现在，走出去，应用你的新知识。*

*![](img/ad5eee9363a455b30096b87ee2a251c9.png)*

***其他重大资源***

*[https://blog.strongbrew.io/rxjs-polling/](https://blog.strongbrew.io/rxjs-polling/)*

*[https://www . site point . com/angular-rxjs-create-API-service-rest-back end/](https://www.sitepoint.com/angular-rxjs-create-api-service-rest-backend/)*

*[https://www.learnrxjs.io/recipes/http-polling.html](https://www.learnrxjs.io/recipes/http-polling.html)*

*原载于 2018 年 8 月 30 日 [makeitnew.io](https://makeitnew.io/polling-using-rxjs-8347d05e9104) 。*