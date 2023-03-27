# Node.js 中的流 SQL

> 原文：<https://itnext.io/streaming-sql-in-node-js-eb419c5bd27e?source=collection_archive---------3----------------------->

![](img/bf1e52b9211f5d67a97e3f42122746f8.png)

亨德里克·科内利森在 [Unsplash](https://unsplash.com/s/photos/stream?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄的照片

有时，您需要一种方法来查看数据库表中的每一行，并对其执行一些操作。

总的来说，SQL 数据库非常擅长处理大量数据。我不认为可以夸张地说，我见过的大多数使用 Hadoop 的公司，只要给他们的 Postgres/MySQL 服务器添加一个索引或一些 RAM 就会更好。您可以在 SQL 中进行惊人的过滤和聚合。有时这些强大的操作不支持您需要的东西，有时您会希望处理 node.js 中的每一行

# 异步迭代

想象以下场景:

*   我们有一个数据库，其中包含大量来自 Twitter 的推文 id。至少几十万。
*   我们想请求 Twitter 上每条推文的点赞数
*   我们将此作为后台作业进行处理，不想占用太多内存。

我们将希望对查询结果进行流式处理，以便从 SQL 中获取行。这个例子同样适用，不管你是对 Postgres 使用`[@databases/pg](https://www.atdatabases.org/docs/pg.html)`，对 MySQL 使用`[@databases/mysql](https://www.atdatabases.org/docs/mysql.html)`，还是对 SQLite 使用`[@databases/sqlite](https://www.atdatabases.org/docs/sqlite.html)`:

```
// replace this with your db of choice
**import** connect, {sql} **from** *'@databases/pg'*;**const** db = connect();**export** **default** **async** **function** updateTweets() {
  **const** tweets = db.queryStream(
    sql`**SELECT** id, likes **FROM** tweets;`
  );
  **for** **await** (**const** tweet **of** tweets) {
    **const** likes = **await** getLikes(tweet.id);
    **if** (likes !== tweet.likes) {
      **await** db.query(
        sql`
          **UPDATE** tweets
          **SET** likes = ${likes}
          **WHERE** id = ${tweet.id};
        `
      );
    }
  }
}
```

这将需要一个非常新的 node 版本，或者像 babel 这样的 transpiler，以便支持异步迭代器提议。它将从我们的数据库中传输推文，在它开始告诉数据库减慢数据发送速度之前，允许缓冲一些推文。一旦我们收到一条推文，并完成了之前对推文的处理，我们就可以开始处理它。

# Node.js 流

假设您需要将一个非常大的数据库表导出到一个 CSV 文件中，以便在其他系统中进行处理，例如一个分析工具，或者导入到一个会计包中。您可以等到获取了所有记录，但是一旦数据可用，就开始向客户端发送数据会更有效。如果他们是通过(相对)较慢的互联网连接下载的话，这一点尤其正确。

这个例子同样适用，不管你是用`[@databases/pg](https://www.atdatabases.org/docs/pg.html)`写 Postgres，还是用`[@databases/mysql](https://www.atdatabases.org/docs/mysql.html)`写 MySQL。它目前不支持 SQLite。

```
// replace this with your db of choice
**import** connect, {sql} **from** *'@databases/pg'*;
**import** {Map} **from** *'barrage'*; **import** stringify **from** *'csv-stringify'*;**const** db = connect();**export default function** getTweetsCSV() {
  **const** tweets = db.queryNodeStream(
    sql`**SELECT** id, likes **FROM** tweets;`
  );
  **const** map = **new** Map(tweet => [tweet.id, tweet.likes]);
  **const** stringifier = stringify();
  stringifier.write([*'Tweet ID'*, *'Likes'*]);
  tweets.pipe(map).pipe(stringifier);
  tweets.on(*'error'*, e => stringifier.emit(*'error'*, e);
  map.on(*'error'*, e => stringifier.emit(*'error'*, e);
  **return** stringifier;
}
```

这里我们请求所有 tweets 作为 node.js 对象流。然后，我们将该流通过管道传输到一个`Map`流中，该流接受每一行，表示为一个对象，并以数组的形式返回该行。最后，我们将这个映射流通过管道传输到 csv-stringifier 流，该流为每一行接收一个数组，并输出一个 csv 文件。如果客户端支持 gzip，我们可以进一步将流传输到`createGzip`。

# 结论

这两种流选项都增加了很多灵活性。

*   99%的时候，你不想要流媒体。这使得您的代码更难理解，并且大多数时候您不需要将整个表拉下来进行处理。只需筛选出你感兴趣的几行，然后一气呵成地进行处理。
*   当您想要对大型表中的每一行应用一个有副作用的操作时，使用异步迭代器方法(即`.queryStream`)
*   当您想要将大型表中的每一行序列化为字符串/二进制格式时，请使用节点流方法(即`.queryNodeStream`)。