# Apache Spark 和 Hadoop HDFS: Hello World

> 原文：<https://itnext.io/apache-spark-and-hadoop-hdfs-hello-world-ed6fb2077c20?source=collection_archive---------2----------------------->

![](img/b02c743f613871b32b39c6ac3d3800fb.png)

演职员表:[techcrunch.com](https://techcrunch.com/2015/07/12/spark-and-hadoop-are-friends-not-foes/?guccounter=1&guce_referrer_us=aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS8&guce_referrer_cs=3LAm_62Zyy12ECBSNt6eFQ)

这篇文章旨在帮助人们开始他们的大数据之旅，帮助他们创建一个简单的环境来测试 Apache Spark 和 Hadoop HDFS 的集成。它不打算描述什么是 Apache Spark 或 Hadoop。

# 先决条件

*   Docker 环境(本地或远程)。
*   本地机器上的一个 DNS 条目将`hadoop`映射到 Docker 主机的 IP 地址。

# Hadoop Docker 映像

*   拉动`teivah/kafka:2.9.2` Docker 图像:

```
docker pull teivah/hadoop:2.9.2
```

该映像启动了一个单节点 Hadoop 集群。

*   运行 Docker 映像:

```
docker run -ti --hostname hadoop -p 50070:50070 -p 9000:9000 -p 50075:50075 -p 50010:50010 teivah/hadoop:2.9.2
```

该命令公开了 3 个端口:50070 是 HDFS namenode 端口，50075 是 HDFS datanode 端口，9000 是 Spark ( `fs.defaultFS`配置)要访问的端口。

*   一旦容器运行，进入 web UI:[http://Hadoop:50070/](http://hadoop:50070/)然后导航到 *Utilities* - > *浏览文件系统*。
*   在根级别上传文件(您可能希望以后创建层次结构)。

# Apache Spark 客户端(Java 语言)

*   Git 克隆以下项目:[https://github.com/teivah/spark-hdfs-helloworld](https://github.com/teivah/spark-hdfs-helloworld)。
*   运行`mvn clean install`安装项目并下载依赖项。
*   打开`io.teivah.spark.SparkHdfsHelloWorld.java`。这个客户端连接到一个 HDFS 集群( *hdfs://hadoop:9000/…* )并执行一系列查询来计算每个单词在文件中出现的次数。
*   将`FILENAME` ( [第 21 行](https://github.com/teivah/spark-hdfs-helloworld/blob/372d3b8a3be9222f3c3ba9c8e71441de6fadb876/src/main/java/io/teivah/spark/SparkHdfsHelloWorld.java#L21))替换为您刚刚上传的文件的名称。
*   执行应用程序。在执行结束时，您应该在`/output`文件夹中创建一个代表结果的`part-00000`文件。

您现在已经准备好使用 Apache Spark 和 Hadoop HDFS 了。

![](img/9a80d93d4e83e996c2959e51f01b0f91.png)

# 进一步阅读

[](https://www.edureka.co/blog/spark-tutorial/) [## Spark 教程| Apache Spark 初学者指南| Edureka

### Apache Spark 是一个我们很高兴通过这个开源集群计算框架开始这一激动人心的旅程…

www.edureka.co](https://www.edureka.co/blog/spark-tutorial/) [](https://logz.io/blog/hadoop-vs-spark/) [## Hadoop 与 Spark:势均力敌的比较| Logz.io

### 每年，市场上似乎都有越来越多的分布式系统来管理数据量、种类和…

logz.io](https://logz.io/blog/hadoop-vs-spark/) ![](img/b6578984dc018c44134738b9e16e76e8.png)![](img/ec9b9710329661db6caf375f69308521.png)

在 Twitter 上关注我 [@teivah](https://twitter.com/teivah)