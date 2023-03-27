# 在现代 Webpack 应用程序中实现 NodeJS 环境变量

> 原文：<https://itnext.io/implement-nodejs-environment-variables-in-a-modern-webpack-app-df20c27fe5f0?source=collection_archive---------1----------------------->

![](img/5e122dadfc783932f47e7006b741a4c8.png)

斯蒂芬·昆泽在 [Unsplash](https://unsplash.com/search/photos/greece?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

当我们开始构建一个现代的 web 应用程序时，我们通常并不关心伸缩性。这包括所有不同的环境，我们将在未来安装这个应用程序。我们想把事情做完，然后开始玩我们的代码库。

因此，我们可能从最基本的开始，即区分我们的应用程序在三个主要环境中的设置:

1.  开发:我们开发新功能的地方
2.  生产:当我们构建一个包，并把它放在一个生产服务器上，这样我们的客户就可以玩我们的酷应用程序了
3.  测试:我们在本地或者在持续集成系统中运行单元测试

我知道你们中的一些人可能会想，现在开始制作是否为时过早。老实说，不是，因为你可能需要从一开始就设置一个临时服务器，这样其他团队成员就可以跟踪你的进度，并定期检查这个全新项目的真实情况。这就是应用了大量优化的类似产品的捆绑包需要使用的地方，对吗？

因此，显然我们将使用 Webpack 来组织我们的代码库，并建立这 3 个主要的环境设置。我的典型 webpack 设置为每个环境提供了一个专用的配置文件，使用一个不言自明的名称，如 webpack.development.config.js 等。

为了避免重复我自己，我喜欢先创建一个 webpack.base.config.js 文件，以便在那里放置一些通用设置。然后我通过使用 [webpack-merge](https://www.npmjs.com/package/webpack-merge) 包来扩展它。所有这些配置文件都放在名为 webpack 的 app 文件夹旁边的一个单独的文件夹中。多独特对吗？

好了，让我们看看 ReactJS 应用程序的简单 webpack.base.config.js 到底是什么样子的:

…我是这样在我的 webpack.development.config.js 中扩展它的:

既然我们已经准备好了一些基本的东西，让我们创建 npm 脚本并在那里实现我们的 webpack 配置文件:

以上都是很基本的东西。我们有一个脚本在开发环境中引导我们的 webpack-dev-server，另一个脚本为生产环境构建我们的包，最后一个脚本用 Jest 运行一些单元测试。设置 Jest 超出了本文的范围，但这是引导 Jest 并运行单元测试套件的典型脚本。

好了，我们已经取得了很大的进步，但是我们实际上还没有传递任何关于每个脚本运行环境的变量。计划很简单。为了保持整洁，我们将在编译时通过`[webpack.DefinePlugin](https://webpack.js.org/plugins/define-plugin/)`在 webpack.base.config.js 文件中定义环境变量，其余的配置文件将继承它们，因为它们扩展了这个基本配置文件。但是我们如何为每个 npm 脚本传递不同的变量呢？

最简单的方法是使用一个名为 [cross-env](https://www.npmjs.com/package/cross-env) 的超级漂亮的包(它使我们避免了由 Windows 操作系统引起的潜在问题)，并在运行每个脚本之前声明我们需要的环境变量。

那么我们需要指定的第一个也是最重要的变量是什么呢？你完全正确，是时候在每个脚本运行之前声明`NODE_ENV`了:

我们做到了，但是 webpack 知道了吗？嗯，不。记住我们上面说的。我们需要修复我们的 webpack.base.config.js 并利用 DefinePlugin:

现在，webpack 确切地知道我们在每种情况下所处的环境，并使用`NODE_ENV`,尤其是在生产构建中，来优化我们的捆绑包。

NodeJS 中的 [process.env](https://nodejs.org/dist/latest-v8.x/docs/api/process.html#process_process_env) 是一个对象，包含关于用户环境的非常有用的信息。因此，正如您看到的，cross-env 为每个案例传递了正确的`NODE_ENV`值，webpack 从中读取该值。如果你想知道我们为什么需要`JSON.stringify()`的原因是 webpack 在那里直接进行文本替换，所以我们需要传递一个带引号的文本值，如果你像我一样感到懒惰，那么`JSON.stringify()`可以很好地完成这项工作:-)

好了，现在我们已经通过 cross-env 将`NODE_ENV`传递给了我们的脚本，我们可以让它们更短一些。仔细想想，我们并不真的需要配置文件的全名。我们可以在 webpack 文件夹中创建一个 index.js 文件，这样做:

…并相应地更改我们的 npm 脚本:

注意，我们直接从`./webpack`文件夹中请求一个配置文件，而不是每次都声明完整路径。

太好了，我们设法让这一切变得非常动态，现在我们可以专注于真正的开发了。

几个月过去了，我们开始有一些基于客户环境等的额外要求。这个应用程序将被安装在一对夫妇的客户内部网，这些都有一些专门的需求。每个安装将使用一个 API，每个客户端使用不同的 IP。嗯，我们需要在编译时在我们的 webpack 包中传递这个 IP。但是我们如何做到这一点呢？嗯，是时候在我们的代码库中使用更多的环境变量了，所以`.env`文件开始讨论了。

`.env`是一个文件，我们在其中声明了一些特定于环境的键值对设置，比如 so `KEY=VALUE`。git 系统绝对应该忽略这个文件，以避免不必要的覆盖。

让我们看一个用 API 解决这个问题的例子:

在每个环境中使用一个特定的`API_URL`来安装我们的应用程序是非常常见的，而`.env`是放置这种设置的合适地方。

既然我们创建了我们的`.env`文件，现在是时候找到一种方法来读取它的内容并将其传递给 webpack。为了实现这一点，我们将使用一个名为 [env-cmd](https://www.npmjs.com/package/env-cmd) 的非常酷的包，与我们使用 cross-env 的方式类似。让我们相应地更新我们的 package.json，看看情况如何:

…然后更新 webpack.base.config.js:

酷。现在我们可以在典型的 [axios](https://github.com/axios/axios) 设置中使用这个特定于环境的`API_URL`变量，如下所示:

就是这样。只要我们每次相应地更新 webpack.base.config.js，我们就可以添加任意多的特定于环境的变量。

对于测试环境，我们通过使用 webpack.development.config.js 使事情变得非常简单。如果您真的需要，可以随意创建一个专用的配置文件来满足您的需要。如你所见，这种模式是可扩展的。干杯！！