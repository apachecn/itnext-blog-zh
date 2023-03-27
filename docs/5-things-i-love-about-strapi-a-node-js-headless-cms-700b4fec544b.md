# 我喜欢 Strapi 的 5 点，一个 Node.js 无头 CMS

> 原文：<https://itnext.io/5-things-i-love-about-strapi-a-node-js-headless-cms-700b4fec544b?source=collection_archive---------0----------------------->

构建高质量的 REST 和 GraphQL APIs 很难。但是有一种简单的方法可以在几分钟内开始。

![](img/99f31236cdbe5713a5006c64b3404eb6.png)

约翰·贝克在 [Unsplash](https://unsplash.com) 上拍摄的照片

一年前，我开始为一个非常受欢迎的网站开发后端。我尝试过不同的方法、不同的技术，甚至不同的编程语言。然后我开始寻找一个可以帮我轻松集成几个功能的 CMS…但是找不到让我满意的 CMS。然后我发现了 [Strapi](https://strapi.io/) ，一个惊人的开源无头 CMS，拥有我所寻找的一切。我不打算深入回顾它；我想重点谈谈我喜欢这款出色的 CMS 的五点:

# 1)不到 2 分钟即可开始

当我开始用 CMS 建立网站时，我从 WordPress 开始。我一直认为，“没有比开始使用 CMS 更简单的方法了！”嗯…我完全错了！有了 Strapi，一分半钟就能上手！

现在您可能想知道:“好极了，但是我如何设置我的数据库连接呢？”。惊人的问题！在 Strapi 中，您有三种不同的环境:开发、试运行和生产。默认情况下(在开发环境中)，Strapi 将建立一个 SQLite 连接，因此您不需要仅仅为了开发过程而启动一个 MongoDB/Postgres/MySQL 实例！我为什么提到这三个数据库？因为 Strapi 是数据库无关的。您可以轻松地从一个数据库切换到另一个数据库(我们实际上在这个网站上从 Postgres 迁移到 MySQL 只用了 15 分钟！).

你不喜欢默认的数据库连接？没关系；您可以如下引导您的新 Strapi 实例:

```
npx create-strapi-app
```

一个提示将要求您提供建立数据库连接所需的所有信息。当您准备在 staging 中测试您的 Strapi 实例时，您可以只为该特定环境指定一个数据库连接(对于生产环境也是如此)！相当牛逼！

# 2)开箱即用的 GraphQL

GraphQL 是一项了不起的技术。不久前，我已经在“[编写 GraphQL 服务器](https://medium.com/openmindonline/js-monday-12-building-a-graphql-server-c8741684f203)”一文中提到过。许多 CMS 公开了 GraphQL APIs，但只有少数是完全免费和开源的。一开始，我只是想测试一下 Strapi 的功能，GraphQL 插件让我大吃一惊！您可以从后台安装它，并且您将准备好在 30 秒内运行您的查询！

我保证，这些 GraphQL 查询的质量会让你大吃一惊！绝对是我见过的最简单和高质量的 GraphQL 界面。

# 3)权限

当您构建 API 时，您可能需要将它们的访问权限限制在特定的用户组。例如，您可能有一些必须保护且不能公开共享的敏感数据。Strapi 使用其令人惊叹的仪表板使管理这种情况变得极其容易:

只需几秒钟，您就可以设置数量惊人的权限规则来保护您的 REST/graph QL API！但是，如何认证才能访问这些路由呢？Strapi 发布了一个 [JWT 令牌](https://jwt.io/)，您可以在 HTTP 请求的授权头中使用它。相当惊人！

# 4)关系

处理 Strapi 实例中数据之间的关系非常简单。假设您正在创建自己的博客，并且有多个作者:在您的数据库中，您可能有“用户”和“文章”两个表，对吗？你将如何处理文章和作者(即“用户”)之间的关系？

真的那么容易吗？是的，它是！您可以仅使用其后台办公室来处理 Strapi 实例内部的大量关系，这总是让我无话可说。

# 5)程序化

大多数情况下，当您使用 CMS 时，您需要进行一些定制，以使其行为适应您的业务需求。嗯，Strapi(再一次)有一个惊人的解决方案，可以根据您的需求定制它的行为。我还没有提到，您使用 Strapi backoffice 定制的大多数设置都保存在文件系统的一些配置文件中。例如，我们之前已经看到，我们在数据库中创建了一个“Articles”表。如果您正在浏览 Strapi 文件夹中的`/api/articles`,您将找到以下目录:

*   配置
*   控制器
*   模型
*   服务

这可能会让您想起 MVC 框架…可能是因为 Strapi 的行为就像 MVC 框架一样！事实上，您可以通过 API 或编辑上述目录中的文件来完成相同的后台任务。例如，你想编辑你的 API 路由，你可以自定义`/api/articles/config/routes.json`文件:

```
{
  "routes": [
    {
      "method": "GET",
      "path": "/articles",
      "handler": "Articles.find",
      "config": {
        "policies": []
      }
    },
    {
      "method": "GET",
      "path": "/articles/count",
      "handler": "Articles.count",
      "config": {
        "policies": []
      }
    },
    {
      "method": "GET",
      "path": "/articles/:id",
      "handler": "Articles.findOne",
      "config": {
        "policies": []
      }
    },
    {
      "method": "POST",
      "path": "/articles",
      "handler": "Articles.create",
      "config": {
        "policies": []
      }
    },
    {
      "method": "PUT",
      "path": "/articles/:id",
      "handler": "Articles.update",
      "config": {
        "policies": []
      }
    },
    {
      "method": "DELETE",
      "path": "/articles/:id",
      "handler": "Articles.delete",
      "config": {
        "policies": []
      }
    }
  ]
}
```

在`config.policies`中，您可以定义权限规则，允许通过身份验证调用特定的路由。您还可以通过编辑`handler`属性来添加和定制由特定路线触发的控制器。但是在哪里为控制器编写 Node.js 代码呢？在`/api/articles/controllers/Articles.js`文件里面！

```
'use strict';
/**
 * Read the documentation (https://strapi.io/documentation/3.0.0-beta.x/concepts/controllers.html#core-controllers)
 * to customize this controller
 */
module.exports = {
  myNewController: async (ctx) => {
    ctx.body = {
      jsonResponse: true
    }
  }
};
```

Strapi 构建在一个流行而强大的 Node.js web 框架 [Koa](http://web.archive.org/web/20210118101749/https://koajs.com/) 之上，所以你可能已经熟悉它的功能了！关于 Strapi，我想提到的最后一点是它令人惊叹的回调系统。打开`/api/articles/models/Articles.js`文件，您将获得一个高级回调系统，它将对您改进 CMS 行为有很大帮助:

```
"use strict";
/**
 * Lifecycle callbacks for the `Articles` model.
 */
module.exports = {
  // Before saving a value.
  // Fired before an `insert` or `update` query.
  beforeSave: async (model, attrs, options) => {},
  // After saving a value.
  // Fired after an `insert` or `update` query.
  afterSave: async (model, response, options) => {},
  // Before fetching a value.
  // Fired before a `fetch` operation.
  beforeFetch: async (model, columns, options) => {},
  // After fetching a value.
  // Fired after a `fetch` operation.
  afterFetch: async (model, response, options) => {}, // Before fetching all values.
  // Fired before a `fetchAll` operation.
  beforeFetchAll: async (model, columns, options) => {},
  // After fetching all values.
  // Fired after a `fetchAll` operation.
  afterFetchAll: async (model, response, options) => {},
  // Before creating a value.
  // Fired before an `insert` query.
  beforeCreate: async (model, attrs, options) => {},
  // After creating a value.
  // Fired after an `insert` query.
  afterCreate: async (model, attrs, options) => {},
  // Before updating a value.
  // Fired before an `update` query.
  beforeUpdate: async (model, attrs, options) => {},
  // After updating a value.
  // Fired after an `update` query.
  afterUpdate: async (model, attrs, options) => {},
  // Before destroying a value.
  // Fired before a `delete` query.
  beforeDestroy: async (model, attrs, options) => {},
  // After destroying a value.
  // Fired after a `delete` query.
  afterDestroy: async (model, attrs, options) => {}
};
```

如您所见，您可以定制 API 的每一个 CRUD 行为。这可能是我最喜欢的 Strapi 特性，因为它帮助我只用几行代码和最少的时间就实现了每个目标。

# 就这些吗？

当然没有！Strapi 是一个令人难以置信的工程，它允许您非常快速地构建高质量的 REST/graph QL API。我只是想从众多的功能中选出我最喜欢的五个特性，这些功能使 Strapi 成为一个完整的、可投入生产的 CMS。如果你想了解更多关于 Strapi 的知识，你可以在这里阅读它的文档:[https://strapi.io/documentation](https://strapi.io/documentation)。

[![](img/e05f00ed2cddfd2907284cb397168c3d.png)](https://github.com/sponsors/micheleriva)