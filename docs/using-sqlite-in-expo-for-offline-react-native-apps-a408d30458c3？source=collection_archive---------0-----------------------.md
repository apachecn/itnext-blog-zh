# 在博览会中使用 SQLite 进行离线反应本地应用程序

> 原文：<https://itnext.io/using-sqlite-in-expo-for-offline-react-native-apps-a408d30458c3?source=collection_archive---------0----------------------->

![](img/09ef437497c733f4d5a94d7e5c65f1fe.png)

决定一个应用程序是否“感觉”本土的最大因素之一是它的速度。很多让应用感觉快速的方法都是优化关键的热门代码路径和拥有正确的动画，但是对于大多数应用来说，至少会有一些部分涉及读写数据。

虽然用户完全离线变得越来越罕见，但尤其是移动用户，网络连接不良仍然很常见。如果你想让你的应用在网络慢的时候感觉很快，你至少需要让你的应用的某些方面脱机工作。

就本文而言，我假设您正在使用[世博会](https://expo.io/)。如果你不熟悉 Expo，这是一个使用 react-native 为 iOS 和 Android 构建本地应用程序的好方法。它基本上让您免去了为构建本机应用程序而设置环境的所有麻烦，并让您可以直接编写 JavaScript/TypeScript。

因此，您的应用程序需要存储数据，但也需要(至少部分)脱机工作。这意味着您需要一种在本地存储一些数据的方法。如果您想避免编译本机模块(并离开 expo 生态系统)，您有 3 个基本选项:

1.  SecureStore
2.  AsyncStorage
3.  SQLite

# 安全商店

> expo-secure-store 提供了一种在设备本地加密和安全存储密钥-值对的方法

如果你需要储存任何秘密钥匙。例如您的 API 的用户认证令牌，安全存储是唯一明智的选择。这是确保静态数据加密的唯一选择，但也只适用于相当少量的数据。如果您需要加密大量数据，您可以在 SecureStore 中存储和加密密钥。

```
**import** * **as** SecureStore **from** 'expo-secure-store';**await** SecureStore.setItemAsync('token', 'my-token-value');**const** myToken = **await** SecureStore.getItemAsync('token');
```

根据我在 iOS 上的经验，如果你的应用程序请求令牌时设备还没有完全解锁，这可能会引发异常。因此，围绕`getItemAsync`方法增加一次快速重试可能是值得的。如果你想做出不同的安全权衡，你可以选择 iOS 上的`keychainAccessible`选项，但是默认是合理的。

# 异步存储

> AsyncStorage 是一个未加密、异步、持久的键值存储系统，对应用程序来说是全局的。

如果您已经在网络上使用了本地存储，异步存储会感觉非常相似。这对于存储相当少量的数据，甚至相当大量的数据非常有用，只要您不需要任何聪明的查询方法。它非常适合存储应用程序的当前导航状态，或者用户将要添加的帖子的草稿。这对于像目前游戏中的高分这样的事情也很有用。

```
**import** { AsyncStorage } **from** 'react-native';**await** AsyncStorage.setItem('token', 'my-token-value');**const** myToken = **await** AsyncStorage.getItem('token');
```

注意，如果您想要存储复杂的 JSON 对象，您将需要使用`JSON.stringify`和`JSON.parse`。这些数据在静态时也没有加密。

# SQLite

> [@databases/expo](https://www.atdatabases.org/docs/websql) 提供了一个 SQL 数据库，它可以在应用程序重启后保持不变，并支持事务以及高级过滤和查询。

[expo-sqlite](https://docs.expo.io/versions/v36.0.0/sdk/sqlite/) 模块提供了一个基于 WebSQL 接口的 SQL 数据库。这非常强大，支持 SQLite 的几乎所有特性。SQLite 也非常适合有离线需求的应用程序。它可以让你在磁盘上存储大量的结构化数据，并把显示当前屏幕所需的部分只读到内存中。

不幸的是，虽然 SQLite 有一个不错的 API，但 WebSQL API 太差了，最终被放弃作为 web 的标准。幸运的是`[@databases/expo](https://www.atdatabases.org/docs/websql)`将 WebSQL API 包装成更有用的东西。

首先，您需要“连接”到您的数据库:

```
**import** connect, {sql} **from** '@databases/expo';**const** db = connect('my-database');
```

您的应用程序可以有多个不同名称的数据库。您不需要显式地创建数据库，如果它不存在，系统会自动为您创建。

`[@databases/expo](https://www.atdatabases.org/docs/websql)`中的交易使用生成器函数。由于 expo/websql 公开的 API 的限制，它们不能使用异步函数。这实际上意味着，在一个事务中，你使用`function*`而不是`async function`，使用`yield`而不是`await`。您也不能在事务中执行任何其他异步操作。

您要做的第一件事是创建一些表。对数据库模式进行版本化是一个好主意，这样您就可以随时升级到最新版本。

```
**const** ready = db.tx(**function*** (tx) {
  **yield** tx.query(sql`
    **CREATE** **TABLE** **IF** **NOT EXISTS** schema_version (
      version **INT** **NOT** **NULL**
    );
  `);
  **const** versionRecord = **yield** tx.query(sql`
    **SELECT** version **FROM** schema_version;
  `);
  **const** version = (
    versionRecord.length
      ? versionRecord[0].version
      : 0
  );
  **if** (version < 1) {
    **yield** tx.query(sql`
      **CREATE** **TABLE** tasks (
        id **TEXT NOT NULL PRIMARY KEY**,
        name **TEXT NOT NULL**,
        completed **BOOLEAN NOT NULL**
      );
    `);
  }
  // to add other versions in the future,
  // we can just add extra if statements
  // and increase LATEST_VERSION
  **const** LATEST_VERSION = 1;
  **if** (version === 0) {
    **yield** tx.query(sql`
      **INSERT** **INTO** schema_version
      **VALUES** (${LATEST_VERSION});
    `);
  } **else** {
    **yield** tx.query(sql`
      **UPDATE** schema_version
      **SET** version = ${LATEST_VERSION};
    `);
  }
});
```

然后，我们可以公开一个简单的 API 来读写数据库中的 TODOs。 [@databases/expo](https://www.atdatabases.org/docs/websql) 会隐式地为我们创建事务，只要我们调用`db.query`

```
**async function** setTodo(id, name, completed) {
  **await** db.query(sql`
    **INSERT INTO** tasks (id, name, completed)
    **VALUES** (${id}, ${name}, ${completed})
    **ON CONFLICT** (id) **DO UPDATE**
    **SET** name=excluded.name, completed=excluded.completed;
  `);
}**async function** getTodo(id) {
  **return** (**await** db.query(sql`
    **SELECT** * **FROM** tasks **WHERE** id=${id};
  `))[0] || **undefined**;
}
```

注意,“completed”字段将作为`0`或`1`返回，因为在存储层，SQLite 并不真正区分数字和布尔值。

在这一点上，看起来我们有了与 AsyncStorage 非常相似的东西(即一个简单的键值存储)，但是现在可以编写更高级的查询，因为我们拥有 SQL 的全部功能。

```
**async function** getUnfinishedTodos() {
  **return await** db.query(sql`
    **SELECT** * **FROM** tasks **WHERE** completed=false;
  `);
}
```

# 结论

*   使用`[SecureStore](https://docs.expo.io/versions/v36.0.0/sdk/securestore/)`存储 API 密钥和加密密钥，因为它经过适当加密，并且在手机锁定时无法从备份/访问。
*   将`[AsyncStorage](https://docs.expo.io/versions/v36.0.0/react-native/asyncstorage/)`用于 JSON/text 的简单 blobs。
*   当你需要能够查询你的数据时，使用`[@databases/expo](https://www.atdatabases.org/docs/websql)`。(如果你愿意，你可以直接使用 [expo API](https://docs.expo.io/versions/v36.0.0/sdk/sqlite/) ，但是很难保证它可靠地工作)