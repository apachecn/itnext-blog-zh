# 在 Next.js 11 中使用 Mongoose

> 原文：<https://itnext.io/using-mongoose-with-next-js-11-b2a08ff2dd3c?source=collection_archive---------0----------------------->

在最新版本的 Next.js 框架中使用 Mongoose ORM for MongoDb 的简单指南。

## 1.创建一个 Next.js 11 项目

创建 Next.js 项目最简单的方法是使用`create-next-app`。在您的终端中运行以下命令。

```
npx create-next-app my-mongoose-app-name
```

该过程完成后，用代码编辑器打开项目目录。

## **2。添加环境变量**

我们只需要一个变量，那就是`MONGODB_URI`。在你的项目目录中创建一个名为`.env.local`的文件。我在这里使用我的本地数据库，你可以改变你自己的值。

## 3.安装猫鼬

现在让我们安装 mongoose npm 包。

```
npm i mongoose
```

## 4.添加连接文件

现在我们需要一个文件来初始化数据库连接。我们还将全局缓存连接。让我们创建一个文件— `lib/dbConnect.js`并粘贴下面的代码。

## 5.创建一个猫鼬模型

让我们创建一个 Mongoose 模型，这样我们可以将一些数据插入到我们的数据库中。你可以使用你喜欢的任何型号。下面是一个简单的用户模型的例子，它有一个名字和一个电子邮件字段。**注意最后一行**，导出与我们在 Express 中的方式不同。这可以防止 Mongoose 重新编译模型。你可以创建这个文件为`/models/User.js`。

## 6.创建 API 路线

以下示例在`/api/users.js`处创建一个端点。它包括两种方法— `GET`和`POST`。`GET`方法获取数据库中的所有用户，`POST`方法使用`req.body`中提供的参数创建一个用户。注意，我们在运行数据库查询之前做了一个`dbConnect()`。

## 7.测试您的路线！

这将是我们的最后一步。为了测试我们的端点，我们可以通过您选择的任何 HTTP 客户端发送请求。下面是 CURL 命令，您可以用它来快速验证一切是否正常。

`*POST*` *首先，用* `*GET*`获取一些数据

```
**POST - CREATE NEW USERS****EXAMPLE REQUEST** curl --request POST \
  --url [http://localhost:3000/api/users](http://localhost:3000/api/users) \
  --header 'Content-Type: application/json' \
  --data '{
 "name": "John Doe",
 "email": "[john@doe.com](mailto:john@doe.com)"
}'**EXAMPLE RESPONSE** {
  "success": true,
  "data": {
    "_id": "6112570b245e401eb895c35d",
    "name": "John Doe",
    "email": "[john@doe.com](mailto:john@doe.com)",
    "__v": 0
  }
}**GET - GET ALL USERS****EXAMPLE REQUEST** curl --request GET \
  --url [http://localhost:3000/api/users](http://localhost:3000/api/users) \
  --header 'Content-Type: application/json'**EXAMPLE RESPONSE** {
  "success": true,
  "data": [
    {
      "_id": "6112570b245e401eb895c35d",
      "name": "John Doe",
      "email": "[john@doe.com](mailto:john@doe.com)",
      "__v": 0
    }
  ]
}
```

🥳就是这样！我们都准备好了！和猫鼬玩得开心！如果你想看更多 Next.js 文章，请留下评论。我们不就是❤️ Next.js 吗？我知道我有！

**完整项目回购地点:**

[https://github.com/skolhustick/nextjs-mongoose-example](https://github.com/skolhustick/nextjs-mongoose-example)

![](img/324eb4f11dec1185302d678dabd054d0.png)

简·安东宁·科拉尔在 [Unsplash](https://unsplash.com/s/photos/database?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

相关文章:

[](https://eshwaren.medium.com/using-prisma-orm-with-mongodb-in-next-js-e42b1f7543e6) [## 在 Next.js 中使用 Prisma ORM 和 MongoDB

### 让我们创建一个简单的 CRUD 应用程序👉

eshwaren.medium.com](https://eshwaren.medium.com/using-prisma-orm-with-mongodb-in-next-js-e42b1f7543e6) 

👇👇👇 🙏

如果你觉得上面的文章有用，请考虑使用下面我的推荐链接订阅 Medium。对你来说没有额外的花费，我将每个月得到一小笔捐款。🙇‍♂️

[](https://medium.com/@eshwaren/membership) [## 通过我的推荐链接加入 Medium

### 作为一个媒体会员，你的会员费的一部分会给你阅读的作家，你可以完全接触到每一个故事…

medium.com](https://medium.com/@eshwaren/membership)