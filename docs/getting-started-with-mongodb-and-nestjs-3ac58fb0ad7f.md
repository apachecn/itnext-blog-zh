# MongoDB 和 NestJS 入门

> 原文：<https://itnext.io/getting-started-with-mongodb-and-nestjs-3ac58fb0ad7f?source=collection_archive---------0----------------------->

![](img/97065b45960545f27b842e50879b3c41.png)

约翰·汤纳在 [Unsplash](https://unsplash.com/s/photos/connect?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

您是否有一个 NestJS 应用程序，并希望将其连接到 MongoDB 数据库？以下是你可以采取的步骤。

## 第一步—创建数据库

要在 MongoDB 服务器中创建一个新的数据库，我们需要启动并运行数据库服务器，并且能够启动一个交互式 shell。

我的数据库是一个开发数据库，通过一个 [Docker 容器](/why-and-how-to-run-your-development-database-with-docker-2e0eab13b733?source=your_stories_page-------------------------------------)在我的本地机器上运行。

如果您有相同的配置，连接到容器并运行`mongo` 命令。

如果您已经在您的机器上直接安装了 MongoDB，那么只需在一个新的终端窗口中运行`mongo`(或者 Windows 的命令提示符)。

![](img/d7bbc0b4d9016e4f2022245f6625b342.png)

一旦连接上，我们可以运行`use new_db_name`来创建一个新的数据库。

![](img/e1692ffd9e01cdee2df9287aea085a94.png)

创建新的数据库

我们目前正在使用`my_new _db`数据库，但是我们不能创建空数据库，因为它不会被保存。通过运行以下命令可以看到这一点:

![](img/dfa91a042189b8cd6ee1886f56e0cc7c.png)

列出所有数据库和当前正在使用的数据库

`db`命令会告诉我们当前使用的是哪个数据库:`my_new_db`。但是，如果我们运行`show dbs`，我们可以看到这个数据库没有列出。那是因为在我们把东西放进去之前，它还没有完全被创造出来。

为了解决这个问题，我们将使用`db.collection_name.insert()`命令插入一个新文档。

![](img/c8dce72e4b7795777aa558e05f66335a.png)

集合名称和文档可以是随机的，因为我们可以在以后删除它们。

## 第二步—为此数据库创建一个新用户

下一步是创建一个专用用户，供我们的 NestJS 应用程序使用。这个用户将只拥有他需要的权限:**读**和**写**。

出于安全原因，我们这样做是为了减少任何不必要的问题。

执行此操作的命令如下:

```
db.createUser({ user: "my_app_user", pwd: "my_password", roles: [ { role: "readWrite", db: "my_new_db" }]})
```

其中`user`和`pwd`是我们希望用于该用户的凭证，而`readWrite`是分配给他的角色。

如果你想知道还有什么其他的角色，每个角色代表什么，那么你可以在[文档](https://docs.mongodb.com/manual/reference/built-in-roles/#database-user-roles)中查看这个页面。

![](img/cb05500d687af2a06911085e7c23d0fb.png)

创建新用户

## 第三步—连接 NestJS 应用程序

我们将使用[mongose](https://www.npmjs.com/package/mongoose)与数据库通信。因此，我们需要从安装所需的依赖项开始。

```
npm install --save @nestjs/mongoose mongoose
```

一旦安装了这些依赖项，我们就可以编写下面的代码来连接到`app.module.ts`中的数据库。

![](img/44ca2dcb01e268786aafcfe2d9ac1030.png)

[生](http://4%2C+195%2C+1%29&t=seti&wt=none&l=javascript&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=import%2520%257B%2520Module%2520%257D%2520from%2520%27%2540nestjs%252Fcommon%27%253B%250Aimport%2520%257B%2520MongooseModule%2520%257D%2520from%2520%27%2540nestjs%252Fmongoose%27%253B%250Aimport%2520%257B%2520AppController%2520%257D%2520from%2520%27.%252Fapp.controller%27%253B%250Aimport%2520%257B%2520AppService%2520%257D%2520from%2520%27.%252Fapp.service%27%253B%250A%250A%2540Module%28%257B%250A%2520%2520imports%253A%2520%255B%250A%2520%2520%2520%2520MongooseModule.forRoot%28%250A%2520%2520%2520%2520%2520%2520%27mongodb%253A%252F%252Fmy_app_user%253Amy_password%2540127.0.0.1%253A27017%252Fmy_new_db%27%252C%250A%2520%2520%2520%2520%29%252C%250A%2520%2520%255D%252C%250A%2520%2520controllers%253A%2520%255BAppController%255D%252C%250A%2520%2520providers%253A%2520%255BAppService%255D%252C%250A%257D%29%250Aexport%2520class%2520AppModule%2520%257B%257D%250A)

标准的连接字符串 URI 格式是:`mongodb://[username:password@]host1[:port1][,…hostN[:portN]][/[defaultauthdb][?options]]`

如果我们用我们的凭证和数据库替换它，我们将得到类似这样的结果:

`mongodb://my_app_user:my_password@127.0.0.1:27017/my_new_db`

## 第四步——与系列互动

> 在下面的例子中，我们将使用我们之前创建的集合和文档。您可以根据自己的具体需求替换它们。

我们将在 NestJS 应用程序中创建一个新的模块、一个新的模式和一个新的服务，以便与数据库中的虚拟集合进行交互。

![](img/c40e7e383fe9a6a2ee0761303adacf1b.png)

在`collection.schema.ts`中，我们将定义文档模式。

![](img/ce5178fbfc7565ebd7c00fd5d96d9d60.png)

[生](https://carbon.now.sh/?bg=rgba%28171%2C+184%2C+195%2C+1%29&t=seti&wt=none&l=javascript&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=import%2520%257B%2520Prop%252C%2520Schema%252C%2520SchemaFactory%2520%257D%2520from%2520%27%2540nestjs%252Fmongoose%27%253B%250Aimport%2520%257B%2520Document%2520%257D%2520from%2520%27mongoose%27%253B%250A%250Aexport%2520type%2520MyFirstCollectionDocument%2520%253D%2520MyFirstCollection%2520%2526%2520Document%253B%250A%250A%2540Schema%28%29%250Aexport%2520class%2520MyFirstCollection%2520%257B%250A%2520%2520%2540Prop%28%29%250A%2520%2520keep%253A%2520string%253B%250A%257D%250A%250Aexport%2520const%2520MyFirstCollectionSchema%2520%253D%2520SchemaFactory.createForClass%28%250A%2520%2520MyFirstCollection%252C%250A%29%253B%250A)

在`my-first-collection.module.ts`内部，我们将使用来自`MongooseModel`的`forFeature()`方法来配置模块，包括定义哪些模型应该在当前范围内注册。

![](img/af34de3675919eab8b9a6146a2c4cb21.png)

[RAW](https://carbon.now.sh/?bg=rgba%28171%2C+184%2C+195%2C+1%29&t=seti&wt=none&l=javascript&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=import%2520%257B%2520Module%2520%257D%2520from%2520%27%2540nestjs%252Fcommon%27%253B%250Aimport%2520%257B%2520MongooseModule%2520%257D%2520from%2520%27%2540nestjs%252Fmongoose%27%253B%250Aimport%2520%257B%250A%2520%2520MyFirstCollection%252C%250A%2520%2520MyFirstCollectionSchema%252C%250A%257D%2520from%2520%27.%252Fcollection.schema%27%253B%250Aimport%2520%257B%2520MyFirstCollectionService%2520%257D%2520from%2520%27.%252Fmy-first-collection.service%27%253B%250A%250A%2540Module%28%257B%250A%2520%2520imports%253A%2520%255B%250A%2520%2520%2520%2520MongooseModule.forFeature%28%255B%250A%2520%2520%2520%2520%2520%2520%257B%250A%2520%2520%2520%2520%2520%2520%2520%2520name%253A%2520MyFirstCollection.name%252C%250A%2520%2520%2520%2520%2520%2520%2520%2520schema%253A%2520MyFirstCollectionSchema%252C%250A%2520%2520%2520%2520%2520%2520%2520%2520collection%253A%2520%27my_first_collection%27%252C%250A%2520%2520%2520%2520%2520%2520%257D%252C%250A%2520%2520%2520%2520%255D%29%252C%250A%2520%2520%255D%252C%250A%2520%2520providers%253A%2520%255BMyFirstCollectionService%255D%252C%250A%2520%2520exports%253A%2520%255BMyFirstCollectionService%255D%252C%250A%257D%29%250Aexport%2520class%2520MyFirstCollectionModule%2520%257B%257D%250A)

默认情况下，NestJS 会将 schema 类映射到一个同名的 MongoDB 集合，但是在末尾有一个附加的“s”——所以在我们的例子中，最终的 mongo 集合名称将是`myfirstcollections`。这在现实生活的例子中非常有意义，因为我们有一个名为`users`或`articles`的集合和一个名为`User`或`Article`的模式。

然而，在本文中，我们在创建集合的过程中使用了一个随机名称，所以我们必须使用`collection`属性明确指定我们想要映射的集合。

最后，在`my-first-collection.service.ts`中，我们可以注入模型并利用`mongoose`包与数据库通信。

![](img/962a57a46f1860fe77b84932d232afec.png)

[原始](https://carbon.now.sh/?bg=rgba%28171%2C+184%2C+195%2C+1%29&t=seti&wt=none&l=javascript&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=import%2520%257B%2520Injectable%2520%257D%2520from%2520%27%2540nestjs%252Fcommon%27%253B%250Aimport%2520%257B%2520InjectModel%2520%257D%2520from%2520%27%2540nestjs%252Fmongoose%27%253B%250Aimport%2520%257B%2520Model%2520%257D%2520from%2520%27mongoose%27%253B%250Aimport%2520%257B%250A%2520%2520MyFirstCollection%252C%250A%2520%2520MyFirstCollectionDocument%252C%250A%257D%2520from%2520%27.%252Fcollection.schema%27%253B%250A%250A%2540Injectable%28%29%250Aexport%2520class%2520MyFirstCollectionService%2520%257B%250A%2520%2520constructor%28%250A%2520%2520%2520%2520%2540InjectModel%28MyFirstCollection.name%29%250A%2520%2520%2520%2520private%2520catModel%253A%2520Model%253CMyFirstCollectionDocument%253E%252C%250A%2520%2520%29%2520%257B%257D%250A%250A%2520%2520async%2520findAll%28%29%253A%2520Promise%253CMyFirstCollection%255B%255D%253E%2520%257B%250A%2520%2520%2520%2520return%2520this.catModel.find%28%29.exec%28%29%253B%250A%2520%2520%257D%250A%257D%250A)

## 步骤五——测试

为了快速测试，我们可以从`main.ts`中的`app`对象中提取该服务，并调用`findAll()`方法。

![](img/5b8e284ab7f444d0c279bbe3030560b5.png)

[RAW](https://carbon.now.sh/?bg=rgba%28171%2C+184%2C+195%2C+1%29&t=seti&wt=none&l=javascript&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=import%2520%257B%2520NestFactory%2520%257D%2520from%2520%27%2540nestjs%252Fcore%27%253B%250Aimport%2520%257B%2520AppModule%2520%257D%2520from%2520%27.%252Fapp.module%27%253B%250Aimport%2520%257B%2520MyFirstCollectionService%2520%257D%2520from%2520%27.%252Fmy-first-collection%252Fmy-first-collection.service%27%253B%250A%250Aasync%2520function%2520bootstrap%28%29%2520%257B%250A%2520%2520const%2520app%2520%253D%2520await%2520NestFactory.create%28AppModule%29%253B%250A%250A%2520%2520const%2520syFirstCollectionSerivce%2520%253D%2520app.get%28MyFirstCollectionService%29%253B%250A%250A%2520%2520console.log%28await%2520syFirstCollectionSerivce.findAll%28%29%29%253B%250A%250A%2520%2520await%2520app.listen%283000%29%253B%250A%257D%250Abootstrap%28%29%253B%250A)

如果我们启动应用程序，我们将看到创建数据库时最初输入的文档。

![](img/64d9b49624e3c66a1c9cf0341ba3cede.png)