# 在新的 Rails 6 应用程序中使用 UUIDs

> 原文：<https://itnext.io/using-uuids-to-your-rails-6-application-6438f4eeafdf?source=collection_archive---------1----------------------->

## 在一个全新的 Rails 应用程序上发现 UUIDs 的威力

![](img/e2073be94b0b41defee64e02177578bd.png)

约书亚·富勒

## *在我的* [*上篇文章*](/why-working-with-uuids-instead-of-ids-is-better-b60d22caf601) *中，我解释了为什么使用 UUIDs 比顺序 id 更好。*

现在我们将创建一个新的 **Rails 6** 应用程序，用 UUIDs 作为主键，用 **PostgreSQL** 作为数据库适配器。

# 将 UUID 应用到我们的应用中

PostgreSQL 可以像数据类型一样存储许多信息。

`rails new rails_uuid -d=postgresql`

为了能够在 PostgreSQL 中使用 UUID 的强大功能，我们需要创建一个迁移。

`rails g migration enable_uuid`

PostgresSQL 现在知道如何创建 UUID 类型的表

# 将 UUID 设置为默认主键类型

在这种配置下，我们的新型号将以 UUID 作为主键类型

# 创建我们的第一个模型

跑`rails g model Post title`

正如您现在看到的，当我们创建表时，我们需要**将** `**uuid**` **指定为主键**。

我们现在运行`rails db:migrate`，从控制台创建 3 个帖子。

我们现在创建 3 个 posts 并检查主键的类型。

```
$ rails console> Post.create(title: 'first post')(0.5ms) BEGIN
 Post Create (5.2ms) INSERT INTO “posts” (“title”, “created_at”, “updated_at”) VALUES ($1, $2, $3) RETURNING “id” [[“title”, “first post”], [“created_at”, “2020–03–13 20:43:09.901543”], [“updated_at”, “2020–03–13 20:43:09.901543”]]
 (2.2ms) COMMIT
=> #<Post id: “10f2b8e9–1fa8–4332–8868–2e11fb3c7ccc”, title: “first post”, created_at: “2020–03–13 20:43:09”, updated_at: “2020–03–13 20:43:09”>> Post.create(title: 'second post')
 (0.5ms) BEGIN
 Post Create (0.7ms) INSERT INTO “posts” (“title”, “created_at”, “updated_at”) VALUES ($1, $2, $3) RETURNING “id” [[“title”, “second post”], [“created_at”, “2020–03–13 20:43:14.779885”], [“updated_at”, “2020–03–13 20:43:14.779885”]]
 (0.9ms) COMMIT
=> #<Post id: “fdd597db-97af-40df-8009-ea6fba2ee9b8”, title: “second post”, created_at: “2020–03–13 20:43:14”, updated_at: “2020–03–13 20:43:14”>> Post.create(title: 'third post')
 (0.5ms) BEGIN
 Post Create (0.7ms) INSERT INTO “posts” (“title”, “created_at”, “updated_at”) VALUES ($1, $2, $3) RETURNING “id” [[“title”, “third post”], [“created_at”, “2020–03–13 20:43:21.232768”], [“updated_at”, “2020–03–13 20:43:21.232768”]]
 (1.0ms) COMMIT
=> #<Post id: “8485ceca-3a85–41e7-b86e-6b6d93dc4e8b”, title: “third post”, created_at: “2020–03–13 20:43:21”, updated_at: “2020–03–13 20:43:21”>
```

完善我们的主键列现在是一个`uuid`类型！

# 订购我们的唱片

我们现在检查我们文章的正确顺序，第三篇文章应该是最新的。

```
$ rails console> Post.last
=> #<Post id: “fdd597db-97af-40df-8009-ea6fba2ee9b8”, title: “second post”, created_at: “2020–03–13 20:43:14”, updated_at: “2020–03–13 20:43:14”>
```

如我们所见，最后一个帖子是“第二个帖子”，而不是“第三个帖子”。这是因为**默认情况下 ActiveRecord 是用主键排序的。**现在我们使用 UUID，ActiveRecord 按照字母数字顺序排序。

为了解决这个问题，我们需要编辑*app/models/application _ record . Rb*文件，并要求 ActiveRecord 不要按**主键**排序，而是按**创建日期**和`created_at`列排序。

我们所有的型号现在都将按 created_at 列进行订购

我们将再试一次。

```
> reload!
> Post.lastPost Load (3.8ms) SELECT “posts”.* FROM “posts” ORDER BY “posts”.”created_at” DESC LIMIT $1 [[“LIMIT”, 1]]
=> #<Post id: “8485ceca-3a85–41e7-b86e-6b6d93dc4e8b”, title: “third post”, created_at: “2020–03–13 20:43:21”, updated_at: “2020–03–13 20:43:21”>
```

祝贺我们现在正确排序我们的记录。

# 与模型的关联

我们现在正在创建属于`Post`的第二个模型`Comment`。

运行`rails g model Comment post:belongs_to body:text`

您必须指定外键的类型

我们现在将验证一切正常。

```
$ rails db:migrate
$ rails console> Post.first.comments.create(body: 'bonjour')> Post.first.comments
 Post Load (2.1ms) SELECT “posts”.* FROM “posts” ORDER BY “posts”.”created_at” ASC LIMIT $1 [[“LIMIT”, 1]]
 Comment Load (11.8ms) SELECT “comments”.* FROM “comments” WHERE “comments”.”post_id” = $1 LIMIT $2 [[“post_id”, “10f2b8e9–1fa8–4332–8868–2e11fb3c7ccc”], [“LIMIT”, 11]]
=> #<ActiveRecord::Associations::CollectionProxy [#<Comment id: “a504f964–4678–4c9e-b0fb-46de58cfde5d”, post_id: “10f2b8e9–1fa8–4332–8868–2e11fb3c7ccc”, content: “jhsdbfjhfsbd”, created_at: “2020–03–13 20:54:41”, updated_at: “2020–03–13 20:54:41”>]>> Comment.last.post
 Comment Load (2.3ms) SELECT “comments”.* FROM “comments” ORDER BY “comments”.”created_at” DESC LIMIT $1 [[“LIMIT”, 1]]
 Post Load (1.4ms) SELECT “posts”.* FROM “posts” WHERE “posts”.”id” = $1 LIMIT $2 [[“id”, “10f2b8e9–1fa8–4332–8868–2e11fb3c7ccc”], [“LIMIT”, 1]]
=> #<Post id: “10f2b8e9–1fa8–4332–8868–2e11fb3c7ccc”, title: “plop”, created_at: “2020–03–13 20:43:09”, updated_at: “2020–03–13 20:43:09”>
```

## 恭喜你，一切都很完美！

源代码:[https://github.com/GuillaumeOcculy/rails_uuid](https://github.com/GuillaumeOcculy/rails_uuid)

## 相关文章:

[](/why-working-with-uuids-instead-of-ids-is-better-b60d22caf601) [## 为什么使用 UUIDs 比使用 id 更好

### 使用 UUIDs 而不是 IDs 有很多好处。

itnext.io](/why-working-with-uuids-instead-of-ids-is-better-b60d22caf601)