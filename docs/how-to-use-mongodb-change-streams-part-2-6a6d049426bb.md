# 如何使用 MongoDB 变更流[第 2 部分]

> 原文：<https://itnext.io/how-to-use-mongodb-change-streams-part-2-6a6d049426bb?source=collection_archive---------2----------------------->

这是一个博客系列的第二部分，涵盖了 MongoDB 变更流以及它如何与`Azure Cosmos DB`一起使用，其中[具有对 MongoDB 服务器版本 3.6](https://docs.microsoft.com/azure/cosmos-db/mongodb-feature-support-36?WT.mc_id=medium-blog-abhishgu) 的有线协议支持(包括[变更流](https://docs.mongodb.com/manual/changeStreams/)特性)。第 1 部分介绍了 Change streams processor 服务的简介和概述，并向您介绍了如何运行该应用程序，以便您可以看到工作中的变更流。

[](/how-to-use-mongodb-change-streams-part-1-da9a5e94ba2) [## 如何使用 MongoDB 变更流[第 1 部分]

### 这篇博文演示了如何在 MongoDB 中使用官方 Go 驱动程序的变更流。我将使用 Azure…

itnext.io](/how-to-use-mongodb-change-streams-part-1-da9a5e94ba2) 

在这一部分中，我们将检查代码，看看事情是如何在幕后工作的。

> *一如既往，代码为* [*可在 GitHub*](https://github.com/abhirockzz/mongodb-changestreams-processor) 上获得

# 概述:更改处理器应用程序

在继续之前，先快速回顾一下 Change Processor 服务(从第 1 部分复制过来):

该应用程序是使用变更流功能的变更处理器服务。这是一个使用官方 MongoDB Go 驱动程序[的](https://godoc.org/go.mongodb.org/mongo-driver/mongo)`[Go](https://golang.org/)`应用程序，但是这些概念应该适用于任何其本地驱动程序支持变更流的其他语言。

它使用`[Watch](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Collection.Watch)` [API](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Collection.Watch) 来订阅特定`Collection`中的变更事件提要，以便通知它正在创建、更新和删除的文档。它从变更事件有效负载(即受影响的文档)中提取相关信息，并将其保存到本地文件中。它还演示了如何使用`Resume Tokens`来保存处理进度。

# 代码走查

![](img/700e1ed8515eaa02fd689013ebf9670c.png)

下面是项目布局:`main.go`文件包含了订阅和处理变更流的大部分逻辑，而`token/resume_token.go`处理保存/检索恢复令牌。

```
.
├── go.mod
├── go.sum
├── main.go
└── token
    └── resume_token.go
```

应用程序首先连接到 Azure Cosmos DB MongoDB API，如果由于某种原因失败，就会退出。如果连接成功，我们将获得一个想要观察的 MongoDB 集合的句柄

```
client, err := mongo.NewClient(options.Client().ApplyURI(mongoURI))
    if err != nil {
        log.Fatal("failed to create client: ", err)
    } ctx, cancel := context.WithCancel(context.Background()) err = client.Connect(ctx)
    if err != nil {
        log.Fatal("failed to connect", err)
    }

  coll := client.Database(mongoDBName).Collection(mongoCollectionName)
    defer func() {
        err = client.Disconnect(context.Background())
        if err != nil {
            fmt.Println("failed to close connection")
        }
    }()
```

是时候开始“跟踪”变更事件的集合了。注意`pipeline`和`opts`参数

```
cs, err := coll.**Watch**(ctx, pipeline, opts)
```

`pipeline`是一只`[*mongo.Pipeline](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Pipeline)`。我们指定了几个阶段作为`Pipeline` - `$match`和`$project`以及`fullDocument`选项的一部分

> *在撰写本文时，* [*由于 Azure Cosmos DB 特有的约束，这些选项是强制性的*](https://docs.microsoft.com/azure/cosmos-db/mongodb-change-streams?WT.mc_id=medium-blog-abhishgu#current-limitations) *。*

这些在代码中是如何体现的:

```
matchStage := bson.D{{"$match", bson.D{{"operationType", bson.D{{"$in", bson.A{"insert", "update", "replace"}}}}}}}
    //matchStage := bson.D{{"$match", bson.D{{"operationType", "insert"}}}}projectStage := bson.D{{"$project", bson.M{"_id": 1, "fullDocument": 1, "ns": 1, "documentKey": 1}}}
    pipeline := mongo.Pipeline{matchStage, projectStage}
    opts := options.ChangeStream().SetFullDocument(options.UpdateLookup)
```

您可以选择使用 resume 令牌(由一个环境变量指定——下一节将详细介绍)。使用`ResumeAfter`字段将恢复令牌设置为`[*options.ChangeStreamOptions](https://godoc.org/go.mongodb.org/mongo-driver/mongo/options#ChangeStreamOptions)`。恢复令牌(如果存在的话)被本地存储在一个名为`token`的文件中——我们在开始我们的变更流之前检查这个令牌，如果它存在就使用它

```
if resumeSupported {
        t, err := token.RetrieveToken()
        if err != nil {
            log.Fatal("failed to fetch resume token: ", err)
        }
        if t != nil {
            opts.SetResumeAfter(t)
        }
    }
```

如前所述，令牌操作(保存和检索)是`token/resume_token.go`的一部分。以下是获取现有令牌的方式:

```
func RetrieveToken() (bson.Raw, error) {
    tf, err := os.Open(tokenFileName)
    if err != nil {
        if os.IsNotExist(err) {
            return nil, nil
        }
        return nil, err
    }
    token, err := bson.NewFromIOReader(tf)
    if err != nil {
        return nil, err
    }
    return token, nil
}
```

我们创建输出文件来存储变更事件，并开始观察集合

```
op, err := os.OpenFile(outputfileName,
        os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
  ...cs, err := coll.Watch(ctx, pipeline, opts)
if err != nil {
  log.Fatal("failed to start change stream watch: ", err)
}
```

不同的 goroutine 用于处理变更事件

```
go func() {
        fmt.Println("started change stream...")
        for cs.Next(ctx) {
            re := cs.Current.Index(1)
            _, err := op.WriteString(re.Value().String() + "\n")
            if err != nil {
                fmt.Println("failed to save change event", err)
            }
        }
    }()
```

goroutine 是这个(相当简单的)处理服务的核心，它作为一个紧密的`for`循环运行，该循环使用`[Next](https://godoc.org/go.mongodb.org/mongo-driver/mongo#ChangeStream.Next)`来获取变更事件。由于`Next`是一个阻塞调用，我们需要考虑干净/优雅的退出机制。传递给`Next`的`ctx`参数扮演了一个重要的角色——它是一个`[cancellable context](https://godoc.org/context#WithCancel)`(稍后将详细介绍如何使用它)

```
ctx, cancel := context.WithCancel(context.Background())
```

一个`exit`通道用于检测程序终止(例如`ctrl+c`)

```
exit := make(chan os.Signal)
signal.Notify(exit, syscall.SIGINT, syscall.SIGTERM)
```

当变更流检测和处理在单独的 goroutine 中运行时，使用`exit`通道阻塞`main` goroutine

```
<-exit
fmt.Println("exit signalled. cancelling context")
cancel()
if resumeSupported {
   token.SaveToken(cs.ResumeToken())
}
```

它做了几件事:

调用`cancel()`，它只不过是作为创建`cancellable`上下文的副产品返回的`[Cancel function](https://godoc.org/context#CancelFunc)`(传递到`Next`)。调用`CancelFunc`将程序终止传播到循环的更改流，允许它退出。

> *请注意，我使用了用户发起的(手动)程序终止(如按 ctrl+c)作为例子。如果处理器服务作为长期运行的服务器组件执行，则同样的概念适用*

另一个关键点是将恢复令牌保存到本地文件:

```
if resumeSupported {
  token.SaveToken(cs.ResumeToken())
}
```

以下是将恢复令牌保存到本地文件的方式:

```
func SaveToken(token []byte) { if len(token) == 0 {
        return
    }
    tf, err := os.Create(tokenFileName) if err != nil {
        fmt.Println("token file creation failed", err)
        return
    }
    _, err = tf.Write(token)
    if err != nil {
        fmt.Println("failed to save token", err)
        return
    }
    fmt.Println("saved token to file")
}
```

关于 MongoDB 变更流的两部分系列到此结束。我希望这是有用的，并有助于理解一个实际例子的帮助功能！