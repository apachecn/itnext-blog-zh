# 创建自己的亚马逊红移数据库

> 原文：<https://itnext.io/create-upsert-yourself-for-amazon-redshift-databases-a46ad8cce120?source=collection_archive---------3----------------------->

## 有了单独的临时表，在 PostgreSQL 中插入和更新数据就变得简单了。

![](img/45f82fcdb526e199754bceed707523aa.png)

# 介绍

我喜欢在创业公司工作的一个原因是，我可以和公司里各种各样的人一起工作，还可以解决有趣的问题。几个月前，营销团队致力于改进我们的分析报告，并希望将数据存储在我们的 Amazon Redshift 数据库中，以便用于构建更详细的报告。

虽然如果所有数据都是全新的，这通常是一个相对简单的 SQL 请求，但有一点让它变得有点棘手，那就是我需要更新数据表中的某些行并插入新行。通常，我会使用一条`UPSERT` SQL 语句来同时做这两件事，但是 Redshift 不支持`UPSERT`语句，所以我不得不更有创造性地手动实现这一点。通过一些研究和大量测试，我了解到通过使用一个临时数据表和几个 SQL 语句串在一起是可能的，并且没有我想象的那么复杂。

**让我们看看如何在基于 PostgreSQL 的 Amazon Redshift 数据库中创建我们自己版本的** `**UPSERT**` **SQL 语句，更新已经存在的数据并添加新数据。**

# 亚马逊红移

