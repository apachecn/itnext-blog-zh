# 为 Concourse 编写自定义资源—检测拉式请求关闭/合并事件

> 原文：<https://itnext.io/writing-a-custom-resource-for-concourse-detecting-pull-request-close-merge-events-e40468eb2a81?source=collection_archive---------3----------------------->

![](img/40d1bcc04ef7c0ab20697bf51dcc7e64.png)

> [点击这里在 LinkedIn](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fwriting-a-custom-resource-for-concourse-detecting-pull-request-close-merge-events-e40468eb2a81%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer) 上分享这篇文章

最近我一直在玩优秀的[广场 CI](https://concourse-ci.org/) 项目。这是一个非常酷的项目，作为一个开源的 CI/CD 解决方案已经获得了很多关注。请看[这一页](https://concourse-ci.org/concourse-vs.html)了解为什么 Concourse 如此出色，以及它与其他 CI/CD 解决方案相比如何。

在 Concourse 内部，所有的交互都是通过[资源和工作](https://concourse-ci.org/concepts.html)来完成的。它的模型是*功能性的*在某种意义上，流水线是由无状态的作业组成的，这些作业具有由资源建模的明确定义的输入和输出。一切都在它自己的容器中运行，这防止了任务间自动化环境的任何污染。为了让作业中的任何步骤共享任何内容，必须明确定义输入和输出。这有时会有点乏味，但它让管道内部发生的一切都非常清晰。所有这些特性使它非常适合用作 CI/CD 解决方案，因为它防止了许多导致构建中断的更神秘和令人沮丧的问题。

# 主旨和动机

使用任何种类的自动化系统实现 CI/CD 都需要系统与 Github 等源代码控制解决方案中发生的事件进行交互。特别是，触发 CI/CD 系统的一个更常见的事件是拉请求。Pivotal 的社区和 Concourse 的维护人员一直都很好，为大多数情况提供资源。已经有一个优秀的`[github-pullrequest-resource](https://github.com/jtarchie/github-pullrequest-resource)`可以用来处理 Github pull 请求，但是有一种情况[没有真正处理，那就是检测已经被合并或关闭的 pull 请求的能力](https://github.com/jtarchie/github-pullrequest-resource/issues/128)。

当以这样一种方式对 CI/CD 建模时，这可能是有用的，即打开的 PR 为应用源代码创建一个部署环境，而关闭的或合并的 PR 启动该环境的清理以回收资源。这种类型的流在云平台中特别有用，在云平台中，成本是按使用计费的，因此平台只在有开放的拉取请求时使用资源。这是我在当前使用 Concourse 的项目中真正需要的一个用例，所以我决定借此机会深入一点，编写我自己的定制资源，并希望记录这个过程，以便对其他人有用。对于 tldr 请查看[项目回购。](https://github.com/shinmyung0/pullrequest-events-resource)

# 了解 Concourse 资源

在我们开始之前，我们应该先试着理解如何为 Concourse 实现一个定制资源。我不想在这里深入讨论这个规范，但是基本上一个 Concourse 资源只是一个实现三个脚本的容器:

*   `/opt/resource/check`:检查资源的新版本
*   `/opt/resource/in`:拉下资源的一个版本
*   `/opt/resource/out`:幂等推一个版本上去

所有的资源都应该实现这 3 个脚本，但是它们并不都必须做些什么。对于不符合资源语义的操作，脚本可以是一个 noop。就我们的资源而言，我们实际上只需要实现`check`和`in`脚本，因为我们并没有从关闭或合并的拉请求中更新任何东西，只是获取关于它们的信息并触发下游作业。

# 了解`check script`

现在我们已经了解了我们需要为这个资源实现哪些脚本，让我们更深入地了解一下规范，以了解`check`应该为这个资源做些什么。分解规格:

*   调用资源类型的`check`脚本来检测资源的新版本。
*   在 stdin 上给出了`source`配置和电流`version`。
*   `source`是一个任意的 JSON 对象，它指定资源的位置，包括任何凭证。这是从[管道配置](http://concourse-ci.org/configuring-resources.html)中逐字传递过来的。
*   `version`是一个带有`string`字段的 JSON 对象，用来惟一地标识资源的一个实例。
*   这将在第一个请求中省略，在这种情况下，资源应该返回当前版本(*而不是*自资源开始以来的每个版本)。
*   它必须按时间顺序将新版本的数组打印到 stdout，包括请求的版本(如果它仍然有效的话)。

上面的规范基本上是说，Concourse 运行时将使用类似下面的命令来运行脚本:

```
echo {...source config json...} | /opt/resource/check
```

对于第一次`check`调用，脚本的输入将只包括`source`配置，但是在随后的请求中，`check`脚本也将被传递*当前*，它告诉资源返回*下一个*有效`version`对象。

有了这个规范，我们可以开始设计我们的`source`配置和`version`对象应该是什么样子。

# 定义版本和源配置

因为我们想从 Github 返回关于关闭和合并的 pull 请求的信息，所以让我们试着理解哪种信息是可用的。因为 Github 提供了[一个优秀的 GraphQL API](https://developer.github.com/v4/) ,我们可以使用 [explorer](https://developer.github.com/v4/explorer/) 来做一些探索，看看我们可以从 pull 请求中获取什么样的数据。经过一些试验，我想到了下面的 GraphQL 查询:

因为我们的查询将是运行`check`脚本所需要的，所以我们几乎可以使用输入参数(带有一些用于凭证和 API 端点的附加字段)作为我们的`source`配置的字段:

```
source:
  graphql_api: [https://api.github.com/graphql](https://api.github.com/graphql)
  access_token: ((github-access-token))
  base_branch: master
  owner: ((github-owner))
  repo: ((repo-name))
  first: ((num-to-fetch))
  states:
    - closed
    - merged
```

该查询将返回类似如下的有效负载:

基于这些信息，我们可以尝试返回一个类似下面的`version`对象数组:

```
{
    "id": "MDExOlB1bGxSZXF1ZXN0MTcxMjQ5NjI1",
    "cursor": "Y3Vyc29yOnYyOpK5MjAxOC0wMi0yNVQxMjozNDo0NC0wODowMM4KNQ/Z",
    "number": "1",
    "url": "[https://github.com/shinmyung0/fixture-repo/pull/1](https://github.com/shinmyung0/fixture-repo/pull/1)",
    "baseBranch": "master",
    "headBranch": "test-merged-branch",
    "state": "MERGED",
    "timestamp": "2018-02-25T20:34:44Z"
}
```

需要强调的一些要点是像`cursor`这样的字段，它可以被传递到我们的 GraphQL 查询的`after`字段中，以便只返回特定光标之后的拉请求。因为“当前的”`version`对象被传递到`check`脚本中以获取“新的”`version`对象，所以包含这个对象会很有用。

# 实现检查脚本

现在我们对我们的`check`脚本应该做什么有了一个清晰的想法，我们可以开始实际实现它了。因为我们知道只要满足规范，Concourse 资源可以用任何语言编写，所以我们可以选择最适合我们工作的语言。因为我们使用的是 GraphQL，所以我决定使用 JS 来实现它。处理异步网络调用很容易，处理 JSON 也很容易，而且现在有很多针对 GraphQL 的客户端库。我非常喜欢在 JS 中使用的 GraphQL 库是 [Apollo](https://www.apollographql.com/docs/react/) 。考虑到所有这些选择，`check`脚本的伪代码应该是这样的:

```
#! /usr/bin/env node// the shebang allows this file to be directly executable async function check() { // read stdin
  // parse and validate input json
  // use configuration to run GraphQL query to fetch PRs
  // convert response payload to version objects
  // output to stdout}

check()
```

对于实际的实现，检查源代码[这里](https://github.com/shinmyung0/pullrequest-events-resource/blob/master/scripts/check)。

# 了解脚本中的

让我们深入了解一下`in`脚本的规格。

*   将`in`脚本作为`$1`传递到目标目录。该脚本必须获取资源并将其放在给定的目录中。
*   脚本在`stdin`上给出了配置的`source`和要获取的资源的精确`version`。
*   该脚本必须发出获取的版本，并且可以发出作为键值对列表的元数据。

基于这个规范，Concourse 运行时将基本上以如下方式调用`in`脚本:

```
echo {... some json ...} | /opt/resource/in outputdir
```

因为我们的拉请求资源只是获取关于拉请求的*信息*,所以除了将`version`对象输出为文件之外，我们不需要获取任何额外的东西。因此，我们可以说，上面执行`in`脚本的结果将产生一些包含传递给`stdin`的`version`对象的`outputdir/pull_request`文件。它也会发出电流`version`到`stdout`。

# 在脚本中实现

基于我们对`in`脚本应该做什么的理解，`in`脚本的伪代码应该如下所示:

```
#! /usr/bin/env nodeasync function doIn() { // read stdin, parse, and validate
  // extract given .version key
  // output version object to $1/pull_request file
  // emit version to stdout}doIn()
```

下游作业可以读取`$1/pull_request`文件，并提取关于最近关闭或合并的拉请求的信息。

# 写作测试

由于我们使用的是 Node JS，所以我们可以使用 [Jest](https://facebook.github.io/jest/) 轻松编写一些单元和集成测试。[单元测试](https://github.com/shinmyung0/pullrequest-events-resource/blob/master/scripts/common.test.js)很容易编写，但是为了能够运行集成测试，我们需要[设置一个 fixture repo](https://github.com/shinmyung0/fixture-repo) 和一些关闭或合并的 pull 请求来测试实际的 API 调用。

因为调用这个 repo 需要一个 Github 访问令牌，所以我们可以将它作为一个[环境变量传递给集成测试套件](https://github.com/shinmyung0/pullrequest-events-resource/blob/master/scripts/integration.test.js#L10)。要做的一件好事是确保验证在测试中已经设置了访问令牌。查看测试代码，详细了解这是如何设置的。

# CI/CD 和发布到 Docker Hub

我们可以使用 Travis CI 轻松地为这个项目设置一些 CI/CD。基本上，CI/CD 工作需要做的就是运行所有测试，然后如果成功，提交被标记为发布，构建 Docker 映像，然后将其发布到公共 Docker Hub。非常简单，所以我不会在这里进行过多的描述。但是如果你好奇，请查看一下 [Travis CI 配置](https://github.com/shinmyung0/pullrequest-events-resource/blob/master/.travis.yml)以及[构建脚本](https://github.com/shinmyung0/pullrequest-events-resource/blob/master/build.sh)。

# 结论

这是本月要发布的一个有趣的项目。这给了我一个很好的机会深入研究 Concourse，这是我最近在工作中经常使用的东西。我对自己定期开源的能力越来越有信心，这也是我今年致力于做的事情。总的来说，Concourse 是一个非常棒的项目，我强烈推荐给任何正在寻找一个非常好的 CI/CD 解决方案的团队。