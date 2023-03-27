# 从 MySQL 和 MongoDB 到 Cassandra 的 PySpark ETL

> 原文：<https://itnext.io/pyspark-etl-from-mysql-and-mongodb-to-cassandra-964a5effc5e5?source=collection_archive---------1----------------------->

![](img/75a09271adb41e03bfddf3e8da14e572.png)

在 Apache Spark/PySpark 中，我们使用抽象，只有当我们想要具体化操作的结果时，才进行实际的处理。为了连接到不同的数据库和文件系统，我们通常使用现成的库。在这个故事中，您将学习如何将数据与 MySQL 和 MongoDB 结合起来，然后将其保存在 Apache Cassandra 中。

# 环境

使用 Docker，或者更准确地说是 Docker Compose 的理想时机。我们将使用 Apache Spark 运行所有数据库和 Jupyter。

```
# Use root/example as user/password credentials
version: '3.1'

services:
  notebook:
    image: jupyter/all-spark-notebook
    ports:
      - 8888:8888
      - 4040:4040
    volumes:
      - ./work:/home/jovyan/work

  cassandra:
    image: 'bitnami/cassandra:latest'

  mongo:
    image: mongo
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example

  mysql:
    image: mysql:5.7
    environment:
      MYSQL_DATABASE: 'school'
      MYSQL_USER: 'user'
      MYSQL_PASSWORD: 'password'
      MYSQL_ROOT_PASSWORD: 'password'
```

# 向 MongoDB 添加数据

我们需要一些数据。我用 Python 写了一个简单的脚本。我们假设 Mongo 里有学生的数据。

# 向 MySQL 添加数据

在 MySQL 中，我们有一个包含组的字典表。我们需要创建一个表并向其中添加数据。

# 在 Cassandra 中创建模式

如果我们想上传数据到 Cassandra，我们需要创建一个 keyspace 和一个相应的表。Cassandra 在 Docker 中，所以我们必须进去运行 cqlsh。

```
sudo docker exec -it simple-spark-etl_cassandra_1 bash
cqlsh --user cassandra --password cassandra
```

然后用适当的模式创建一个键空间和一个表。

```
CREATE KEYSPACE school WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '1'};

CREATE TABLE school.students (
    name text,
    surname text,
    age int,
    group_id int,
    group_number text,
    skills set<text>,
    something_important int,
    PRIMARY KEY (name, surname)
) ;
```

这个模式是虚构的，所以不要在其中寻找任何意义。在 Apache Cassandra 中，数据建模是特定的。在“非训练”条件下，数据应正确建模。

# PySpark ETL 到 Apache Cassandra

我们需要使用 PYSPARK_SUBMIT_ARGS 变量提供适当的库，并配置源代码。如您所见，代码并不复杂。所有的工作都由图书馆接管。

# spylon 内核的问题

最初我想用 Jupyter 中的 Spylon 内核在 Scala 中写 w.w .代码。不幸的是，它的 MongoDB 库有问题。PySpark 和 spark-shell 可以毫无问题地处理它。

# 被卖方收回的汽车

[https://github.com/zorteran/wiadro-danych-simple-spark-etl](https://github.com/zorteran/wiadro-danych-simple-spark-etl)