[Redshift](https://docs.aws.amazon.com/redshift/latest/dg/c_redshift-sql.html) 是亚马逊网络服务基于 PostgreSQL 的云数据仓库，增加了管理超大型数据集和支持这些数据的高性能分析和报告的功能。

> *它类似于谷歌的 BigQuery 或微软的 Azure 云数据仓库，如果你过去使用过其中任何一个的话。*

尽管 Redshift 是基于 PostgreSQL 的，但它也有一些不同之处，其中之一就是缺少 T4 功能。

## 没有内置的升级功能

亚马逊很乐意承认这个缺点，并建议使用一个[暂存表来执行合并](https://docs.aws.amazon.com/redshift/latest/dg/c_best-practices-upsert.html)。这听起来很有道理，但是关于如何做到这一点的 AWS 文档有点少。

所以在我弄明白之后，我想分享一下，每一步都有代码示例。我们开始吧。

# 创建您自己的 UPSERT

> ***注意:*** *本教程假设您熟悉基本的 SQL 语法。如果您希望参考本文中的任何命令，我建议您查看* [*红移 SQL 命令文档*](https://docs.aws.amazon.com/redshift/latest/dg/c_SQL_commands.html) *。*

如果没有本地的，我们就自己做。有多种方法可以重新创建`UPSERT`,但是我选择的方法实际上是删除生产表中与临时表中的行匹配的所有现有行，然后将临时表中的所有数据插入生产表中。

这听起来很复杂(因为它有点复杂)，但是一旦过程中的每一步都被分解了，就没那么糟糕了。

对于本文，我将建立一个表(USER_UPDATES ),它是一个虚构的 USERS 表的副本。在 USER_UPDATES 中，我将修改表中的一些现有数据(更新)，并在表中创建一些新数据(插入)，然后我们将把新修改的数据更新到 USERS 表中。

**前提数据修改设置**

```
-- Create a sample table as a copy of the fake USERS table 

CREATE TABLE user_updates AS
SELECT * FROM users;

-- Change every third row so we have updated users

UPDATE user_updates
SET 
    firstname = upper(firstname),
    lastname = upper(lastname),
    lastseen = '2022-11-13'
AND mod(userid, 3) = 0;;

-- Add some new rows of users so we have insert examples 
-- This example creates a duplicate of every fifth row of user data

INSERT INTO user_updates
SELECT 
      (userid + 127) AS userid, 
      firstname, 
      lastname, 
      getdate() AS lastseen,
FROM user_updates
AND mod(userid, 5) = 0;
```

现在已经有了一些更新的用户数据，是时候将它添加回 USERS 表中了，同时确保已经在表中的用户数据被覆盖和更新，并添加新的用户数据。

**1。制作暂存表**

我们需要做的第一件事是编写一个 SQL 命令来创建一个临时表，它是数据最终将被写入的生产表的精确副本。

```
-- Create temporary table that's a duplicate of the production users table

CREATE TEMP TABLE temp_users (LIKE users);
```

创建一个临时表与在 SQL 中创建一个常规表几乎是一样的(只需添加`TEMP`)，我们不必详细列出每一列，只需使用`LIKE <table_name>`告诉 SQL 将这个表的列作为它最终要添加数据的表的副本。

**2。将新数据插入临时表**

现在已经创建了临时表，可以将先决条件设置步骤中修改过的数据插入到临时表中了。在这个 INSERT 语句中，我们将从 USER_UPDATES 表中获取所有数据，并将其插入临时的 TEMP_USERS 表中。

```
-- Take all the data from the user_updates table and insert it into the temporary table

INSERT INTO temp_users
SELECT
       userid,
       firstname,
       lastname,
       lastseen,
FROM user_updates;
```

这里，来自 USER_UPDATES 的所有数据暂时放在 TEMP_USERS 表中。

**3。开始交易**

一个**事务**是在数据库中完成的一个工作单元——工作可以是从创建表到删除表的任何事情。这里发生的事务将从生产表用户中删除特定的用户数据行(这些用户是在临时表中更新了数据的现有用户)，然后将他们的新数据添加到表中。

```
-- Start the work of deleting existing user data from the table and adding the new data instead

BEGIN TRANSACTION;
```

关于 PostgreSQL 中的事务，只要执行成功，就会在语句结束时隐式执行提交。如果执行失败，则执行回滚。这变得特别方便，因为如果新用户数据未能添加到 USERS 表中，它将回滚并自动将旧用户数据替换回表中。

**4。删除永久表中需要用新数据替换的行**

一旦事务开始，要做的第一件事就是从永久表(USERS)中删除所有用户 ID 与 TEMP_USERS 表中的用户 ID 相匹配的行。

```
-- Delete the users from the database whose data needs to be updated

DELETE FROM users 
USING 
    temp_users 
WHERE users.userid = temp_users.userid;
```

首先，通过这样做`DELETE`,我们确保当新数据被添加到 USERS 表中时，不会有重复的用户数据。

**5。将临时表中的所有数据插入永久表**

现在可以安全地将 TEMP_USERS 表中的所有内容插入到 USERS 表中，而不用担心复制现有的用户数据。

```
-- Add all the user data from the temp table to the prod table 

INSERT INTO users
SELECT * FROM temp_users;
```

**6。结束交易**

一旦事务的所有部分都成功完成，它就可以像开始一样结束。

```
-- End the work

END TRANSACTION;
```

现在还有最后一件事要做:稍微清理一下。

**7。删除临时表**

最后，因为 TEMP_USERS 表已经完成了它的工作，不再需要它，所以它被丢弃(删除)。

```
-- Delete the temporary table now that its job is done

DROP TABLE temp_users;
```

一旦每个步骤都被分解并解释清楚，创造我们自己的`UPSERT`也不是那么糟糕。

# 结论

Amazon Redshift 是一个流行、强大的数据仓库，但即使它基于 PostgreSQL，它也缺少一些更好的功能，如`UPSERT`，这正是我需要帮助一个团队工作的功能。

因为我需要用新信息更新数据表，并更新该表中的现有数据，所以我需要比简单的 SQL `INSERT`更多的东西。幸运的是，借助一个临时表和几个有针对性的`DELETE`和`INSERT`查询，创建`UPSERT`功能并没有我想象的那么困难。不算太寒酸。

过几周再来看看——我会写更多关于 JavaScript、React、IoT 或其他与 web 开发相关的东西。

如果你想确保你不会错过我写的一篇文章，在这里注册我的时事通讯:[https://paigeniedringhaus.substack.com](https://paigeniedringhaus.substack.com/)

感谢阅读。我希望看到如何创建自己的 SQL `UPSERT`函数对您有所帮助。这可能不是第一次也不是最后一次需要向表中插入和更新数据，最好有一些如何最好地完成它的选项。

# 参考资料和更多资源

*   [AWS 红移文档，更新并插入新数据](https://docs.aws.amazon.com/redshift/latest/dg/t_updating-inserting-using-staging-tables-.html)
*   [AWS 红移文档、亚马逊红移和 PostgreSQL](https://docs.aws.amazon.com/redshift/latest/dg/c_redshift-and-postgres-sql.html)
*   [PostgreSQL](https://www.postgresql.org/)

*原载于*[*https://www.paigeniedringhaus.com*](https://www.paigeniedringhaus.com/blog/create-upsert-yourself-for-amazon-redshift-databases)*。*