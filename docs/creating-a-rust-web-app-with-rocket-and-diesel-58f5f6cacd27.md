# 用 Rocket 和 Diesel 创建 Rust Web 应用程序

> 原文：<https://itnext.io/creating-a-rust-web-app-with-rocket-and-diesel-58f5f6cacd27?source=collection_archive---------0----------------------->

![](img/d8e8333499e538957d6b20e567cdd4bd.png)

安迪·赫尔曼万在 [Unsplash](https://unsplash.com/s/photos/rocket?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

## 我玩得很开心

*变更日志:【2022 年 12 月 30 日—使用 Medium 的新代码块进行语法突出显示*

D独立的、分离的、简洁的、包含电池的——在对火箭进行修补后，我会将这些形容词与它联系在一起。我会说我真的玩得很开心。请点击此处快速查看。

这是另一篇关于 Rust + Rocket + Diesel 技术栈的文章，比如这里的和这里的。本文像对待任何其他 web 框架一样对待这个框架——从 hello world 开始，然后是**映射**，然后是**序列化**，然后是**应用配置**，最后是**数据库配置和访问**。

因为我们是原型制作，所以除非另有说明，否则将所有内容都放在`main.rs`中。

这是我们将要使用的堆栈:

*   🦀锈
*   🚀火箭
*   ⛽柴油公司
*   🐘Postgres
*   🐳码头工人

开发工具:

*   visual Studio Code+[rust-analyzer](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer)插件
*   ✉️邮递员
*   ⿳桌游

# 内容

1.  [你好世界](#aa4a)
2.  `[GET /blog-posts/*](#f510)` [端点](#f510)
3.  `[POST /blog-posts](#9027)` [终点](#9027)
4.  [错误](#9415)
5.  [App 配置](#856a)
6.  [柴油设置](#0066)
7.  [迁移](#4501)
8.  [模式&模式](#b4d9)
9.  [柴油连接](#ae71)
10.  [更新](#5bf5) `[GET /blog-posts](#5bf5)` [端点](#5bf5)
11.  [更新](#aa0c) `[POST /blog-posts](#aa0c)` [端点](#aa0c)

## 0.锈

使用`[rustup](https://www.rust-lang.org/tools/install)`安装最新版本的 Rust 并运行

```
**rustup** default stable && **rustup** update
```

然后安装[货表](https://crates.io/crates/cargo-watch)进行热重装；

```
**cargo** install cargo-watch
```

## 1.你好世界

我们从简单的开始。创建一个名为`rocket-diesel`的新项目:

```
**cargo** new rocket-diesel --bin
**cd** rocket-diesel
```

添加火箭依赖项:

```
**cargo** add rocket@0.5.0-rc.2 --features=json
```

在`main.rs`中增加以下内容:

```
#[macro_use] extern crate rocket;

#[get("/")]
fn index() -> &'static str {
    "Hello, world!"
}

#[launch]
fn rocket() -> _ {
    let rocket = rocket::build();
    rocket.mount("/", routes![index])
}
```

1.  建造一枚火箭— `rocket::build()`
2.  **在火箭上安装**路线— `rocket.mount()`
3.  **发射**——`#[launch]`

奔跑

```
**cargo** watch -x run
```

![](img/9c07f42e232c4d4566af3c628d555f26.png)

太可爱了。

## 2.获取/博客帖子/*端点

使用 [serde](https://serde.rs/) 板条箱对 dto 进行序列化和去序列化。

```
**cargo** add serde@1.0.140
```

创建一个可以序列化和反序列化的结构。

```
#[derive(Serialize, Deserialize)]
struct BlogPost {
    id: i32,
    title: String,
    body: String,
    published: bool,
}
```

让我们创建一个`**GET /blog-posts/random**`端点。

Rocket 对 JSON 有一流的支持(还记得我们加的`--features=json`？)用一个`Json<T>`返回值创建端点。

```
#[get("/random")]
fn get_random_blog_post() -> Json<BlogPost> {
    Json(
        BlogPost {
            id: 1,
            title: "My first post".to_string(),
            body: "This is my first post".to_string(),
            published: true,
        }
    )
}
```

并将它安装到路线的`/blog-posts`矢量(第 7 行)

```
#[launch]
fn rocket() -> _ {
    let rocket= rocket::build();

    rocket
      .mount("/", routes![index])
      .mount("/blog-posts", routes![get_random_blog_post])
}
```

如果你去[http://127 . 0 . 0 . 1:8000/blog-posts/random](http://127.0.0.1:8000/blog-posts/random)，你会看到 JSON 的响应。

现在我们要创建`**GET /blog-posts/<id>**`。

```
#[get("/<id>")]
fn get_blog_post(id: i32) -> Json<BlogPost> {
    Json(
      BlogPost {
          id,
          title: "Some title".to_string(),
          body: "Some body".to_string(),
          published: true,
      }
    )
}
```

将`get_post`添加到路线矢量中。然后在[http://127 . 0 . 0 . 1:8000/blog-posts/1000](http://127.0.0.1:8000/blog-posts/1000)查看响应。

最后，我们想创建`**GET /blog-posts**`来检索所有书籍。你知道该怎么做。将响应类型更改为`Json<Vec<Book>>`。

```
#[get("/")]
fn get_all_blog_posts() -> Json<Vec<BlogPost>> {
    Json(vec![
        BlogPost {
            id: 0,
            title: "Harry Potter".to_string(),
            body: "There once lived a boy".to_string(),
            published: true,
        },
        BlogPost {
            id: 1,
            title: "Fantastic Beast".to_string(),
            body: "There once lived a beast".to_string(),
            published: true,
        }
      ]
    )
}
```

将`get_all_posts`添加到路线矢量中。然后在[http://127 . 0 . 0 . 1:8000/blog-posts](http://127.0.0.1:8000/blog-posts)查看响应。

## 3.帖子/博客-帖子端点

用`application/json`数据过帐映射？在宏中使用`data =` ！别忘了`Json`包装！

```
#[post("/", data = "<blog_post>")]
fn create_blog_post(blog_post: Json<BlogPost>) -> Json<BlogPost> {
    blog_post
}
```

在 HTTP 客户端中尝试一下。

## 4.错误

如果在调用上述 POST 端点时数据类型错误，将自动返回 HTTP 422:

```
{
    "error": {
        "code": 422,
        "reason": "Unprocessable Entity",
        "description": "The request was well-formed but was unable to be followed due to semantic errors."
    }
}
```

## 5.应用程序配置

应用程序配置是 [12 因素应用程序](https://12factor.net/)的一部分，我们很好奇它如何与 Rocket 一起工作。提示:依赖注入😍。

配置在`Rocket.toml`中定义。创建此文件并添加以下内容:

```
# ./Rocket.toml

[default]
name = "raimi"
age = 20
```

在`src/main.rs`中将该配置定义为一个 Rust 结构。这里，我们将该结构命名为`Config`。

```
#[derive(Deserialize)]
struct Config {
    name: String,
    age: u8,
}
```

在 Rocket 中，配置是[管理的状态](https://rocket.rs/v0.5-rc/guide/state/)，可以通过`&State<T>`检索。这里，我们试图通过使用`&State<Config>`在 GET 端点中使用`Config`值。美丽的❤️.

```
// ./src/main.rs

use rocket::State;

#[get("/config")]
fn get_config(config: &State<Config>) -> String {
    format!(
      "Hello, {}! You are {} years old.", config.name, config.age
   )
}
```

最后，在第 8 行，我们告诉 Rocket，它必须从`Rocket.toml`(或另一个提供者)提取`Config`配置值，并将其用作托管状态。在第 9 行中，我们包括了新的`get_config`路线。

```
use rocket::fairing::AdHoc;

#[launch]
fn rocket() -> _ {
    let rocket = rocket::build();

    rocket
        .attach(AdHoc::config::<Config>())
        .mount("/", routes![index, get_config])
        .mount(
            "/blog-posts",
            routes![
                get_random_blog_post,
                get_blog_post,
                get_all_blog_posts,
                create_blog_post
            ],
        )
}
```

## 🐳Docker 设置

在我们继续之前，让我们稍微休息一下。我们将使用 Docker Hub 的官方 Postgres 映像来设置 Postgres。创建一个`docker-compose.yml`文件:

```
version: '3.1'
services:
  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: example
    ports:
      - "5432:5432"
```

并运行`docker-compose up`。请注意，连接字符串如下所示(我们稍后将使用此信息):

```
postgresql://postgres:example/localhost:5432/postgres
```

## 6.柴油设置

我们将使用 Diesel 进行同步、阻塞式 ORM。让我们安装它，以便我们可以在 CLI 中使用它:

```
**cargo** add diesel_cli@1.4.1 --no-default-features --features postgres
```

## 7.移民

默认情况下，Diesel 在`migrations`文件夹中查找迁移。我们会创造它

```
**mkdir** migrations
```

并使用以下命令生成新的迁移脚本:

```
**diesel** migration generate create_blog_posts
```

在`up.sql`文件中，添加

```
-- Your SQL goes here
CREATE TABLE blog_posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR NOT NULL,
  body TEXT NOT NULL,
  published BOOLEAN NOT NULL DEFAULT 'f'
)
```

而在`down.sql`中，

```
-- This file should undo anything in `up.sql`
DROP TABLE blog_posts;
```

然后运行迁移，指定数据库 URL。

```
**diesel** migration run \
  --database-url postgresql://postgres:example@localhost:5432/postgres
```

## 8.模式和模型

将 Diesel 作为依赖项添加到应用程序中:

```
**cargo** add diesel@1.4.1
```

然后添加以下内容，替换现有的`BlogPost`结构:

```
#[macro_use] extern crate diesel;
use diesel::{Insertable, Queryable, table};

table! {
    blog_posts (id) {
        id -> Int4,
        title -> Varchar,
        body -> Text,
        published -> Bool,
    }
}

#[derive(Serialize, Deserialize, Queryable, Debug, Insertable)]
#[table_name = "blog_posts"]
struct BlogPost {
    id: i32,
    title: String,
    body: String,
    published: bool,
}
```

注意`table!`宏是从以下命令中派生出来的:

```
**diesel** print-schema
```

这个宏根据数据库模式扩展成一堆代码来表示`blog_post`表及其列。该代码用于将`BlogPost`结构与`blog_posts`表相关联，并根据列`id`、`title`、`body`和`published`验证结构成员的名称。太棒了。

## 9.柴油连接

添加为同步 ORM 提供支持的 Rocket 库。

```
**cargo** add rocket_sync_db_pools@0.1.0-rc.2 \
  --features=diesel_postgres_pool
```

在`Rocket.toml`中配置我们的数据库:

```
[default.databases.my_db]
url = "postgres://postgres:example@localhost:5432/postgres"
```

在我们的应用程序中使用这种配置:

```
#[database("my_db")]
pub struct Db(diesel::PgConnection);
```

然后将`Db`作为整流罩连接到第 6 行的火箭上:

```
#[launch]
fn rocket() -> _ {
    let rocket = rocket::build();

    rocket
        .attach(Db::fairing())
        .attach(AdHoc::config::<Config>())
        .mount("/", routes![index, custom])
        .mount(
            "/blog-posts",
            routes![
                get_random_blog_post,
                get_blog_post,
                get_all_blog_posts,
                create_blog_post
            ],
        )
}
```

点击保存按钮，并确保您的应用程序可以运行。

## 10.创建一篇博客文章

现在，我们将更新我们的`create_blog_post`，这样任何新的请求都将保存在数据库中:

```
#[post("/", data = "<blog_post>")]
async fn create_blog_post(
  connection: Db,
  blog_post: Json<BlogPost>,
) -> Json<BlogPost> {

    connection
        .run(move |c| {
            diesel::insert_into(blog_posts::table)
                .values(&blog_post.into_inner())
                .get_result(c)
        })
        .await
        .map(Json)
        .expect("boo")

}
```

使用 HTTP 客户端并提交 POST 请求，然后查看数据是否存储在数据库中！我用 TablePlus 做这个。

## 11.获取所有博客文章

最后，我们将更新`get_all_blog_posts`来检索我创建的所有博客文章:

```
#[get("/")]
async fn get_all_blog_posts(connection: Db) -> Json<Vec<BlogPost>> {
    connection
        .run(|c| blog_posts::table.load(c))
        .await
        .map(Json)
        .expect("Failed to fetch blog posts")
}
```

检查您是否在[http://localhost:8000/blog-posts](http://localhost:8000/blog-posts)看到了所有的博客帖子。

还和我在一起吗？太棒了，谢谢你读到这里！你可以在这里找到完整的应用:[https://github.com/remykarem/rocket-diesel](https://github.com/remykarem/rocket-diesel)。一定要看看我在下面贴的一些链接！

我觉得火箭怎么样？到目前为止，我喜欢火箭。感觉像是 Spring Boot + FastAPI(但可能是因为我只知道这两个框架)。看起来 Rocket 附带了很多电池(例如 Json)，作为该库的用户，我很欣赏这一点。我觉得[托管状态](https://rocket.rs/v0.5-rc/guide/state/#managed-state)和[请求-本地状态](https://rocket.rs/v0.5-rc/guide/state/#request-local-state)的概念类似于在 Spring Boot 管理 beans，但我不能确定，当我再次谈到它时，我会了解更多。

我觉得柴油怎么样？作为一个查询构建者，我认为 API 太冗长了。我简要地看了一下 SQLx 是如何工作的，我想下次我会尝试这样做。

直到那时！

**参考文献**

[https://rocket.rs/](https://rocket.rs/)

[https://diesel.rs/guides/getting-started](https://diesel.rs/guides/getting-started)

[https://github . com/TmLev/crud-rest-API-rust-rocket-diesel-postgres](https://github.com/TmLev/crud-rest-api-rust-rocket-diesel-postgres)

【https://dev.to/davidedelpapa/rocket-tutorial-01-basics-4ph9 

[https://github . com/SergioBenitez/Rocket/blob/v 0.5-RC/examples/databases/src/diesel _ SQLite . RS](https://github.com/SergioBenitez/Rocket/blob/v0.5-rc/examples/databases/src/diesel_sqlite.rs)

我在人工智能、机器学习、编程语言、Web 框架、生产力和学习方面发表文章。

*如果你喜欢阅读更多关于 web 框架的内容，你可以* [*订阅*](https://remykarem.medium.com/subscribe) *随时接收我发布的更新，或者* [*通过我的推荐链接注册*](https://remykarem.medium.com/membership) *！请注意，您的会员费的一部分将作为介绍费分摊给我。*