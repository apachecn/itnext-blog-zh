# Go 中的电报机器人:并发 SQLite

> 原文：<https://itnext.io/telegram-bot-in-go-concurrent-sqlite-e6176fac088e?source=collection_archive---------5----------------------->

![](img/28f85def32accb315107a48fcb55bce7.png)

[上次](https://medium.com/@detunized/telegram-bot-in-go-database-f8714381c858)我将 SQLite 添加到我的机器人中，同时将请求处理转移到一个 goroutine 中。这意味着我在我的代码库中引入了并发数据库访问。

通常一个人应该想，然后做。虽然不理想，但当使用`git`时，也可以反过来做。但重要的是不要忘记在某些时候思考。就像我差一点就做了，差一点就在我刚刚介绍的基础上堆上更多的 bug。

我记得前几天读过或看过一些关于 SQLite 的东西，他们明确表示我必须确保不要同时从多个地方访问数据库。Go 是为并发而生的，所以我确信我使用的 go-sqlite3 已经控制住了它。事实证明，并不尽然。

来自[常见问题解答](https://github.com/mattn/go-sqlite3#faq):

> 我可以在多个例程中同时使用它吗？
> 
> 是只读的。但是，不可写。参见 [#50](https://github.com/mattn/go-sqlite3/issues/50) 、 [#51](https://github.com/mattn/go-sqlite3/issues/51) 、 [#209](https://github.com/mattn/go-sqlite3/issues/209) 、 [#274](https://github.com/mattn/go-sqlite3/issues/274) 。

就这样，我掉进了一个大兔子洞。我搜索和研究，googled 和 binged(不，不是真的)，阅读了`go-sqlite3`的源代码和 SQLite 的文档(顺便说一下，这些都很好)。所有这些只是为了发现自己处于一种不知道是否可以信任`go-sqlite3`包来处理我的数据库工作的境地。尽管 FAQ 声明不支持写并发，但我发现了矛盾的说法。人们说如果我使用多个连接就没问题。但是我不会每次都打开一个新的连接。

长话短说，我发现了一个[博客帖子](https://crawshaw.io/blog/go-and-sqlite)解决了完全相同的问题，并以 [Go 包](https://github.com/crawshaw/sqlite)的形式提供了一个解决方案。

```
go get -u crawshaw.io/sqlite
```

根据这篇文章和这个包的作者 David 所说，我应该能够相信这个包能够可靠地处理并发性。现在，我不是简单地建立一个到数据库的连接，而是创建一个我想要的大小的显式池(在本例中是 16 个):

```
import "crawshaw.io/sqlite/sqlitex"

func openDB() *sqlitex.Pool {
    db, err := sqlitex.Open("./since.db", 0, 16)
    if err != nil {
        log.Panic(err)
    }

    return db
}
```

现在，每次我想访问数据库时，我都必须从池中获得一个连接，并且不要忘记在完成后将它放回原处。在连接可用之前，从池中获取连接可能会被阻塞。

```
func execSQL(db *sqlitex.Pool, sql string) {
    connection := db.Get(nil)
    defer db.Put(connection)

    err := sqlitex.Exec(connection, sql, nil)
    if err != nil {
        log.Panic(err)
    }
}
```

关于错误处理的补充说明。我目前不会故意处理错误，以免减慢自己的速度。但我也不忽视他们。像任何自尊的软件工程师一样，当我收到一个错误时，我会惊慌失措。对于某些错误，比如启动时无法打开数据库，惊慌失措是完全正常的。但是，如果其中一个消息有错误，恐慌就不是一个好的选择。这就像在我们应该返回 HTTP/404 的时候关闭服务器一样。

现在，将传入的消息存储在数据库中变成了这样:

```
func store(message *tgbotapi.Message, db *sqlitex.Pool) {
    connection := db.Get(nil)
    defer db.Put(connection)

    err := sqlitex.Exec(
        connection,
        "INSERT INTO events (user, name, date) VALUES (?, ?, ?);",
        nil,
        message.From.ID,
        message.Text,
        message.Date)

    if err != nil {
        log.Panic(err)
    }
}
```

我不喜欢的一点是，如果我想使用`sqlitex.Exec`，我会被迫使用位置 SQL 参数。如果我想使用像`"... VALUES ($user, $name, $date)"`这样的列名，我就必须使用更加冗长的 API。我自己准备陈述，然后逐步完成。像这样:

```
func store(message *tgbotapi.Message, db *sqlitex.Pool) {
    connection := db.Get(nil)
    defer db.Put(connection)

    insert := connection.Prep("INSERT INTO events (user, name, date) VALUES ($user, $name, $date);")
    insert.SetInt64("$user", int64(message.From.ID))
    insert.SetText("$name", message.Text)
    insert.SetInt64("$date", int64(message.Date))

    _, err := insert.Step()
    if err != nil {
        log.Panic(err)
    }

    // Done with this query
    // TODO: Is it really needed? What happens when this isn't called?
    err = insert.Reset()
    if err != nil {
        log.Panic(err)
    }
}
```

如果我看到位置论点成为一个问题，我会切换到这种做事方式。

虽然这次我很想让我的机器人不那么笨，但我只是在尝试防弹我的 SQLite 访问方法的同时设法切换了库。只是重构，没有特色。这是软件工程师典型的一天。好吧，那就第二天。

如果你好奇，可以在 GitHub 上找到代码[。这个版本被标记为`day-3`。](https://github.com/detunized/since-bot/tree/day-3)

*原发布于*[*detunized.net*](https://detunized.net/posts/2019-04-01-telegram-bot-in-go-concurrent-sqlite/)*2019 年 4 月 1 日*