# NestJS 入门

> 原文：<https://itnext.io/getting-started-with-nestjs-307556e02c8d?source=collection_archive---------1----------------------->

## 如何用 TypeScript & NestJS 构建 web 服务

![](img/9bc90a0d9bbf4975d372db4e03230e9d.png)

NestJS 标志—[https://nestjs.com](https://nestjs.com)

## 介绍

今天的开发人员在构建 web 服务和其他服务器端应用程序时有很多选择。Node 已经成为一种流行的选择，然而许多程序员更喜欢比 JavaScript 更健壮的语言，尤其是那些来自现代 OOP 语言的语言，如 C#、C++或 Java(仅举几例)。虽然 [TypeScript 非常适合 Node](/server-side-typescript-with-node-c5cef1584684) ，但 NestJS 框架将它带到了一个全新的水平，为现代服务器端开发人员提供了最先进的工具，以使用组件、提供者、模块和其他有用的高级抽象来构建持久的高性能应用程序。

在本文中，我们将研究用 NestJS 构建一个简单的 API 服务器的过程，以处理一个基本的应用程序场景:创建、存储和检索一个普通商店的产品列表。

如果你想要这个项目的源代码副本，[点击这里](https://github.com/kenreilly/nest-js-example)查看。

## **项目设置**

用 Nest 构建需要一个节点环境。如果你还没有的话，[去他们的网站](https://nodejs.org/en/download/)为你的操作系统下载一份。

安装框架很简单:

```
**$** npm i -g @nestjs/cli 
```

此项目是通过运行以下命令使用 Nest CLI 工具创建的:

```
**$** nest new nest-js-example
```

这个命令将使用所需的配置文件、文件夹结构和服务器样板文件创建一个全新的嵌套项目。

## 应用程序入口点

配置和运行服务器的主文件是 **src/main.ts:**

该文件导入了用于创建应用程序的 NestFactory 类，以及主应用程序模块文件(我们稍后将对此进行研究)，然后通过实例化它并监听端口 *3000* 来引导应用程序。

## 应用模块

声明 app 组件的文件是 **src/app.module.ts** :

该文件用作导入其余应用程序组件并在模块中声明的位置，该模块被导入到我们之前检查的文件(main.ts)中。当被指示生成新组件时，[嵌套 CLI 工具](https://docs.nestjs.com/cli/usages)将根据需要自动更新该文件。这里，*项目*的控制器和服务被导入并添加到模块中。

## 项目控制器

我们要检查的下一个文件是**src/items/items . controller . ts**:

该文件定义了用于创建项目和检索先前创建的项目列表的控制器。这里导入了几个关键组件:

*   **createitemdo**:一个[数据传输对象](https://docs.nestjs.com/controllers#request-payloads)，定义项目数据如何通过网络发送(即它是 JSON 数据结构)
*   **ItemsService** :一个[提供者](https://docs.nestjs.com/providers)，负责处理或存储项目数据
*   **项目**:定义项目内部数据结构的接口

装饰器`@Controller('items')`向框架表明这个类将服务于 REST 端点`/items`，并且 **ItemsController** 的构造器获取了 **ItemsService** 的一个实例，它在内部使用该实例来服务于两个 HTTP 方法:

*   `POST /items`(根据请求 JSON 创建新项目)
*   `GET /items`(检索之前创建的项目列表)

对这两个方法的请求由 *create* 和 *findAll* 方法处理，这两个方法通过`@Post()`和`@Get()`装饰器绑定到各自的 HTTP 方法。装饰器也可以以同样的方式支持额外的方法，比如`@Put()`或`@Delete()`等。

## 项目对象接口

接下来是两个定义存储项接口的文件，一个用于内部使用，比如编译时类型检查( **Item** )，另一个用于定义传入 JSON 的预期结构的外部接口(**createitemdo**):

**条目**接口为一个典型的商店条目定义了三个属性:名称、描述和价格。这确保了应用程序设计中不会混淆项目是什么或它有什么属性。

**createitemdo**类反映了**项**中的属性，用`@IsNotEmpty()`修饰每个属性，以确保 REST API 端点需要所有这些属性。

这两个类的所有属性都是强类型的，这是 TypeScript 的主要优点之一(因此得名)。乍一看，这提高了上下文理解的水平，并且当与实时代码分析工具(例如 VSCode 中的 [IntelliSense)一起使用时，大大减少了开发时间。对于拥有数百甚至数千个不同的类和接口的大型项目来说尤其如此。对于我们这些缺乏无限容量的完美照相记忆的人(比如我自己)，这远比试图随意记住成千上万的具体细节要容易得多。](https://code.visualstudio.com/docs/editor/intellisense)

## 项目服务

最后是创建和检索项目的服务， **items.service.dart** :

**ItemsService** 类为 Item 对象定义了一个简单的数组，该数组基本上充当我们的示例项目的易失性内存数据存储。写入和读取该“数据存储”属性的两种方法是:

*   创建(将一个**项目**保存到列表中并返回其 id)
*   findAll(返回先前创建的**项目**对象的列表

## 测试它

要启动服务器，使用标准的`npm run start`命令。一旦启动并运行，就可以通过从 CURL 发出 HTTP 请求来测试应用程序:

```
**$** curl -X POST localhost:3000/items -d '{"name":"trinket", "description":"whatever", "price": 42.0}'
```

运行这个命令将返回一个 JSON 响应，带有创建的项目的 id*和*。要查看已经创建的项目列表:

```
**$** curl localhost:3000/items
```

上面对/items 的 GET 请求将返回当前存储在内存中的项目的 JSON 响应。输出应该类似于:

```
[{"{\"name\":\"trinket\", \"description\":\"whatever\", \"price\": 42.0}":""}]
```

## 结论

相对而言，NestJS 是服务器端开发游戏的新成员，它具有一系列用于快速构建和部署企业级服务的特性，这些服务支持现代应用程序客户端的需求，并遵循诸如 [SOLID](https://en.wikipedia.org/wiki/SOLID) 和[十二因素应用程序方法论](https://12factor.net)等原则。

欲了解更多信息，请点击查看 NestJS 网站。

谢谢你看我的文章。编码快乐！

> 肯尼斯·雷利( [8_bit_hacker](https://twitter.com/8_bit_hacker) )是 [LevelUP](https://lvl-up.tech/) 的 CTO