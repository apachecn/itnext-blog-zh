# 特里诺论游牧

> 原文：<https://itnext.io/trino-on-nomad-79cb398a826?source=collection_archive---------3----------------------->

![](img/931c24a1e80e442c9a02a9e2d8bbd94c.png)

克里斯·莱贾拉祖在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

[Trino(原 PrestoSQL)](https://trino.io/) 是数据湖中流行的分布式交互查询引擎。Trino 不仅可以作为查询引擎，还可以作为数据湖中的数据准备引擎。作为数据平台组件，Trino 是我在 data lake 中最喜欢使用的组件之一。在这里，我将向您展示如何在 Nomad 上部署 Trino。

# 先决条件

在开始之前，您应该可以使用以下组件。

*   Hive Metastore:在这篇文章中，Hive 目录将被添加到 Trino。如果您没有可用的 Hive Metastore，您可以在 Nomad 上看到 [Hive Metastore。](/hive-metastore-on-nomad-5378bf1e0a2c)
*   S3 或迷你 S3 对象存储
*   nomad 1 . 0 . 4 版
*   咨询模板 v0.22.1

# 构建 Trino docker 映像

因为我将使用运行在 Nomad 上的 Trino 服务器和 Trino CLI 的映像，所以我们应该首先构建它们。

## 构建 Trino 服务器 docker 映像

Trino 服务器的`Dockerfile`看起来如下:

```
FROM openjdk:11-slim

ENV APP_HOME /opt/trino-server
ENV TRINO_USER trino
ARG TRINO_VER

RUN useradd -ms /bin/bash -d ${APP_HOME} ${TRINO_USER}

RUN apt-get update && apt-get install -y curl dnsutils netcat python --no-install-recommends \
   && rm -rf /var/lib/apt/lists/*

RUN set -ex \
    && PACK_NAME=trino-server-${TRINO_VER} \
    && FILE_NAME=${PACK_NAME}.tar.gz \
    && curl -O https://repo1.maven.org/maven2/io/trino/trino-server/${TRINO_VER}/${FILE_NAME} \
    && tar -zxf ${FILE_NAME} \
    && cp -R ${PACK_NAME}/* ${APP_HOME} \
    && mkdir -p ${APP_HOME}/etc/catalog \
    && rm -rf ${FILE_NAME} \
    && rm -rf ${PACK_NAME}

RUN ls -al ${APP_HOME}

RUN chmod a+x -R ${APP_HOME}/bin
RUN chown ${TRINO_USER}: -R ${APP_HOME}

RUN set -ex \
    && echo 'trino soft nofile 131072' >> /etc/security/limits.d/trino.conf \
    && echo 'trino hard nofile 131072' >> /etc/security/limits.d/trino.conf
RUN cat /etc/security/limits.d/trino.conf

USER ${TRINO_USER}
WORKDIR ${APP_HOME}
```

Trino `356`将用于构建图像。创建并推送 Trino 服务器映像。

```
# image.
export TRINO_VERSION=356
export TRINO_IMAGE=mykidong/trino:${TRINO_VERSION}# build.
docker build . --build-arg TRINO_VER=${TRINO_VERSION} -t ${TRINO_IMAGE};## push.
docker push ${TRINO_IMAGE};
```

## 构建 Trino CLI docker 映像

Trino CLI 将用于查询 Nomad 上 Trino 服务器中的数据。让我们创造它的`Dockerfile`。

```
FROM openjdk:8-slim

RUN apt-get update && apt-get install -y curl less --no-install-recommends \
   && rm -rf /var/lib/apt/lists/*

ARG TRINO_VER
RUN curl -o /opt/trino-cli https://repo1.maven.org/maven2/io/trino/trino-cli/${TRINO_VER}/trino-cli-${TRINO_VER}-executable.jar \
   && chmod +x /opt/trino-cli

# Remove curl.
RUN apt-get --purge remove -y curl && apt-get autoremove -y
```

并建设推送 Trino CLI 的形象。

```
# image.
export TRINO_VERSION=356
export TRINO_CLI_IMAGE=mykidong/trino-cli:${TRINO_VERSION}# build.
docker build . --build-arg TRINO_VER=${TRINO_VERSION} -t ${TRINO_CLI_IMAGE};## push.
docker push ${TRINO_CLI_IMAGE};
```

# 在游牧者身上部署崔诺

我们将在 Nomad 上部署一名 Trino 协调员、三名 Trino 工作人员和一名 Trino CLI，因此 Trino 工作概述如下。

```
job "trino" {
  namespace = "trino"
  ...
  group "coordinator" {
    count = 1
    ...
  } 
  group "worker" {
    count = 3
    ...
  } 
  group "cli" {
    count = 1
    ...
  }
}
```

让我们来看看小组的详细情况`coordinator .`

```
group "coordinator" {
    count = 1
    restart {
      attempts = 3
      delay = "30s"
      interval = "5m"
      mode = "fail"
    }
    network {
      port "http" {
      }
    }
    task "trino-server" {
      driver = "docker"
      kill_timeout = "300s"
      kill_signal = "SIGTERM"
      template {
        data = <<EOF
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port={{ env "NOMAD_HOST_PORT_http" }}
query.max-memory=5GB
query.max-memory-per-node=1GB
query.max-total-memory-per-node=2GB
query.max-stage-count=200
task.writer-count=4
discovery-server.enabled=true
discovery.uri=[http://{{](/{{) env "NOMAD_IP_http" }}:{{ env "NOMAD_HOST_PORT_http" }}
EOF
        destination = "local/config.properties"
      }
      template {
        data = <<EOF
node.id={{ env "NOMAD_NAMESPACE" }}-{{ env "NOMAD_JOB_NAME" }}-{{ env "NOMAD_GROUP_NAME" }}-{{ env "NOMAD_ALLOC_ID" }}
node.environment=production
node.data-dir=/opt/trino-server/data
spiller-spill-path=/tmp
max-spill-per-node=4TB
query-max-spill-per-node=1TB
EOF
        destination = "local/node.properties"
      }
      template {
        data = <<EOF
-server
-Xmx16G
-XX:-UseBiasedLocking
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+ExplicitGCInvokesConcurrent
-XX:+ExitOnOutOfMemoryError
-XX:+HeapDumpOnOutOfMemoryError
-XX:ReservedCodeCacheSize=512M
-XX:PerMethodRecompilationCutoff=10000
-XX:PerBytecodeRecompilationCutoff=10000
-Djdk.attach.allowAttachSelf=true
-Djdk.nio.maxCachedBufferSize=2000000
EOF
        destination = "local/jvm.config"
      }
      template {
        data = <<EOF
connector.name=hive-hadoop2
hive.metastore.uri=thrift://{{range $index, $element := service "hive-metastore"}}{{if eq $index 0}}{{ .Address }}:{{ .Port }}{{end}}{{end}}
hive.allow-drop-table=true
hive.max-partitions-per-scan=1000000
hive.compression-codec=NONE
hive.s3.endpoint=[https://nginx-test.cloudchef-labs.com](https://nginx-test.cloudchef-labs.com)
hive.s3.path-style-access=true
hive.s3.ssl.enabled=true
hive.s3.max-connections=100
hive.s3.aws-access-key=cclminio
hive.s3.aws-secret-key=rhksflja!@#
EOF
        destination = "local/catalog/hive.properties"
      }
      config {
        image = "mykidong/trino:356"
        force_pull = false
        volumes = [
          "./local/config.properties:/opt/trino-server/etc/config.properties",
          "./local/node.properties:/opt/trino-server/etc/node.properties",
          "./local/jvm.config:/opt/trino-server/etc/jvm.config",
          "./local/catalog/hive.properties:/opt/trino-server/etc/catalog/hive.properties"
        ]
        command = "bin/launcher"
        args = [
          "run"
        ]
        ports = [
          "http"
        ]
        ulimit {
          nofile = "131072"
          nproc = "65536"
        }
      }
      resources {
        cpu = 200
        memory = 4096
      }
      service {
        name = "trino-coordinator"
        port = "http"
        check {
          name = "rest-tcp"
          type = "tcp"
          interval = "10s"
          timeout = "2s"
        }
      }
    }
  }
```

Trino 协调员任务将在该组中运行。该组中有四个模板。这些模板将生成 Trino 配置`config.properties, node.properties, jvm.config`和 Hive 目录配置`hive.properties`，它们将被安装到容器中的配置路径。在 service 节中，服务`trino-coordinator`将被注册到 Consul。这个`trino-coordinator`服务将被用作查询 SQL 的发现路径和接口。

且看第二组`worker`小节。

```
 group "worker" {
    count = 3
    restart {
      attempts = 3
      delay = "30s"
      interval = "5m"
      mode = "fail"
    }
    network {
      port "http" {
      }
    }
    task "await-coordinator" {
      driver = "docker"
      config {
        image        = "busybox:1.28"
        command      = "sh"
        args         = ["-c", "echo -n 'Waiting for service'; until nslookup trino-coordinator.service.consul 2>&1 >/dev/null; do echo '.'; sleep 2; done"]
        network_mode = "host"
      }
      resources {
        cpu    = 100
        memory = 128
      }
      lifecycle {
        hook    = "prestart"
        sidecar = false
      }
    }
    task "trino-server" {
      driver = "docker"
      kill_timeout = "300s"
      kill_signal = "SIGTERM"
      template {
        data = <<EOF
coordinator=false
http-server.http.port={{ env "NOMAD_HOST_PORT_http" }}
query.max-memory=5GB
query.max-memory-per-node=1GB
query.max-total-memory-per-node=2GB
query.max-stage-count=200
task.writer-count=4
discovery.uri=http://{{range $index, $element := service "trino-coordinator"}}{{if eq $index 0}}{{ .Address }}:{{ .Port }}{{end}}{{end}}
EOF
        destination = "local/config.properties"
      }
      template {
        data = <<EOF
node.id={{ env "NOMAD_NAMESPACE" }}-{{ env "NOMAD_JOB_NAME" }}-{{ env "NOMAD_GROUP_NAME" }}-{{ env "NOMAD_ALLOC_ID" }}
node.environment=production
node.data-dir=/opt/trino-server/data
spiller-spill-path=/tmp
max-spill-per-node=4TB
query-max-spill-per-node=1TB
EOF
        destination = "local/node.properties"
      }
      template {
        data = <<EOF
-server
-Xmx16G
-XX:-UseBiasedLocking
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+ExplicitGCInvokesConcurrent
-XX:+ExitOnOutOfMemoryError
-XX:+HeapDumpOnOutOfMemoryError
-XX:ReservedCodeCacheSize=512M
-XX:PerMethodRecompilationCutoff=10000
-XX:PerBytecodeRecompilationCutoff=10000
-Djdk.attach.allowAttachSelf=true
-Djdk.nio.maxCachedBufferSize=2000000
EOF
        destination = "local/jvm.config"
      }
      template {
        data = <<EOF
connector.name=hive-hadoop2
hive.metastore.uri=thrift://{{range $index, $element := service "hive-metastore"}}{{if eq $index 0}}{{ .Address }}:{{ .Port }}{{end}}{{end}}
hive.allow-drop-table=true
hive.max-partitions-per-scan=1000000
hive.compression-codec=NONE
hive.s3.endpoint=https://nginx-test.cloudchef-labs.com
hive.s3.path-style-access=true
hive.s3.ssl.enabled=true
hive.s3.max-connections=100
hive.s3.aws-access-key=cclminio
hive.s3.aws-secret-key=rhksflja!@#
EOF
        destination = "local/catalog/hive.properties"
      }
      config {
        image = "mykidong/trino:356"
        force_pull = false
        volumes = [
          "./local/config.properties:/opt/trino-server/etc/config.properties",
          "./local/node.properties:/opt/trino-server/etc/node.properties",
          "./local/jvm.config:/opt/trino-server/etc/jvm.config",
          "./local/catalog/hive.properties:/opt/trino-server/etc/catalog/hive.properties"
        ]
        command = "bin/launcher"
        args = [
          "run"
        ]
        ports = [
          "http"
        ]
        ulimit {
          nofile = "131072"
          nproc = "65536"
        }
      }
      resources {
        cpu = 200
        memory = 4096
      }
      service {
        name = "trino-worker"
        port = "http"
        check {
          name = "rest-tcp"
          type = "tcp"
          interval = "10s"
          timeout = "2s"
        }
      }
    }
  }
```

任务`await-coordinator`是一个预启动任务，将在任务`trino-server`之前运行。这个`await-coordinator`任务将等待服务`trino-coordinator`准备就绪。三名 Trino 工人将在该组中运行。

最后，让我们看看`cli`这个群体。

```
group "cli" {
  count = 1
  restart {
    attempts = 3
    delay = "30s"
    interval = "5m"
    mode = "fail"
  }
  task "await-coordinator" {
    driver = "docker"
    config {
      image        = "busybox:1.28"
      command      = "sh"
      args         = ["-c", "echo -n 'Waiting for service'; until nslookup trino-coordinator.service.consul 2>&1 >/dev/null; do echo '.'; sleep 2; done"]
      network_mode = "host"
    }
    resources {
      cpu    = 100
      memory = 128
    }
    lifecycle {
      hook    = "prestart"
      sidecar = false
    }
  }
  task "trino-cli" {
    driver = "docker"
    kill_timeout = "300s"
    kill_signal = "SIGTERM"
    config {
      image = "mykidong/trino-cli:356"
      force_pull = true
      command = "tail"
      args = [
        "-f",
        "/dev/null"
      ]
    }
    resources {
      cpu = 100
      memory = 256
    }
  }
}
```

Trino CLI 任务将在该组中运行。使用 Trino CLI，您可以访问 Trino 集群并查询数据。

完整的`trino.nomad`可以看这里:[https://gist . github . com/myki dong/37e 22 e 66026 a 7850 b 6 b 8 f 65 A8 f 6457 ce](https://gist.github.com/mykidong/37e22e66026a7850b6b8f65a8f6457ce)

首先，为 trino 集群创建一个名称空间。

```
nomad namespace apply -description "Trino Cluster" trino;
```

然后，运行 Trino nomad 作业。

```
nomad job run trino.nomad;
```

在 Nomad 上成功提交作业后，您将看到 trino 作业的状态。

```
nomad job status -namespace trino trino
...Allocations
ID        Node ID   Task Group   Version  Desired  Status   Created    Modified
81eaef50  457a8291  worker       0        run      running  2h26s ago  1h59m ago
c014f467  457a8291  coordinator  0        run      running  2h26s ago  1h59m ago
c1dcb51e  709ee9cc  worker       0        run      running  2h26s ago  1h59m ago
cf006809  457a8291  cli          0        run      running  2h26s ago  1h59m ago
d83876e0  457a8291  worker       0        run      running  2h26s ago  1h59m ago
```

我们现在在 Nomad 上运行 Trino 集群。

# 使用 Trino CLI 查询 Trino 中的数据

可以用 Trino CLI 查询 Trino 中的数据。为了在 Trino 中查询一些数据，我将在 MinIO 中创建示例表。您可以使用与 S3 兼容的另一个对象存储来存储数据。

下面的`test.json`将被用作使用 spark 在 MinIO 中创建表格的源数据。

```
{"itemId":"any-item-id0","quantity":2,"price":1000,"baseProperties":{"uid":"any-uid0","eventType":"cart-event","version":"7.0","ts":1527304486873}}
{"itemId":"any-item-id0","quantity":2,"price":1000,"baseProperties":{"uid":"any-uid0","eventType":"cart-event","version":"7.0","ts":1527304486873}}
{"itemId":"any-item-id0","quantity":2,"price":1000,"baseProperties":{"uid":"any-uid0","eventType":"cart-event","version":"7.0","ts":1527304486873}}
{"itemId":"any-item-id0","quantity":2,"price":1000,"baseProperties":{"uid":"any-uid0","eventType":"cart-event","version":"7.0","ts":1527304486873}}
{"itemId":"any-item-id0","quantity":2,"price":1000,"baseProperties":{"uid":"any-uid0","eventType":"cart-event","version":"7.0","ts":1527304486873}}
{"itemId":"any-item-id0","quantity":2,"price":1000,"baseProperties":{"uid":"any-uid0","eventType":"cart-event","version":"7.0","ts":1527304486873}}
{"itemId":"any-item-id0","quantity":2,"price":1000,"baseProperties":{"uid":"any-uid0","eventType":"cart-event","version":"7.0","ts":1527304486873}}
{"itemId":"any-item-id0","quantity":2,"price":1000,"baseProperties":{"uid":"any-uid0","eventType":"cart-event","version":"7.0","ts":1527304486873}}
```

要在 MinIO 中创建表，请编写如下 spark 作业。

```
...
String metastoreUrl = System.*getProperty*("metastoreUrl");

SparkConf sparkConf = new SparkConf().setAppName("create sample table");
sparkConf.setMaster("local[2]");

// delta lake log store for s3.
sparkConf.set("spark.delta.logStore.class", "org.apache.spark.sql.delta.storage.S3SingleDriverLogStore");

SparkSession spark = SparkSession
        .*builder*()
        .config(sparkConf)
        .enableHiveSupport()
        .getOrCreate();

Configuration hadoopConfiguration = spark.sparkContext().hadoopConfiguration();
hadoopConfiguration.set("fs.defaultFS", "s3a://mykidong");
hadoopConfiguration.set("fs.s3a.endpoint", "https://nginx-test.cloudchef-labs.com");
hadoopConfiguration.set("fs.s3a.access.key", "cclminio");
hadoopConfiguration.set("fs.s3a.secret.key", "rhksflja!@#");
hadoopConfiguration.set("fs.s3a.path.style.access", "true");
hadoopConfiguration.set("fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem");
hadoopConfiguration.set("hive.metastore.uris", "thrift://" + metastoreUrl);
hadoopConfiguration.set("hive.server2.enable.doAs", "false");
hadoopConfiguration.set("hive.metastore.client.socket.timeout", "1800");
hadoopConfiguration.set("hive.execution.engine", "spark");

// read json.
String json = StringUtils.*fileToString*("data/test.json");
String lines[] = json.split("\\r?\\n");
Dataset<Row> df = spark.read().json(new JavaSparkContext(spark.sparkContext()).parallelize(Arrays.*asList*(lines)));

df.show(10);

// write delta.
df.write().format("delta")
        .option("path", "s3a://mykidong/test-delta")
        .mode(SaveMode.*Overwrite*)
        .save();

// create delta table.
spark.sql("CREATE TABLE IF NOT EXISTS test_delta USING DELTA LOCATION 's3a://mykidong/test-delta'");

// read delta.
Dataset<Row> delta = spark.sql("select * from test_delta");

delta.show(10);

// create persistent parquet table with external path.
delta.write().format("parquet")
        .option("path", "s3a://mykidong/test-parquet")
        .mode(SaveMode.*Overwrite*)
        .saveAsTable("test_parquet");

// read parquet from table.
Dataset<Row> parquet = spark.sql("select * from test_parquet");

parquet.show(10);
```

运行 spark 作业后，将在 Hive Metastore 中创建 delta lake 表和基于 parquet 的表。

我们需要 Trino 协调器的 ip 地址和端口以及 Trino CLI 任务的分配 id，以便使用 Trino CLI 查询 Trino 中的数据。

获取 Trino 协调器的 ip 地址和端口。

```
cat <<EOF > service.tpl
{{range \$index, \$element := service "trino-coordinator"}}{{if eq \$index 0}}{{ .Address }}:{{ .Port }}{{end}}{{end}}
EOF
consul-template -template service.tpl -dry;
>
10.0.0.200:21409
```

Trino 协调器地址作为`10.0.0.200:21409`返回。

我们来获取 Nomad 上 Trino CLI 任务的分配 id。

```
nomad job status -namespace trino trino;
...
Allocations
ID        Node ID   Task Group   Version  Desired  Status   Created     Modified
81eaef50  457a8291  worker       0        run      running  30m50s ago  29m50s ago
c014f467  457a8291  coordinator  0        run      running  30m50s ago  30m8s ago
c1dcb51e  709ee9cc  worker       0        run      running  30m50s ago  29m56s ago
cf006809  457a8291  cli          0        run      running  30m50s ago  29m37s ago
d83876e0  457a8291  worker       0        run      running  30m50s ago  29m52s ago
```

任务组`cli`的分配 id 为`cf006809`。

现在，使用运行在 Nomad 上的 Trino CLI 运行以下命令来访问 Trino 中的配置单元目录。

```
export ALLOC_ID=cf006809;
export COORDINATOR=10.0.0.200:21409;
nomad alloc exec -namespace trino -task trino-cli ${ALLOC_ID} /opt/trino-cli --server ${COORDINATOR} --catalog hive --schema default;
```

如果您在 Trino 中查询数据，您将从 Trino 中得到如下结果。

```
...trino:default> show tables;
    Table
--------------
 test_delta
 test_parquet
(2 rows)Query 20210521_102401_00001_ufy36, FINISHED, 3 nodes
Splits: 36 total, 36 done (100.00%)
4.63 [2 rows, 56B] [0 rows/s, 12B/s]trino:default> select * from test_parquet;
                           baseproperties                            |    itemid    | price | quantity
---------------------------------------------------------------------+--------------+-------+----------
 {eventtype=cart-event, ts=1527304486873, uid=any-uid0, version=7.0} | any-item-id0 |  1000 |        2
 {eventtype=cart-event, ts=1527304486873, uid=any-uid0, version=7.0} | any-item-id0 |  1000 |        2
 {eventtype=cart-event, ts=1527304486873, uid=any-uid0, version=7.0} | any-item-id0 |  1000 |        2
 {eventtype=cart-event, ts=1527304486873, uid=any-uid0, version=7.0} | any-item-id0 |  1000 |        2
 {eventtype=cart-event, ts=1527304486873, uid=any-uid0, version=7.0} | any-item-id0 |  1000 |        2
 {eventtype=cart-event, ts=1527304486873, uid=any-uid0, version=7.0} | any-item-id0 |  1000 |        2
 {eventtype=cart-event, ts=1527304486873, uid=any-uid0, version=7.0} | any-item-id0 |  1000 |        2
 {eventtype=cart-event, ts=1527304486873, uid=any-uid0, version=7.0} | any-item-id0 |  1000 |        2
(8 rows)Query 20210521_102415_00003_ufy36, FINISHED, 2 nodes
Splits: 18 total, 18 done (100.00%)
5.13 [8 rows, 5.85KB] [1 rows/s, 1.14KB/s]
```

就是这样。