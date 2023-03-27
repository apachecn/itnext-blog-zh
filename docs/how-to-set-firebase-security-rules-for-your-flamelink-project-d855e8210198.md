# 如何为您的 Flamelink 项目设置 Firebase 安全规则

> 原文：<https://itnext.io/how-to-set-firebase-security-rules-for-your-flamelink-project-d855e8210198?source=collection_archive---------5----------------------->

![](img/c634905835deffb8e6df379fa259cf99.png)

所以你终于准备好把你的 Flamelink 项目的训练轮子拿下来并投入使用了。在此之前，您是否已经为您的数据库设置了适当的规则？不，真的，你应该！

> 如果这是你第一次听说 Flamelink，一个 Firebase 的 CMS，看看我们的[网站](https://flamelink.io)开始吧。将你的 Firebase 项目链接到 Flamelink 后，请回到这里阅读关于保护你的内容。

## 编辑:请注意，本文讨论的是为 Firebase 实时数据库而不是云 Firestore 设置安全规则。CF 将很快发布更多信息。

不久前，一个新的 Firebase 项目以*测试模式*发布，即。*读取*和*写入*对实时数据库上的任何人开放。从那时起，Firebase 的优秀人员决定改变这一点，默认在*锁定模式*下没有读写权限。这样做是因为许多开发人员从不费心为他们在生产中运行的项目收紧安全规则，让他们的数据库对任何人开放。

现在，当您的数据库处于锁定模式时，Flamelink 无法工作，因为我们无法从您的浏览器读取/写入数据库。在锁定模式下访问数据库的唯一方法是从服务器环境访问，这需要通过服务帐户进行访问。在 Flamelink，我们决定不走这条路，让您，最终用户，完全控制您的项目，以及您在晚上睡觉时给我们的访问级别。就我们所能提供的无缝用户体验而言，这是有代价的，将来我们可能会提供这两个选项，但我离题了。

为了快速开始使用 Flamelink，我们建议您为 RTDB(实时数据库)设置以下数据库规则:

```
{
  "rules": {
    "flamelink": {
      ".read": "auth != null",
      ".write": "auth != null",
      "users": {
        ".indexOn": ["email", "id"]
      }
    }
  }
}
```

用简单的英语来说，就是:

> 在" **flamelink** "名称空间之外没有访问权限，但对" **flamelink** "名称空间内的已验证用户有读写权限。

*用户在“email”和“id”字段上的索引只是为了更好的查询性能，对于这篇关于访问控制的文章并不重要。*

这对于快速入门来说是不错的，但是您可以想象一下，允许任何经过身份验证的用户写入您的数据库并不是生产就绪的安全性。另一方面，您可能希望任何人都可以阅读某些内容，不管他们是否登录——比如您网站上的博客帖子等。那么这怎么改善呢？让我们来看几个选项。

## 需要知道的事情

关于为 RTDB 设置安全性规则，需要了解一些事项:

1.  从服务器访问时，安全规则会被完全忽略，只有客户端(浏览器)访问时才会应用安全规则
2.  如果规则授予父节点读/写访问权限，则 DB 结构中进一步嵌套的任何其他子节点也将具有访问权限。换句话说，如果 DB 结构中更高一级的规则已经是**真**，那么您不能将规则设置为**假**。

如果您还不熟悉 RTDB 安全规则，请观看此视频，它是一个非常好的介绍:

摘自 https://firebase.google.com/docs/database/security/

## 您的应用程序或网站的读取权限

最简单的方法是授予任何人对非敏感内容的读取权限，因此我们将首先解决这个问题。

```
{
  "rules": {
    "flamelink": {
      ".read": "auth != null",
      ".write": "auth != null",
      "users": {
        ".indexOn": ["email"]
      },
      "environments": {
        "$environment": {
          "content": {
            **"nonSensitiveContentType": {
              ".read": true
            }**
          }
          "schemas": {
            **".read": true**
          }
        }
      }
    }
  }
}
```

您需要注意的是“nonSensitiveContentType”属性，您可以用特定内容类型的键替换它。这是特定于您的数据的，所以请查看您的数据库。您可以为任意多的内容类型执行此操作。如果您愿意，也可以通过设置以下内容来使所有内容可读:

```
"content": {
  **".read": true**
}
```

这正是我们在示例中对“模式”所做的。如果你使用官方的 [Flamelink JavaScript SDK](https://flamelink.github.io/flamelink) ，你将不得不授予“模式”的读取权限，因为这用于确定字段是否有效、是否相关以及其他一些好处，如缓存。

你的应用程序用户的另一个读访问选项是仍然要求用户被认证，但是使用 [Firebase 的匿名登录](https://firebase.google.com/docs/auth/web/anonymous-auth)。这样做的好处是，你的数据库只能从你的应用程序中读取(或者你是否允许对你的项目进行身份验证)，而不能通过 REST 端点读取。

## 特定用户的写权限

要将对数据库的写访问权限仅限于 Flamelink CMS 用户，您可以在规则中指定唯一的 id(uid ),如下所示:

```
{
  "rules": {
    "flamelink": {
      ".read": "auth != null",
      **".write": "auth.uid === '2TnyIXYi3FPeizykrJiLT972Oy53'",**
      "users": {
        ".indexOn": ["email"]
      }
    }
  }
}
```

您可以在您的 [Firebase 控制台](https://console.firebase.google.com)的“认证”部分找到您的用户的 UID。您也可以非常容易地指定多个 uid:

```
".write": "auth.uid === '2TnyIXYi3FPeizykrJiLT972Oy53' || auth.uid === 'LOkg1qVvLgTHWPyOkeBgrGaNuHy3'"
```

如果您决定匿名登录您的所有应用程序用户，您可以通过检查“匿名”提供商来进一步限制写入:

```
".write": "**auth.provider !== 'anonymous'**"
```

## 非常动态的规则

首先，我想说我们并不建议你必须这样做，但这是可能的。继续…

在 Flamelink 中，用户被分配到权限组，每个权限组都有一个唯一的 ID。这些权限组映射到应用程序中的特定权限。例如，一个权限组可以被配置为只允许对模式的"**视图**访问，但是对内容的完全 [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) 访问。我们可以利用这些权限组来动态限制数据库级别的访问。

对我坦白，这可能会变得很糟糕。我们将首先看一下如何在您的内容类型上实施" **view** "权限，但是同样的技术可以用于任何其他 CRUD 操作。

```
{
  "rules": {
    "flamelink": {
      ".read": "auth != null",
      ".write": "auth != null",
      "environments": {
        "$environment": {
          "content": {
            "$contentType": {
              "$locale": {
                **".read": "auth != null && root.child('flamelink').child('permissions').child(root.child('flamelink').child('users').child(auth.uid).child('permissions').val() + '').child('content').child($environment).child($contentType).child('view').val() === true"**
              }
            }
          }
        }
      }
    }
  }
}
```

哇！什么鬼东西？！好吧，让我们把它分解一下，因为这个想法很简单，语法没那么复杂。我保证会有意义的。

**想法:**获取用户的权限组，并检查该权限组是否被设置为允许对特定内容的“查看”权限。

**语法:**规则由两部分组成:获取权限组 ID，然后检查该组的权限配置。

```
root
  .child('**flamelink**')
  .child('**users**')
  .child(**auth.uid**)
  .child('**permissions**')
  .val() + ''
```

这段代码从数据库的根目录开始，一直到`flamelink.users.<uid>.permissions`，其中< uid >是试图访问数据库的用户的用户 id。这个数据库字段的值是一个整数，所以我们用`+ ''`将它转换成一个字符串，这样我们就可以在规则的下一部分使用它。

```
root
  .child('**flamelink**')
  .child('**permissions**')
  .child(**<our-previous-query>**)
  .child('**content**')
  .child(**$environment**)
  .child(**$contentType**)
  .child('**view**')
  .val() === true
```

同样，我们从 DB 的根目录开始，向下钻取，直到得到实际的权限组配置:`flamelink.permissions.<user-permission-group>.content.<environment>.<content-type>.view`。

每个权限组配置由以下 4 个布尔属性组成，这些属性映射到标准 CRUD 配置:

```
{
  **create**: true,
  **delete**: false,
  **update**: true**,
  view**: true
}
```

要检查任何其他权限，只需将“**视图**替换为“**更新**”、“**删除**或“**创建**”。

您可能还注意到了规则开头的`auth != null`部分。这是为了确保我们仍在检查用户是否登录，否则，我们所有的努力工作都会被某个没有登录的人取消。

这就是`".read"`规则。`".write"`规则类似于我们的读取，但更复杂，因为我们还需要考虑用户试图对数据做什么，以确定我们是否应该检查**创建**、**更新**或**删除**配置。

我们是勇敢的开发者，所以让我们继续。

```
{
    ".write": "**auth !== null** &&
    ((**!data.exists()** &&
      root
        .child('flamelink')
        .child('permissions')
        .child(
          root
            .child('flamelink')
            .child('users')
            .child(auth.uid)
            .child('permissions')
            .val() + ''
        )
        .child('content')
        .child($environment)
        .child($contentType)
        .child('**create**')
        .val() === true) ||
      (**!newData.exists()** &&
        root
          .child('flamelink')
          .child('permissions')
          .child(
            root
              .child('flamelink')
              .child('users')
              .child(auth.uid)
              .child('permissions')
              .val() + ''
          )
          .child('content')
          .child($environment)
          .child($contentType)
          .child('**delete**')
          .val() === true) ||
      (**data.exists() && newData.exists()** &&
        root
          .child('flamelink')
          .child('permissions')
          .child(
            root
              .child('flamelink')
              .child('users')
              .child(auth.uid)
              .child('permissions')
              .val()
          )
          .child('content')
          .child($environment)
          .child($contentType)
          .child('**update**')
          .val() === true))"
  }
```

现在我们扯掉了绷带，这里发生了什么？

除了对登录用户的`auth != null`检查，我们的规则有 3 个不同的部分，每个部分处理不同的动作(创建、删除和更新)。

对于我们的 **create** 动作，我们利用 Firebase 的`data.exist()`方法来检查特定内容当前是否不存在数据。这就是我们知道有人试图添加新数据的方式。

对于我们的**删除**操作，我们使用`newData.exists()`方法来检查新数据是否不存在。如果用户的行为不会产生新的数据，我们知道他们试图删除一些东西。

对于我们最后的**更新**动作，我们结合了`data.exists()`和`newData.exists()`方法来确定用户试图将现有数据更改为其他数据。

还不算太糟，是吗？

有关如何应用的完整示例，请参见本[要点](https://gist.github.com/jperasmus/73741d5d86ad146b56b61f552786c167)。

这种方法并非没有局限性。由于 Flamelink 是一款常青树，并且不断发展的产品，因此会不断添加新功能，这可能会导致新节点添加到数据库中。如果您对数据库的限制太多，以至于我们无法对您的数据库结构进行必要的更新，您将无法使用这些闪亮的新特性。您可以通过将我们前面看到的 UID 特定规则与这个动态设置相结合来解决这个问题，并确保如果当前登录的用户是项目的所有者，则可以对数据库进行任何写入。这将确保当推出新功能并且所有者登录到项目中时，应用必要的数据库结构更改。

> 尽管如此，我们很少进行结构上的改变，因为产品的常青特性。

## Firebase 自定义声明

我们把最好的留到了最后。最有说服力的解决方案是使用 Firebase 鲜为人知的特性:[自定义声明](https://firebase.google.com/docs/auth/admin/custom-claims)。我们很乐意将带有自定义声明的 Flamelink 开箱即用，但自定义声明只能使用 Firebase Admin SDK 从特权服务器环境中设置。这意味着你，项目所有者，将不得不自己处理这个问题。

**什么是自定义声明？**

简单地说，自定义声明是在用户帐户上设置的自定义属性。例如，您可以为用户设置一个`isAdmin`属性。这非常强大，因为它提供了在 Firebase 应用程序中实现各种访问控制策略的能力，包括基于角色的访问控制。令人惊奇的是，这些自定义属性可以用在数据库的安全规则中。

**关于如何使用它们的一些想法**

自定义声明应仅用于访问控制，而不是存储任何其他用户数据。最好在数据库中存储额外的数据。

在设置自定义声明时，您可以保持简单，并在所有 Firebase 用户上设置一个名为`flamelinkUser`的属性，该属性应该对内容具有写访问权限。或者，您可以将声明设置为您喜欢的详细声明，但是请记住自定义声明有效负载不应超过 1000 字节的限制。建议将其设置得尽可能小，因为这些声明会随所有网络请求一起发送，并且较大的负载会对性能产生负面影响。

**如何在我们的安全规则中使用这些自定义声明？**

一旦设置好，在我们的数据库安全规则中检查自定义声明就变得非常容易。所有自定义声明都在经过身份验证的用户的身份验证令牌上设置。

```
{
  "rules": {
    "flamelink": {
      ".read": "auth != null",
 **".write": "auth.token.flamelinkUser === true"**
    }
  }
}
```

**如何为您的用户设置自定义声明？**

设置自定义声明的唯一要求是使用 Firebase Admin SDK 从服务器环境中进行设置，无论是使用您运行的独立 Express 服务器还是使用 Firebase 的[云功能，都由您决定。代码如下所示(示例使用 JavaScript，但是您可以使用任何受支持的服务器端语言):](https://firebase.google.com/docs/functions/)

```
// import admin SDK
**const admin = require('firebase-admin');**// initialize admin app with any of the supported options
**admin.initializeApp(/* config here */);**// create your custom claims object (whatever you want)
**const customClaims = {
  flamelinkUser: true
};**// set the custom claims object for given UID
**admin.auth().setCustomUserClaims(user.uid, customClaims)**
```

`admin.auth().setCustomUserClaims()`方法返回一个承诺。值得注意的是，设置新的自定义声明会覆盖任何现有的自定义声明，因此您可能希望在再次设置之前先检索现有的声明并对其进行更新。

## 结论

希望这能让您了解 Firebase 安全规则是多么强大和灵活。我鼓励你在 [Firebase 的文档](https://firebase.google.com/docs/database/security/)中阅读更多关于这些规则的内容。

如果您对我们如何改进这些安全规则有任何其他想法，请在下面的评论中告诉我们，或者加入我们的 [Slack 社区，](https://flamelink.io/slack)我们非常欢迎您的加入。