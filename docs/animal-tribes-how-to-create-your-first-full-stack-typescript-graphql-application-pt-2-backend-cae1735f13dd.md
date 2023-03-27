# 动物部落:如何创建你的第一个全栈类型脚本 GraphQL 应用程序？—第二部分:后端

> 原文：<https://itnext.io/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-pt-2-backend-cae1735f13dd?source=collection_archive---------2----------------------->

如何使用 Typescript、Node、React 和 GraphQL 构建第一个全栈应用的完整教程

![](img/a10e774e28b9a234df5210fa68acd219.png)

# 1.介绍

这是**动物部落的第二部分:如何创建你的第一个全栈式 Typescript GraphQL 应用？**教程。如果您没有看到第 1 部分，请点击下面的链接。

[](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-76786e5520ed) [## 动物部落:如何创建你的第一个全栈类型脚本 GraphQL 应用程序？

### 如何使用 Typescript、Node、React 和 GraphQL 构建第一个全栈应用的完整教程

medium.com](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-76786e5520ed) 

在这一部分中，我们将使用 Node JS、Typescript、Express、GraphQL 和 MongoDB 来开发我们的服务器。

一件重要的事情是将我们的代码添加到 [GitHub](https://github.com/) 中。关于这一点我还会多说一些。如果你没有 Git 和/或 GitHub 的经验，可以看看我下面的文章。

[](https://medium.com/swlh/what-do-i-need-to-know-about-git-5017bde0b572) [## 关于 Git，我需要了解什么？

### 初学者教程

medium.com](https://medium.com/swlh/what-do-i-need-to-know-about-git-5017bde0b572) 

# 2.设置 MongoDB

**第一步:**首先，我们需要一个关于 [MongoDB Clouds](https://www.mongodb.com/) 的账户。点击**免费试用**。

![](img/a1191a2f6f1ef7b831c55bfb0f841f1a.png)

**第 2 步:**在字段中填入您的数据，然后点击**免费开始**。

![](img/e91a8f8ce75c6318f325a2abfd038640.png)

**步骤 3:** 选择选项**共享集群**，点击**创建集群**。

![](img/e52e6c081500da7537361dac3e7e38fe.png)

**第四步:**在**云提供商**上，选择 **AWS** 选项，并为该地区选择离你的国家最近的选项，在我的例子中，是**法兰克福(欧盟-中央-1)** 。然后，点击**创建一个集群**。

![](img/bf43536bf359c77ad23eb4955097d88e.png)

第五步:你应该会看到一个类似于这个的屏幕。

![](img/454b8f4c574f8419f95467d9e1b69523.png)

**第 6 步:** MongoDB 将完成新数据库的部署。等待部署完成。然后，点击**连接**。

![](img/4904bd192849db96237049b67395cdf4.png)

**第 7 步:**点击**添加不同的 IP 地址**并添加 **0.0.0.0/0** ，点击**添加 IP 地址**。然后为您的数据库创建一个用户，在我的例子中，我正在创建一个 **admin** 用户，使用一些**密码，点击**创建 MongoDB 用户。并点击**选择一种连接方式**。

![](img/a062ae40d5bb5b47c49791bfda79cc8a.png)![](img/adf664a0009dff593229eb7e1010ef94.png)

**第 8 步:**选择选项**连接您的应用**。

![](img/18f4a3a6f367133a56e4f7c92e500eea.png)

**第九步:**然后，复制连接字符串，保存到别的地方，我们很快就会用到。然后，关闭它。

![](img/537f5015fd4df04424ee439c40860ef3.png)

连接字符串应该是这样的。

```
mongodb+srv://admin:**<password>**[@cluster0](http://twitter.com/cluster0)-9ksfk.mongodb.net/test?retryWrites=true&w=majority
```

不要忘记将 **<密码>** 替换为您为用户设置的密码。⚠️ **注意** ⚠️:这不是 MongoDB 密码，而是您在**步骤 7** 中为数据库用户设置的密码。例如，如果您将用户定义为 **admin** ，将密码定义为 **admin** ，那么应该是:

```
mongodb+srv://admin:**admin**[@cluster0](http://twitter.com/cluster0)-9ksfk.mongodb.net/test?retryWrites=true&w=majority
```

# 3.设置服务器

为了设置服务器，我们首先需要安装的是[节点 JS](https://nodejs.org/en/) 。为此，让我们安装 [nvm](https://github.com/nvm-sh/nvm) 并按照[步骤安装](https://github.com/nvm-sh/nvm#installing-and-updating)。

然后，你可以做

```
nvm install node
```

使用这个命令，您将安装**节点**、 **npm** 和 **npx** 。

让我们创建一个名为**动物部落服务器**的文件夹。为了改进我们的编码，我们可以设置格式化程序、链接程序和代码修复程序。幸运的是，我们有 [Google Typescript Style](https://www.npmjs.com/package/gts) 为我们安装所有这些。

```
$ npx gts init
```

这样做，我们应该有一个类似下图的文件夹结构。

![](img/53c90640942488554f39cc968580c353.png)

为了在我们编码的时候提高效率，我们将在保存时设置 lint。我用 [VS 代码](https://code.visualstudio.com/)作为我的编辑器。所以我们可以创建一个名为**的文件夹。vscode** 并添加一个名为 **settings.json** 的文件。另一种方法就是只需在 Mac 上输入 cmd+shitf+p(Windows 上 ctrl+shift+p)，输入设置，选择第一个选项。

![](img/9390a765674462882d2faf23b971c5e2.png)

小心，文件夹**。vscode** 应该在我们的**动物部落服务器**文件夹之外。

![](img/d278d202d35a8027503458baca7f7a4a.png)

对于该文件，您将激活 **eslint** 并在保存时格式化。

我喜欢用[纱](https://yarnpkg.com/)而不是 [npm](https://www.npmjs.com/) ，但如果你更喜欢用 npm，可以跳过这一步。删除 **package-lock.json** 。

```
$ rm package-lock.json
```

现在，让我们添加依赖项。要在 TypeScript 中添加依赖项，我们需要在依赖项名称前面加上“@types/”,例如，如果我们想安装 **express** ，我们需要安装 **@types/express** 。

我将在这里添加我们将用于该项目的所有依赖项。然后…

```
$ yarn add \
[@types/bcrypt](http://twitter.com/types/bcrypt) \
[@types/cors](http://twitter.com/types/cors) \
[@types/dotenv](http://twitter.com/types/dotenv) \
[@types/express-graphql](http://twitter.com/types/express-graphql) \
[@types/jsonwebtoken](http://twitter.com/types/jsonwebtoken) \
[@types/lodash](http://twitter.com/types/lodash) \
[@types/mongoose](http://twitter.com/types/mongoose) \
[@types/yup](http://twitter.com/types/yup) \
bcrypt \
concurrently \
cors \
dotenv \
express \
express-graphql \
graphql \
graphql-playground-middleware-express \
jsonwebtoken \
lodash \
module-alias \
mongoose \
nodemon \
ts-node \
tsc \
tslint \
tslint-config-airbnb \
tsutils \
types \
typescript \
yup
```

有些依赖对于我们的应用程序的功能来说不是必需的，但是对于开发来说是必需的，比如 tests 和 prettier。在这种情况下，我们需要安装那些依赖项，但只是针对开发模式，然后…

```
yarn add **-D** \
[@types/chai](http://twitter.com/types/chai) \
[@types/express](http://twitter.com/types/express) \
[@types/mocha](http://twitter.com/types/mocha) \
[@types/node](http://twitter.com/types/node) \
[@types/supertest](http://twitter.com/types/supertest) \
chai \
gts \
mocha \
prettier \
supertest \
typescript
```

package.json 应该是这样的:

也许，图书馆的版本会有所不同，但这取决于你何时阅读本教程。

现在我们需要对我们的**脚本**进行一些修改，以添加更多的功能。让我们将下面的代码添加到 package.json 中。

```
"start": "concurrently --kill-others 'npm run tsc:watch' 'npm run dev'",
"dev": "nodemon build/src/index.js",
"lint": "npx tslint --init",
"tsc": "./node_modules/.bin/tsc",
"tsc:watch": "./node_modules/.bin/tsc -w",
"compile": "tsc -p .",
"test": "NODE_ENV='test' mocha build/src/tests/*.test.js --exit",
"script": "ts-node"
```

最终的文件应该是这样的。

> 嘿，山姆，你添加了太多的库、依赖项、脚本…我都糊涂了...*😖*

不要担心，我会一步一步地解释，当我使用每一个的时候。如果我现在把所有的内容都放进去，你的脑子会爆炸的。那就深呼吸。

让我们的代码遵循缩进、标记等模式。让我们使用[更漂亮的](https://prettier.io/)，然后让我们更新 **.prettierrc.js** 文件。

如您所见，这种配置将帮助我们更好地编码，它将自动为我们修复代码。例如，如果我们创建一个带引号的字符串，它将很容易转换成单引号。它会识别我们，然后继续下去。你可以添加更多的选项，并设置你喜欢的，见[这里](https://prettier.io/docs/en/options.html)漂亮者提供的选项。你可以在这里找到更多细节。

还有一件事，让我们更新 tsconfig.json 文件。

有了这个配置，我们将能够像这样导入文件。

```
**import** {abcd} **from** '@local/efgh'
```

这很重要，因为我们可以避免许多”../"在我们的代码中。

```
**import** {abcd} **from** '../../../efgh'
```

你可以在这篇文章中了解更多我所谈论的内容。

因此，有了这些配置，我们就可以着手编写代码了。

更新 **index.ts** 文件以创建一个简单的服务器。

让我们开始应用程序。

```
$ yarn start
```

您应该会看到类似这样的内容。

![](img/20f0b08daeebe00371ff2dd7ee9d5ed7.png)

然而，我们不能做任何操作，因为我们的服务没有任何 API。然后，让我们一步步地创建我们的服务器。

## 3.1.设置环境变量。

在安装过程中，我们安装的一个库是 [dotenv](https://www.npmjs.com/package/dotenv) 。使用 **dotenv，**我们可以设置允许我们隐藏密钥的环境变量，因为这些变量存储在**中。env** 文件。在这个文件中，我们将创建负责存储 utils 变量的变量，主要是数据库链接(见第 2 节)和我们的 JWT 秘密。

> 山姆，我知道你谈到的数据库链接，但什么是这样的 JWT？

啊哈，JWT 是 **JSON Web 令牌**，它是我们用来控制应用程序认证的工具。如果你想了解更多，你可以看看我的一个帖子，在那里我可以更深入地了解 JWT。

[](https://medium.com/swlh/authentication-how-to-create-a-nodejs-application-using-jwt-cee8bc5a89fe) [## 身份验证:如何使用 JWT 创建 NodeJS 应用程序

### 用 JSON Web 令牌保护 API

medium.com](https://medium.com/swlh/authentication-how-to-create-a-nodejs-application-using-jwt-cee8bc5a89fe) 

所以，让我们创造出**。env** 文件。

```
$ touch .env
```

如果你是 Windows 用户，只需输入

```
type nul > .env
```

⚠️注意:我不会把重点放在 Windows 命令上，但正如你所看到的，创建它们很容易。当我说**触摸**时，只需将其替换为**类型的 nul >** ，或者使用您的 IDE 的命令。

然后，让我们在创建的文件中添加下面的代码。

我将环境( **NODE_ENV** )设置为*开发*，将端口( **SERVER_PORT** )设置为 5000。在开发模式下，URL ( **SERVER_URL** )是 [HTTP://localhost](http://HTTP://localhost) ，我的秘密( **JWT_SECRET** )是动物秘密(可以更好，有创意，我没有😅)而我的服务器是 **SERVER_DB_DEV** 和 **SERVER_DB_TEST** 。

请注意，我在开发链接中更改了数据库。

```
mongodb+srv://admin:admin@cluster0-9ksfk.mongodb.net/**development**?retryWrites=true&w=majority
```

这样，我们就有了一个用于开发的数据库和另一个用于运行测试的数据库。

不要推动**非常重要。env** 文件到您的 git。然后，将该文件添加到**中。gitignore** 。让我们创建并添加到我们的文件。

```
$ touch .gitignore
```

加上这个。

```
*# dotenv environment variables file* .env
```

我已经有一个. gitgnore 文件，避免推送不必要的东西。你可以用它。

## 3.2.设置配置文件夹

在这一步中，我们将为每个环境创建配置文件。正如我在前一章提到的…

[](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-76786e5520ed) [## 动物部落:如何创建你的第一个全栈类型脚本 GraphQL 应用程序？

### 如何使用 Typescript、Node、React 和 GraphQL 构建第一个全栈应用的完整教程

medium.com](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-76786e5520ed) 

…我们将创建三个环境:开发、测试和生产。因此，我们需要创建三个文件，并再创建一个文件来控制所有这些文件，即 **index.ts** 文件。

```
$ cd src
$ mkdir config
$ touch config/index.ts config/development.ts config/test.ts config/production.ts
```

在 **development.ts** 文件中，我们来设置**上的变量。env** ，然后:

在第一行中，我们正在加载**中的变量。env** 文件，创建并导出配置变量。

同样的事情也适用于**测试。**

在这个文件中有三个不同之处，在 L4(第四行)上，我直接添加了字符串“HTTP://localhost ”,而不是从. env 中获取它。我只是想说明您可以直接设置一个变量，因为测试将在我们的机器上进行，所以为它创建另一个变量是没有意义的。L5 也差不多，因为我们可以随心所欲的设置端口。在 L6，我们正在建立从**获得的正确数据库。env** 。

**production.ts** 文件有区别，它没有**要求(' dotenv') *。配置* ()** 行。原因是，我们不是从 dotenv 获得这些变量，而是从云平台的环境变量中获得。请相信我🙂，我将在第 4 部分向您展示如何做到这一点。

[](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-e7891ec4963a) [## 动物部落:如何创建你的第一个全栈类型脚本 GraphQL 应用程序？

### 如何使用 Typescript、Node、React 和 GraphQL 构建第一个全栈应用的完整教程

medium.com](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-e7891ec4963a) 

现在，只要这样做:

最后，让我们创建索引文件。

在 L10 中，我们根据环境(NODE_ENV)编写了配置变量。然后，我们完成了配置文件夹。

## 3.3.设置数据库

在这一步中，我们将创建连接和断开应用程序与数据库连接的函数。

```
$ cd src
$ mkdir db
$ touch db/index.ts
```

对于 MongoDB 连接，我们将使用 mongoose。

在[猫鼬官网](https://mongoosejs.com/docs/connections.html)可以看到所有的连接选项。

## 3.4.设置模型

在这一节中，我们将创建模型来使我们的应用程序工作。正如我们在第 1 部分中提到的，我们需要战士和技能。如果您没有看到第 1 部分，请查看下面的链接。

[](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-76786e5520ed) [## 动物部落:如何创建你的第一个全栈类型脚本 GraphQL 应用程序？

### 如何使用 Typescript、Node、React 和 GraphQL 构建第一个全栈应用的完整教程

medium.com](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-76786e5520ed) 

让我们创建我们的文件夹和文件。

```
$ cd src
$ mkdir models
$ touch models/warrior-model.ts models/skill-model.ts models/tribe-model.ts
```

> 等等等等萨姆。我理解了 warrior-model.ts 和 skill-model，但是你为什么要创建一个叫做 tribe-model.ts 的文件呢？*🤔*

谢谢提醒，我创建了两个文件: **warrior.ts** 和 **skill.ts** 。但是我想补充一点，为部落创建一个枚举，我将在模型的文件夹中创建一个文件，因为它可以更有组织性。🙂

所以我们开始吧。

正如我在第 1 部分中所说的，战士必须有名字，战士名字，加入游戏的密码，以及所属的部落。

为了安全起见，我们不能将密码直接存储在数据库中，我们需要对它进行加密。为此，我们将使用 [bcrypt](https://www.npmjs.com/package/bcrypt) 。因此，在保存战士之前，我们要加密密码。

正如您在 L4 中意识到的，我们有一个部落枚举的导入，但是我们仍然没有文件中的内容。所以，让我们来创造它。

这些是我在申请中选择的部落。

最后，我们有技能。

## 3.5.设置中间件

要设置我们的中间件，我们需要 JSON Web 令牌(JWT)，我们要做的唯一一件事就是验证令牌。

让我们创建我们的文件夹和文件。

```
$ cd src
$ mkdir middlewares
$ touch middlewares/validate-token.ts
```

在 ValidateResponse 类型中，我们有 id、warrior 名称、iat(发布于)和 exp(过期声明)，这两个最新的名称可以在 [jsonwebtoken 库](https://www.npmjs.com/package/jsonwebtoken)中找到。

## 3.6.设置规则

在数据库中添加任何东西之前，都需要以某种方式进行验证，然后我们需要建立一些规则来避免添加错误的值。

为此，我们将使用 [Yup 库](https://www.npmjs.com/package/yup)。有了 yup，我们将能够验证一个必需的字段，字符数，如果是一个数字，如果战士的名字已经存在，等等。

让我们创建我们的文件夹和文件。

```
$ cd src
$ mkdir rules
$ touch rules/warrior-rules.ts rules/skill-rules.ts
```

对于 warrior，我们必须验证三件事:注册、登录和部落。

现在我们需要考虑项目架构。如我之前所说，注册时，我们需要:全名、战士名和密码。那么，规则应该是:

*   **全名**:必选字符串
*   **战士名字**:必输字符串，至少 3 个字符，并且是唯一的名字。
*   **密码**:为安全起见，由字母、数字和/或符号:@、！、#和%。

对于登录，我们有一些非常相似的东西:

*   **战士名字**:必填字符串，至少 3 个字符，应该存在于数据库中。
*   **密码**:必填字符串，至少 6 个字符，与用户密码匹配。

对于部落来说，这很简单，只是一个必需的大写字符串。

考虑到这一点，我们可以编写 **rules/warrior-rules.ts** 文件。

对于技能，我们知道一个战士拥有它们，所以我们唯一需要验证的是战士是否有一个战士名字。

## 3.7.设置 GraphQL

最后，我们将跳过 [GraphQL](https://graphql.org/) 的概念。我将介绍其中的一些。

*   **类型** : GraphQL 有自己的模式定义语言(SDL)，在这里我们可以创建一个 API 的模式。
*   **查询**:我们可以通过查询获取数据。
*   **突变**:我们可以用突变写数据。
*   **解析器**:实施的关键部件

这是你现在唯一需要知道的事情。我知道我说得很简洁，但如果你想更深入，你可以看看[如何掌握](https://www.howtographql.com/)。在我看来，这是关于 GraphQL 最好的教程之一。

了解了这一点，我们就能够创建我们的 GraphQL 结构。我们已经安装了 [GraphQL 库](https://www.npmjs.com/package/graphql)，接下来让我们创建文件夹和文件。

```
$ cd src
$ mkdir graphql
$ mkdir graphql/resolver
$ touch graphql/mutation.ts graphql/query.ts graphql/schema.ts graphql/type.ts
$ touch graphql/resolver/warrior-resolver.ts graphql/resolver/skill-resolver.ts
```

酷，来自 [GraphQL 库](https://www.npmjs.com/package/graphql)，我必须解释一些概念。

对于打字，我们将使用 **GraphQLString** ， **GraphQLInt** ， **GraphQLID** 。如您所见， **GraphQLString** 类型为**字符串，GraphQLInt** 类型为**整数，**我们有一个特殊类型 **GraphQLID** ，类型为 **ID。**

**GraphQLObjectType** 创建自己的类型，并且 **GraphQLNonNull** 禁止字段为空**。**

**然后，我需要为战士和技能创建一个类型。但是我将添加另一个注册/登录专用的类型。如你所知，我们要使用 JWT，那么我需要一个**令牌**，这个类型将由令牌和战士本身组成。**

**然后，我们来编写 **graphql/type.ts** 文件。**

**太好了！！！我们有 GraphQL 类型。所以，让我们来处理关于我们游戏的问题。我们需要一个查询来获取**战士**的信息，此外我们还需要另一个查询来获取**对手**的信息。记住战士不能和同一个部落的人战斗，他需要和另一个部落的人战斗。然后我们需要得到所有的**对手**(一个对手名单)。最后是一个战士/对手的**技能**。对于这个项目，出于教学的原因，我将添加更多我们**不会**在前端使用的查询。**

**很好，让我们再来考虑一下建筑。我们需要以三种方式操作战士:注册、登录和更新部落。这意味着，我们需要创建一个向数据库添加新战士的函数(注册)，我们需要在传递战士名称和密码(登录)时验证用户是否存在，并更新用户选择的部落。我们知道，**突变**负责写数据，然后我们需要创建它。然而，为了使我们的代码更简单，我将创建**解析器**来说明这一点，然后您就可以调用变异解析器了。**

**让我们记下**graph QL/resolvers/warrior-resolver . ts**。**

**不错！！！对于技能，我们只需要知道，当我们创建一个新的战士时，我们需要添加他/她的技能，并且每次战士有战斗时，我们都更新技能值。当一个战士被创造出来的时候，我们会在 50 到 100 之间随机添加数值。**

**然后，让我们编写**graph QL/resolvers/skill-resolver . ts**。**

**有了分解器，我们就能制造突变。我们可以从解析器中导入函数。我们来写 **graphql/mutation.ts****

**太棒了，通过类型、查询和变异，我们可以创建模式。让我们编写 **graphql/schema.ts****

**很简单，对吧？🙂**

## **3.8.完成设置**

**现在我们已经有了格式良好的所有结构，我们可以更改 **index.ts** 来添加 [GraphiQL](https://github.com/graphql/graphiql) 。是的，中间有个“我”。**

**有了这个，您就能够测试应用程序了。**

**在终端中，您应该键入。**

```
$ yarn start
```

**您应该会看到类似这样的内容:**

**![](img/e2347d35072e41dbd4ffe13668d83fa9.png)**

**如果一切运行正常，我们可以在任何浏览器中测试我们的 GraphQL 应用程序。在我的应用程序中，我将使用 Google Chrome。然后，打开浏览器，输入**

```
[http://localhost:[PORT]](http://localhost:[PORT])/graphql
```

**就我而言:**

```
[http://localhost:5000/graphql](http://localhost:5000/graphql)
```

**您应该会看到图形界面。**

**![](img/00adb6f3621bde787dc9ed68833e55b3.png)**

**我们可以做所有的查询和突变。但是，我真的很喜欢一个叫做 [GraphQL playground](https://github.com/prisma-labs/graphql-playground) 的工具。然后，让我们将它添加到我们的应用程序中。这只是关于偏好。**

**区别在 L13–14 和 L28 线。现在，在浏览器中键入:**

```
[http://localhost:[PORT]](http://localhost:[PORT])/playground
```

**就我而言:**

```
[http://localhost:5000/](http://localhost:5000/graphql)playground
```

**经过这一更改，您应该会看到这样的屏幕:**

**![](img/99fedb31d6ef9a1985c754c7156913e2.png)**

**右侧是我们创建的文档和模式，您可以看到已经为您的应用程序创建的所有内容。**

**![](img/8833b84eecfd8fca6bc300b2b28f0936.png)**

**(计划或理论的)纲要**

**![](img/d95de55aacd6daf3bfda9d824a5e4c70.png)**

**文件（documents 的简写）**

**然后，让我们测试端点。**

## **3.9.测试**

**在这一部分，我将向您展示我们的应用程序的测试。我将从两个方面来做这件事。通过手动测试和单元测试。在第一种情况下，我将使用 Playground 手动展示这一点，第二种方式是使用 chai 和 mocha。**

****3.9.1。手动测试****

*****3.9.1.1。*测试报名(突变)****

**首先，让我们测试第一个突变来开始我们的应用程序，让我们测试一下**注册**。我们需要将全名、战士名和密码作为参数传递。然后，作为结果，我想确切地看到战士 ID、战士名字和全名，以比较事情是否顺利。**

**如果我们在操场上测试时点击播放按钮。我们应该看到:**

**![](img/95132a09cc16a971a4e0c7fc09097896.png)**

**但是，如果我们再次单击 play 按钮，它应该会抛出一个错误，通知用户已经存在。**

**![](img/73f5a9301c1e27f513e4f4cfc7061287.png)**

**很好，为了确保我们的应用程序正常工作，让我们看看我们在 MongoDB 上插入的数据。**

**这就是了。**

**![](img/c9f0114a473d56c1bcc87e3ea9e875bd.png)**

*****3.9.1.2。测试登录(突变)*****

**对于登录，我们只需要战士的名字和密码。在这种情况下，我们将在已创建的用户中使用相同的凭据。但是，如果你记得，这个突变返回战士和令牌，然后，从战士，我想看到确切的战士 ID，战士的名字，和战士所属的部落，检查他们是否正确。**

**![](img/3d8b1f695947b101138cc2228a13d961.png)**

**如我们所见，部落值为空。原因是我们没有为用户设置部落。但是我们有一种突变来做到这一点。让我们开始吧。**

*****3.9.1.3。测试 updateTribe(突变)*****

**关于这种变异的一个重要的事情是，因为它是受保护的，这意味着我们需要授权来使它工作，这种授权正是我们从以前的变异(登录)中获得的令牌。**

**如果您试图在没有授权的情况下调用突变，您应该会看到一个错误，通知您必须提供 JWT。**

**![](img/dae8303777a0fa1c0b304a769ab5aa45.png)**

> **嘿，山姆，我想我明白了。当然，它应该抛出一个错误。令牌与用户相关，如果你不提供令牌，你就不能更新战士。**

**没错。！！这就是我们不需要通过战士 ID 来更新它的原因。我们只需要授权令牌。为了提供授权，我们需要从 **HTTP 头**传递，你可以在操场底部找到。**

```
{
  "Authorization": <YOUR TOKEN>
}
```

**例如**

```
{
  "Authorization": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVlYzRmMzIwMjc3NjgxYTRmYjVhM2E5NCIsIndhcnJpb3JuYW1lIjoic2FtMSIsImlhdCI6MTU4OTk2NTYwNiwiZXhwIjoxNTkwMDUyMDA2fQ.a2O5k0gi1GvXWmaPb-4aTPLetL55MjBEdsYS8nRZanA"
}
```

**那么，我们开始吧。**

**![](img/3af55a96ec5ba3d557f4c332612439cf.png)**

*****3.9.1.4。测试 addSkill(突变)*****

**现在，让我们测试战士的技能增加。正如我们所知，我们将随机设置数据，然后我们只需要调用突变，但我会在返回时添加技能，以查看设置了哪些技能，以及战士名称。**

**![](img/05e45331d62a5ec9c711754a1ca8b431.png)**

****3.9.1.5*。测试更新技能(突变)*****

**如你所知，当战士战斗时，它会改变他/她的技能，那么让我们在智慧上加 10 分。你也看到了，现在的智慧是 60，那我们就改成 70 吧。如果我们不通知其他技能，它会保持不变。**

**⚠️注意:在接下来的调用中，您需要授权令牌，但是因为您知道这一点，并且我已经向您展示了这一点，我将不再在图片中显示授权，只是为了更好地可视化。**

**![](img/6bde625d409de87c0d981ae87472adad.png)**

*****3.9.1.6。*测试战士(查询)****

**现在是测试查询的时候了，我们已经添加了数据，我们可以检查是否一切都是正确的。**

**让我们查询一个战士，看看结果。我们想要获取 id、姓名、战士姓名和战士所属的部落。**

**![](img/3fb281fee1d3a9428ff419dabd5659fd.png)**

**一个有趣的事情是，如果我们想得到密码，反过来，它应该是加密的。**

**![](img/d787407187f455b30ab1d4d03a82e37b.png)**

***3.9.1.7**。***测试对手(查询)**

**这个查询在我们的应用程序中非常有用，因为在一个查询中我们可以获得所有关于技能和战士的信息。**

**在 MongoDB 上，我会得到一个对手的信息。对手是属于另一个部落的另一个战士。**

**我创建这个用户是为了测试。你已经知道如何做到这一点，你可以很容易地注册另一个用户。**

**![](img/8637aea6b79da7288692038832f9220e.png)**

**有了这个，我就可以查询对手了。**

**![](img/0c270cf9c020bca29c2c536af2f82ea5.png)**

*****3.9.1.8。*测试对手(查询)****

**最后，我们有最后一个查询。这个查询将得到一个对手列表，记住在这个列表中我们不能有和战士来自同一个部落的对手(这就是为什么我们称他们为对手，而不是盟友)。**

**![](img/0c8cbd0ceacdc764e0c2f41c6b388522.png)**

****3.9.2。单元和集成测试****

**相当酷！我们已经手工测试了我们的应用程序。但是我们要做的是让我们的应用程序变得更好。为此，我们将创建单元和集成测试。**

**如果你看一下[测试驱动开发(TDD)](https://en.wikipedia.org/wiki/Test-driven_development) ，你会发现在方法论中，测试应该在开发本身之前编写。但是，出于说教的原因，我把它们留在了最后。**

**我准备用[柴](https://www.chaijs.com/)和[摩卡](https://mochajs.org/)进行测试。如果你想更深入地了解这一点，可以看看我的帖子，我写下了如何使用 chai 和 mocha 进行测试。**

**[](/how-to-make-tests-using-chai-and-mocha-e9db7d8d48bc) [## 如何用柴和摩卡做测试？

### NodeJS 应用程序的 TDD

itnext.io](/how-to-make-tests-using-chai-and-mocha-e9db7d8d48bc) 

首先，让我们创建文件夹和文件。

```
$ cd src
$ mkdir tests
$ touch tests/helpers.ts tests/warrior.test.ts tests/skill.test.ts
```

然后，我们要写下一些帮手。我们需要一个函数来创建战士，一个创建技能，另一个清理数据库。创建最后一个是为了避免数据间的干扰。

好了，现在我们可以为**测试/warrior.test.ts** 创建测试了。在这个文件中，我们将创建查询和变异来帮助我们编写测试。然后我们需要做一些特定的测试:

*   应该得到战士
*   难道**不应该**用无效令牌获得战士吗
*   应该得到所有战士
*   应该**不要**用无效令牌得到所有战士吗
*   应该注册一个战士
*   难道**不应该**注册一个没有战士名字的战士吗
*   难道**不应该**签约一个没有名字的战士吗
*   不应该**注册一个没有密码的战士**
*   应该登录一个战士
*   是否应该让**而不是**用错误的凭证登录战士
*   应该将一个部落升级为战士
*   **不应该**更新部落为无令牌战士

最后，我们可以测试技能。在**测试/skill.test.ts** 中，我们将测试:

*   应该从战士那里获得技能
*   应该获得所有战士的技能
*   应该得到一个对手
*   **不应该**得到一个不存在的对手(错误的 ID)
*   应该得到所有对手

# 4.结论

仅此而已。

这是教程的第二步。在本章中，你已经学习了如何使用 typescript 和 GraphQL 构建一个完整的服务器。你会在这里找到下一部分

[](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-pt-3-frontend-dc69f71e1d62) [## 动物部落:如何创建你的第一个全栈类型脚本 GraphQL 应用程序？—第三部分:前端

### 如何使用 Typescript、Node、React 和 GraphQL 构建第一个全栈应用的完整教程

medium.com](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-pt-3-frontend-dc69f71e1d62) 

其他部分你可以在这里找到。

[](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-76786e5520ed) [## 动物部落:如何创建你的第一个全栈类型脚本 GraphQL 应用程序？

### 如何使用 Typescript、Node、React 和 GraphQL 构建第一个全栈应用的完整教程

medium.com](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-76786e5520ed) [](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-e7891ec4963a) [## 动物部落:如何创建你的第一个全栈类型脚本 GraphQL 应用程序？

### 如何使用 Typescript、Node、React 和 GraphQL 构建第一个全栈应用的完整教程

medium.com](https://medium.com/@samarony.barros/animal-tribes-how-to-create-your-first-full-stack-typescript-graphql-application-e7891ec4963a) 

你可以在我的 [GitHub](https://github.com/samaronybarros) 中找到完整的项目，然后搜索[动物部落服务器库](https://github.com/samaronybarros/animal-tribes-server)。

我希望我对你的知识有所贡献。有什么地方需要改进才能写出更好的文章，欢迎随时告诉我。告诉我，如果你有文章或代码的改进。

感谢您耐心阅读本文。下一篇文章再见。

看看我的其他文章。谢谢你能来。

# 5.我的其他文章

[](https://medium.com/swlh/how-to-create-your-first-mern-mongodb-express-js-react-js-and-node-js-stack-7e8b20463e66) [## 如何创建你的第一个 MERN (MongoDB，Express JS，React JS 和 Node JS)栈

### 嗨伙计们，

medium.com](https://medium.com/swlh/how-to-create-your-first-mern-mongodb-express-js-react-js-and-node-js-stack-7e8b20463e66) [](https://medium.com/swlh/how-do-i-deploy-my-code-to-heroku-using-gitlab-ci-cd-6a232b6be2e4) [## 如何使用 GitLab CI/CD 将我的代码部署到 Heroku？

### 关于如何使用舞台和生产环境的教程

medium.com](https://medium.com/swlh/how-do-i-deploy-my-code-to-heroku-using-gitlab-ci-cd-6a232b6be2e4) [](/how-to-make-tests-using-chai-and-mocha-e9db7d8d48bc) [## 如何用柴和摩卡做测试？

### NodeJS 应用程序的 TDD

itnext.io](/how-to-make-tests-using-chai-and-mocha-e9db7d8d48bc) 

# 关于我

Sam Barros 是一名巴西人，作为一名软件工程师在柏林生活和工作。他是一个技术爱好者，他总是用自己的生活来帮助人们。

连接到:

*   领英
*   [GitHub](https://github.com/samaronybarros)
*   [中型](https://medium.com/@samarony.barros)
*   [个人网站](https://sambarros.com/)**