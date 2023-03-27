# 使用 Node.js 在 Postgres 中存储 JSON

> 原文：<https://itnext.io/storing-json-in-postgres-using-node-js-c8ff50337013?source=collection_archive---------0----------------------->

![](img/ce931d3e75a6519cb51f1653e27b85f5.png)

[布兹](https://unsplash.com/@ladybugz_?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)女士在 [Unsplash](https://unsplash.com/images/animals/elephant?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

让 MongoDB 这样的 NoSQL 数据库如此受欢迎的一个原因是，当您需要的时候，只需添加一个 JSON 就可以了。然而，您可能没有意识到的是，Postgres 对 JSON 的支持几乎一样好。除了在字段中抛出一些 JSON 之外，您还可以使用所有常见的 SQL 操作来查询它。你得到`JOIN` s，交易，指数等。

# JSON 与 JSONB

向 postgres 添加 JSON 字段时，第一步是在 JSON 和 JSONB 之间做出决定。我给你简单说一下:

> 永远用 JSONB，千万不要用 JSON

JSONB 就像 JSON 一样，只是它不存储 JSON 的实际字符串，而是存储一个有效的二进制表示。您可能想要存储 JSON 的唯一原因是，如果您想要跟踪原始 JSON 序列化中的空白。这应该是不需要的，如果你想要 JSON 的“漂亮”视图，你可以使用`JSON.stringify`来美化它。我无法想象为什么你会需要原来的空白。

# 创建表

既然已经决定使用`JSONB`作为格式，就可以像平常一样创建表格了。

```
**CREATE** **TABLE** my_data (
  id **TEXT NOT NULL PRIMARY KEY**,
  data **JSONB NOT NULL**
);
```

这创建了一个表，它有一个名为`id`的主键，类型为`TEXT`，还有一个`data`列来存储我们的 JSON 数据。

# 读取和写入 JSON 数据

如果您使用的是`[@databases/pg](https://www.atdatabases.org/docs/pg)`客户端，您可以像读取任何其他值一样读取和写入 Postgres 数据:

```
**import** connect, {sql} **from** '@databases/pg';**const** db = connect();**export** **async** **function** get(id) {
  **const** [row] = **await** db.query(
    sql`
      **SELECT** data
      **FROM** my_data
      **WHERE** id=${id}
    `
  );
  **return** row ? row.data : null;
}**export** **async** **function** set(id, value) {
  **await** db.query(sql`
    **INSERT** **INTO** my_data (id, data)
    **VALUES** (${id}, ${value})
    **ON** **CONFLICT** id
    **DO** **UPDATE** **SET** data = EXCLUDED.data;
  `);
}
```

这为使用 postgres 数据库的 JSON blobs 提供了一个简单的键值存储。

# 查询 JSON

假设我们在“NoSQL 博客数据库”中存储了一些博客文章:

```
**await** set('post-a', {
  author: 'ForbesLindesay',
  title: 'Post A',
  body: 'This post is about the letter A',
});**await** set('post-b', {
  author: 'ForbesLindesay',
  title: 'Post B',
  body: 'This post is about the letter B',
});**await** set('post-a-rebuttal', {
  author: 'JoeBloggs',
  title: 'Post A - Rebuttal',
  body: 'Forbes was wrong about the letter A',
});
```

现在假设我们想在`ForbesLindesay`之前获得所有博客文章的列表。author 字段隐藏在 JSONB 字段中，但这并不意味着我们不能在查询中使用它。

```
**export** **async** **function** listByAuthor(author) {
  **return** **await** db.query(
    sql`
      **SELECT** data
      **FROM** my_data
      **WHERE** data ->> 'author'
          = ${author}
    `
  );
}
```

在这里，`->>`操作符的意思是“获取这个属性的值”。只有当值是字符串、数字或布尔值时，它才起作用。如果值是另一个对象，就必须使用`->`操作符，意思是“以 JSON 的形式获取这个属性的值”。

很明显，这意味着您可以在这里使用 SQL 的全部功能，但仅举另一个例子，我们可以获得所有作者的列表:

```
**export** **async** **function** getAuthors() {
  **return** (**await** db.query(
    sql`
      **SELECT** **DISTINCT** data ->> 'author' as author
      **FROM** my_data
    `
  )).map(({author}) => author);
}
```

这里，我们从数据中提取作者，然后使用 SQL 的`DISTINCT`操作符只返回每个作者一次。

# 结论

在 Postgres 中，您可以像使用任何其他值一样使用 JSON，为 JSON blobs 建立一个类似 NoSQL 的商店并将其用作整个数据库会很容易。这不一定意味着你**应该**。这个 JSON 数据完全是无模式的，所以在将它插入数据库之前，正确地验证它是否匹配任何预期的结构是非常重要的。当您需要存储大型 JSON 结构并且您还不确定如何查询它们时，这非常有用，但是在大多数情况下，我仍然建议在 SQL 中使用显式字段，并使用连接来存储嵌套列表等。