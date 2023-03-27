# Golang 与清洁建筑

> 原文：<https://itnext.io/golang-and-clean-architecture-19ae9aae5683?source=collection_archive---------0----------------------->

![](img/c4b0b56a427de5ccf15f7f8e9bcb4a78.png)

不仅是 Dame 时间，还有 Go 时间

我想在 NBA 季后赛早期发表这篇文章，当时[达米恩·利拉德](https://thedailymixreport.files.wordpress.com/2020/08/damianlillard_dametime1-1.jpg?w=950)正在打球。虽然现在有点晚了，但他仍然值得大声呼喊。

## 在本帖中，我们将回顾并部署一个遵循干净架构原则的 golang 应用程序。
源代码可以在[这里](https://github.com/subzero112233/aws-golang-lambda)找到。

*   更新:3.7.2021:删除了 OpenAPI 客户端生成，因为它没有多大帮助(而且我还必须手动编写验证/错误)，而是创建了请求-响应模型。

# 简介:

我们中的大多数人可能都参与过大型项目，其中部分代码很难扩展、理解，有时甚至难以维护。有时混乱是如此严重，以至于没人想碰那个部分/组件/系统，在你意识到之前，你已经陷入了一个需要严重重构的系统，甚至更糟，需要完全重写。

团队的规模只是在一定程度上有关系，因为当软件的架构很差时，用新的功能来扩展系统迟早会变得很困难，而且如上所述，困难会非常严重，甚至维护都变得非常繁重。

幸运的是，随着项目的增长，架构良好的软件仍然易于更改和理解，并使开发人员专注于支持业务。

Bob 叔叔的“干净的架构”一书提供了指导方针和深入的解释，并举例说明了如何确保你的项目不会随着时间的推移而腐烂。这是一本绝对必读的书。

**遵循清洁架构原则的项目将是:**

1. **UI 无关:**改变 UI 不应该影响项目的其他部分。
2。**独立于框架:**使用什么库并不重要。框架将被用作工具，不会强制任何业务规则。
3。**数据库无关:**项目不应该关心或者依赖于所选择的数据库。此外，我们希望尽可能晚地选择数据库，以保持灵活性
4。**可测试的:**可测试的系统是您有信心与之一起工作的系统——因此它是一个易于扩展或维护的系统。
5。**严格它的依赖规则:**源代码依赖只能指向内部，意味着一个内圈对外圈一无所知。

我们应用程序的整体架构:

![](img/270a7fded42881e528b41647600b149e.png)

使用的技术:
Golang，Gin (golang)
AWS: Lambda，带自定义授权器的 API 网关，DynamoDB。

目录结构:

```
.
├── api
│   ├── handler
│   │   ├── handler.go
│   │   ├── helpers.go
│   │   └── model.go
│   └── middleware
│       └── middleware.go
├── cloudformation
│   └── apigateway
│       ├── apig.yaml
│       └── create_apigateway.sh
├── entity
│   ├── entity.go
│   ├── errors.go
│   └── validations.go
├── go.mod
├── go.sum
├── lambdas
│   ├── authorizer
│   │   ├── deploy.sh
│   │   ├── main.go
│   │   └── template.yaml
│   └── users
│       ├── deploy.sh
│       ├── env.json
│       ├── main.go
│       └── template.yaml
├── Makefile
├── openapi.yaml
├── pkg
│   └── jwtparser
│       ├── jwtparser.go
│       └── jwtparser_test.go
├── README.md
├── repository
│   ├── dynamodb
│   │   └── dynmodb.go
│   ├── mockdatabase
│   │   └── mockdatabase.go
│   └── mysql
│       ├── init.sql
│       ├── model.go
│       └── mysql.go
└── usecase
    └── users
        ├── repository.go
        ├── service.go
        ├── usecase.go
        ├── users.go
        └── users_test.go17 directories, 33 files
```

# 干净的建筑:

干净的建筑由四个圆圈组成

![](img/7bc856a133b00839e0e634ff235627e1.png)

清洁建筑四圈

## 实体:

> 实体封装了最通用和最高级的业务规则。当外界发生变化时，他们最不可能改变。

这些实体不依赖于任何其他圆。

实体包还包含验证和自定义错误。

## 使用案例:

> 用例层包含**应用特定的**业务规则。

用例层只依赖于实体。实体的变化需要用例层的变化，但是其他层的变化不需要。

在我们的应用程序中，这些是注册、登录和问好方法。

## 接口适配器:

> 接口适配器层包含一组适配器，将数据从对用例和实体最方便的格式转换为对某些外部机构(如数据库或 Web)最方便的格式。

在 api 目录中，我们有`**handler**`和`**middleware**`包。
处理器处理 http 请求、响应和验证:

中间件:

## 框架和适配器:

> 这一层是所有细节的所在:网络是一个细节，数据库也是。

在框架和适配器层，我们有数据库、UI 等。
这是最外在的层次，必然比其他圈子变化更频繁。然而，这里的变化应该不会影响到内部圈子。

## 让我们看看这一切是如何组合在一起的:

首先，需要在 [Makefile](https://github.com/subzero112233/aws-golang-lambda/blob/master/Makefile) 中设置你的 S3 工件桶。
如果你没有水桶，那就跑:

```
aws s3api create-bucket --bucket <BUCKET_NAME> --region <REGION>
```

展开`users`和`authorizer`兰姆达斯:

```
make build && make deploy
```

然后，部署我们的 api 网关。

```
reshef ~ $ make deploy-apigWaiting for changeset to be created..
Waiting for stack create/update to complete
Successfully created/updated stack - aws-lambda-go-apigateway[https://](https://gjfwhy1kzg.execute-api.us-east-1.amazonaws.com/deploy/)acrymgwwwf[.execute-api.us-east-1.amazonaws.com/deploy/](https://gjfwhy1kzg.execute-api.us-east-1.amazonaws.com/deploy/)
```

脚本输出的最后一行包含我们将使用的 api 端点—[*https://gjwhy1kzg . execute-API . us-east-1 . amazonaws . com/deploy/*](https://gjfwhy1kzg.execute-api.us-east-1.amazonaws.com/deploy/)

# API 端点

**报名:**

```
reshef ~ $ curl --location --request POST 'https://acrymgwwwf.execute-api.us-east-1.amazonaws.com/deploy/users/signup' --header 'Content-Type: application/json' --data-raw '{"username": "batz", "password": "batz123456", "address": "mapo 4", "first_name": "batz", "last_name": "hatzav"}' {"message":"user successfully created"}
```

**签到:**

```
reshef ~ $ curl --location --request POST 'https://acrymgwwwf.execute-api.us-east-1.amazonaws.com/deploy/users/signin' --header 'Content-Type: application/json' --data-raw '{"username": "batz", "password": "batz123456"}' {"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MjQyODE0NTUsImlhdCI6MTYyNDI3Nzg1NSwidXNlcm5hbWUiOiJ5b3plciJ9.qht3rQoKymZeIBDUBhn5My4CPMexR_8ItqbsOMhYaxM"}
```

**/hello 端点:** 使用从前面的 api 调用中收到的令牌，我们向/hello 端点发送一个请求

```
reshef ~ $ curl --location --request GET 'https://acrymgwwwf.execute-api.us-east-1.amazonaws.com/deploy/hello' --header 'Content-Type: application/json' --header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MjQyODUyOTIsImlhdCI6MTYyNDI4MTY5MiwidXNlcm5hbWUiOiJiYXR6In0.L19gD2MOPBma5yBd43UIHBj6ThdCYLl9_jn3wRPHBpo' {"message":"Hello batz"}
```

## **解决清洁架构方法**

我个人喜欢 Lambda 和 DynamoDB 的组合。两者都是无服务器的，规模惊人，并为您节省大量的维护和操作时间。
DynamoDB 可以轻松地(在正确配置的情况下)处理(几乎)任意数量的并发请求，而无需使用连接池、设置 RDS 代理或任何类似的东西。

然而，在我们想象的公司中，一段时间后，我们的业务开始高速增长，产品经理要求我们添加更多的功能。
分解需求后，我们最终意识到 NoSQL 数据库不再适合这项任务，我们决定改用 SQL 数据库。

这完全没问题:我们可能会以某种方式开始做一些事情，随着业务的增长，需求会发生变化，我们无法预料和无法知道的事情会发生。

在我们的存储库目录中，我们有一个名为 **mysql:** 的包

我们唯一需要修改的文件是 **users lambda 中的 **main.go** 文件。** 那就不一样了 **:**

![](img/d863058b2e08c6ecd80c9f06fb352025.png)

main .走左边的 dynamodb。对于右边的 mysql。

微小的变化。为了部署我们的更改，我们再次运行:

```
make build
make deploy
```

我们准备好了。

PS:考虑到在现实生活场景中，我们需要将现有数据从旧数据库迁移到新数据库，对于 SQL 数据库，最好有一个数据库迁移工具来管理数据库模式。我推荐[飞行路线](https://flywaydb.org/)来完成这个任务。

现在想想从 AWS lambda 迁移到 Kubernetes 或其他平台，或者从 Gin 迁移到 Echo，等等，有多容易。通过这种方式，技术和业务的转换、改变和发展都很容易。

希望你觉得有用。