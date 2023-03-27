# 一个带标记的模板文字实用程序，用于简化参数化 SQL 查询的编写

> 原文：<https://itnext.io/a-tagged-template-literals-utility-for-ease-of-composing-parameterized-sql-queries-f507e159466e?source=collection_archive---------3----------------------->

![](img/8426c170356a53860f288ce0c1135168.png)

照片由[卡斯帕·卡米尔·鲁宾](https://unsplash.com/@casparrubin?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

# 模板文字

从 ES6 (JavaScript 2015)开始，“模板文字”功能可用，并提供了一种将变量和表达式插入字符串的简单方法。例如:

```
const firstName = "Joe";
const lastName = "Bloggs";const message = `Welcome ${firstName}, ${lastName}!`;
console.log(message);
// will output: Welcome Joe, Bloggs!
```

当您必须手动编写 SQL 查询时，这种语法可能也很方便:

```
const sqlQuery = `SELECT * FROM users WHERE id='${userId}'`;
```

但是，这种方法可能(并且经常)导致 SQL 注入漏洞，因为参数未经适当处理就直接嵌入到 SQL 字符串中。

# 参数化 SQL 查询

为了避免 SQL 注入漏洞，大多数数据库客户端库都支持“参数化 SQL 查询”。以“ [node-postgres](https://node-postgres.com/features/queries) 为例，可以发送如下参数化的 SQL 查询:

```
client.query("SELECT * FROM users WHERE id = $1", [userId]);
```

对于上面的参数化 SQL 查询，参数替换不会发生在客户端。事实上，未更改的查询文本`SELECT * FROM users WHERE id = $1`将与参数值`userId`一起发送到 PostgreSQL 服务器。然后，该参数将被安全地替换到查询中，并在服务器端使用久经考验的参数替换代码。

尽管有 SQL 注入漏洞防御的好处，但在您的代码中构建复杂的参数化 SQL 查询并保持正确的参数顺序可能很困难，尤其是在构建包含条件查询条件的查询时。这里有一个例子:

```
const parameters = [];
const conditions = [];if (req.query["field1"]) {
  conditions.push("field1 = $1");
  parameters.push(req.query["field1"]);
}
if (req.query["field2"]) {
  conditions.push("field2 = $2");
  /* The query might break here. when `req.query["field1"]` doesn't have a value, this will be the first condition. Thus, thr query string should be "field2 = $1". */
  parameters.push(req.query["field2"]);
}const where = conditions.length ? " WHERE " + conditions.join(" AND ") : "";client.query("SELECT * FROM users" + where, parameters);
```

# 标记的模板文字

当仔细观察上面的问题时，我们会意识到该问题来自于连接查询片段的需要，同时用查询片段保持所涉及的参数的上下文信息。“带标签的模板文字”正是解决这个问题的合适工具。

标记模板文本是模板文本的更高级形式。它允许你用一个函数来解析模板文字。该函数接收占位符值列表，并返回一个值作为字符串插值结果。字符串插值结果通常是字符串。但是该函数可以选择返回一个更高级的数据结构(例如一个对象),该数据结构携带查询文本字符串和相关参数。这允许我们在字符串插值/复杂查询构造期间，始终将查询文本字符串和相关参数作为一个整体来处理。

# SQLSyntax 实用工具

“ [SQLSyntax](https://github.com/t83714/SQLSyntax) ”是一个 nodejs 库，它是基于上面类似的思想创建的。最初的想法实际上来自 scala 世界中流行的 [ScalikeJDBC](http://scalikejdbc.org/) 库。“ [SQLSyntax](https://github.com/t83714/SQLSyntax) ”库提供了一个`sqls`函数，该函数总是产生一个 SQLSyntax 类的实例作为字符串插值结果。通过利用 es6 的标记模板文字特性，用户可以享受模板文字语法的便利，同时确保所涉及的参数的上下文信息总是得到适当的维护，并与涉及参数的查询片段保持一致。

下面有一个简单的例子说明它的基本用法:

```
import SQLSyntax, {sqls} from "sql-syntax";// the return value is an instance of SQLSyntax class
const query:SQLSyntax = sqls`SELECT * FROM users WHERE user_id = ${userId} AND number = ${number}`;// we can generate SQL query text string & binding parameters array for querying database. 
const [sql, parameters] = query.toQuery();// sql: "SELECT * FROM users WHERE user_id = $1 AND number = $2"
// parameters: [userId, number]
const result = await client.query(sql, parameters);// Or more concisely using spread syntax for function calls
const result = await client.query(...query.toQuery());
```

`sqls`函数还可以识别任何作为字符串插值传递的 SQLSyntax 类实例值。当发生这种情况时，`sqls`函数将合并所有相关查询片段的 SQL 查询文本字符串(由 SQLSyntax 类实例值表示)，创建一个新的参数列表并返回一个新的 SQLSyntax 类实例。

这里有一个例子:

```
import SQLSyntax, {sqls} from "sql-syntax";// create 2 query fragments that involve parameters
const condition1:SQLSyntax = sqls`user_id = ${userId}`;
const condition2:SQLSyntax = sqls`number = ${number}`;const [sql1, parameters1] = condition1.toQuery();
// sql1: "user_id = $1"
// parameters1: [userId]// Create the final query by passing the query fragments
const query:SQLSyntax = sqls`SELECT * FROM users 
WHERE ${sqls`field1=${field1Val}`} AND ${condition1} AND ${condition2}`;const [sql, parameters] = query.toQuery();
// sql: "SELECT * FROM users WHERE field1=$1 AND user_id=$2 AND number=$3"
// parameters1: [field1Val, userId, number]// execute the query
const result = await client.query(...query.toQuery());
```

# 对其他数据库的支持

`SQLSyntax`对象的默认`toQuery`方法将生成针对 postgreSQL 的 SQL 文本字符串。如果它对您不起作用，您可以用自己的实现替换该逻辑:

```
import SQLSyntax from "sql-syntax";SQLSyntax.customToQueryFunc = (s:SQLSyntax) => {
    //you own implementation...
}
```

# 尝试一下

为了进行试验，您可以通过以下方式使用 yarn 或 npm 安装该库:

```
// if you use npm 
npm install sql-syntax// if you use yarn
yarn add sql-syntax
```

也可以访问其 github repo:[https://github.com/t83714/SQLSyntax](https://github.com/t83714/SQLSyntax)或 [API 文档](https://t83714.github.io/SQLSyntax/classes/SQLSyntax.html)了解更多信息。