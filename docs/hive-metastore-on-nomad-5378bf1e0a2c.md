# Nomad 上的 Hive Metastore

> 原文：<https://itnext.io/hive-metastore-on-nomad-5378bf1e0a2c?source=collection_archive---------2----------------------->

![](img/deef98acef1851296017c9f671290102.png)

Hive Metastore 是数据湖中最重要的组件之一。Hive Metastore 用作数据湖中的数据目录。有许多执行引擎可以使用 Hive Metastore，例如 Spark、Presto、Tez 上的 Hive、Spark 上的 Hive 等。你可以在 Kubernetes 上部署 [Hive Metastore，但是在这里，我将讨论如何在 Nomad 上部署 Hive Metastore。](/hive-on-spark-in-kubernetes-115c8e9fa5c1)

# 先决条件

实际上，因为我有在 Kubernetes 上部署 Hive Metastore 的经验，所以将 Kubernetes Yaml 转换为 Nomad Job 并不困难。你可以在 Kubernetes 的 Spark 上看到 [Hive 的`Install Hive Metastore`部分，先了解一下如何在 Kubernetes 上部署 Hive Metastore。](/hive-on-spark-in-kubernetes-115c8e9fa5c1)

在 Nomad 上部署 Hive Metastore 之前，以下组件应该可用。

*   Ceph v14
*   Ceph CSI v3.3.1:使用 Ceph CSI，可以从 Ceph 块存储配置 nomad 卷。
*   S3 或迷你 S3 对象存储
*   nomad 1 . 0 . 4 版

# 为 Hive Metastore 部署 MySQL 服务器

需要部署 MySQL 服务器来保存 Hive Metastore 的元数据。我们需要在 Nomad 中创建一个名称空间。在此命名空间中，将创建与 Hive Metastore 相关的所有作业。

```
nomad namespace apply -description "Hive Metastore" hive-metastore;
```

要在 MySQL 中持久化数据，需要由 Ceph CSI 提供卷。您可以查看[如何使用 Ceph CSI](/provision-volumes-from-external-ceph-storage-on-kubernetes-and-nomad-using-ceph-csi-7ad9b15e9809) 在 Nomad 上提供卷的详细信息。在 Nomad 上注册卷之前，让我们在 ceph 中创建一个池的映像。

```
sudo rbd create csi-vol-00000000-1111-2222-bbbb-cacacacacac1 --size 2048 --pool myPool --image-feature layering;
```

向 Nomad 注册一个卷。

```
cat <<EOF > hive-metastore-mysql-volume.hcl
type = "csi"
id   = "hive-metastore-mysql"
name = "hive-metastore-mysql"
external_id     = "0001-0024-62c42aed-9839-4da6-8c09-9d220f56e924-0000000000000009-00000000-1111-2222-bbbb-cacacacacac1"
access_mode     = "single-node-writer"
attachment_mode = "file-system"
mount_options {
  fs_type = "ext4"
}
plugin_id       = "ceph-csi"
secrets {
  userID  = "admin"
  userKey = "AQAvo5JgP++oEhAAeZb1j/MTWyLGGJC6abCNFw=="
}
context {
  clusterID = "62c42aed-9839-4da6-8c09-9d220f56e924"
  pool      = "myPool"
  imageFeatures = "layering"
}
EOF# register volume.
nomad volume register -namespace=hive-metastore hive-metastore-mysql-volume.hcl;
```

音量状态可以这样看。

```
nomad volume status -namespace hive-metastore;
Container Storage Interface
ID        Name                  Plugin ID  Schedulable  Access Mode
hive-met  hive-metastore-mysql  ceph-csi   true         single-node-writer
```

这个注册的卷将安装在以下 mysql 服务器作业中。让我们在 Nomad 上运行 MySQL 服务器作业。

```
cat <<EOF > hive-metastore-mysql-server.nomad
job "hive-metastore-mysql-server" {
  namespace = "hive-metastore"
  datacenters = ["dc1"]
  type        = "service"group "mysql-server" {
    count = 1volume "ceph-mysql" {
      type      = "csi"
      read_only = false
      source    = "hive-metastore-mysql"
    }network {
      port "db" {
        static = 3306
      }
    }restart {
      attempts = 10
      interval = "5m"
      delay    = "25s"
      mode     = "delay"
    }task "mysql-server" {
      driver = "docker"volume_mount {
        volume      = "ceph-mysql"
        destination = "/srv"
        read_only   = false
      }env {
        MYSQL_ROOT_PASSWORD = "password"
      }config {
        image = "mysql:5.7"     
 args  = ["--datadir", "/srv/mysql"]
        ports = ["db"]
      }resources {
        cpu    = 500
        memory = 1024
      }service {
        name = "hive-metastore-mysql-server"
  port = "db"check {
          type     = "tcp"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }
}
EOF# run mysql server job.
nomad job run hive-metastore-mysql-server.nomad;
```

我们来看看乔布斯的状态。

```
nomad status -namespace hive-metastore;
ID                           Type     Priority  Status   Submit Date
hive-metastore-mysql-server  service  50        running  2021-05-13T05:20:53Z
```

有一些方法可以连接到运行在 Nomad 上的 MySQL 服务器。可以直接访问分配的 mysql 服务器容器，也可以像这样使用 MySQL 客户端访问。

```
mysql -h hive-metastore-mysql-server.service.consul -u root -p
```

`hive-metastore-mysql-server.service.consul`的主机名可以使用 BIND 进行解析，以转发 Consul 服务的 DNS，更多详情请查阅[Consul 文档](https://learn.hashicorp.com/tutorials/consul/dns-forwarding#bind-setup)，或者可以查看[https://myki dong . medium . com/install-nomad-cluster-d9a 40d 2206 F5](https://mykidong.medium.com/install-nomad-cluster-d9a40d2206f5)中的`Forward DNS for Consul Service Discovery`部分。

# 初始化配置单元 Metastore 数据库架构

为 Hive Metastore 部署 MySQL 服务器后，需要在 MySQL 服务器中创建 Hive Metastore DB 模式。

让我们在 Nomad 上运行 Hive Metastore 架构初始化作业。

```
cat <<EOF > init-hive-metastore-schema.nomad
job "init-hive-metastore-schema" {
  namespace = "hive-metastore"
  datacenters = ["dc1"]
  type        = "batch"group "mysql-server" {
    task "init-schema" {
      driver = "docker"  
      config {
        image = "mykidong/hivemetastore:v3.0.0"  
  command = "/opt/hive-metastore/bin/schematool"
  args  = [
    "--verbose",
    "-initSchema", 
    "-dbType", 
    "mysql" , 
    "-userName", 
    "root",
          "-passWord", "password", 
    "-url", 
    "jdbc:mysql://hive-metastore-mysql-server.service.consul:3306/metastore_db?createDatabaseIfNotExist=true&connectTimeout=1000"
  ]       
      }
      resources {
        cpu    = 100
        memory = 256
      }
    }
  }
}
EOF# run init schema batch job.
nomad job run init-hive-metastore-schema.nomad;
```

这份工作是`batch`的类型。运行作业后，MySQL 中已经创建了许多 db 表。

```
MySQL [(none)]> use metastore_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -ADatabase changed
MySQL [metastore_db]> show tables;
+---------------------------+
| Tables_in_metastore_db    |
+---------------------------+
| AUX_TABLE                 |
| BUCKETING_COLS            |
| CDS                       |
| COLUMNS_V2                |
| COMPACTION_QUEUE          |
| COMPLETED_COMPACTIONS     |
| COMPLETED_TXN_COMPONENTS  |
| CTLGS                     |
| DATABASE_PARAMS           |
| DBS                       |
| DB_PRIVS                  |
| DELEGATION_TOKENS         |
| FUNCS                     |
| FUNC_RU                   |
| GLOBAL_PRIVS              |
| HIVE_LOCKS                |
| IDXS                      |
| INDEX_PARAMS              |
| I_SCHEMA                  |
| KEY_CONSTRAINTS           |
| MASTER_KEYS               |
| METASTORE_DB_PROPERTIES   |
| MIN_HISTORY_LEVEL         |
| MV_CREATION_METADATA      |
| MV_TABLES_USED            |
| NEXT_COMPACTION_QUEUE_ID  |
| NEXT_LOCK_ID              |
| NEXT_TXN_ID               |
| NEXT_WRITE_ID             |
| NOTIFICATION_LOG          |
| NOTIFICATION_SEQUENCE     |
| NUCLEUS_TABLES            |
| PARTITIONS                |
| PARTITION_EVENTS          |
| PARTITION_KEYS            |
| PARTITION_KEY_VALS        |
| PARTITION_PARAMS          |
| PART_COL_PRIVS            |
| PART_COL_STATS            |
| PART_PRIVS                |
| REPL_TXN_MAP              |
| ROLES                     |
| ROLE_MAP                  |
| RUNTIME_STATS             |
| SCHEMA_VERSION            |
| SDS                       |
| SD_PARAMS                 |
| SEQUENCE_TABLE            |
| SERDES                    |
| SERDE_PARAMS              |
| SKEWED_COL_NAMES          |
| SKEWED_COL_VALUE_LOC_MAP  |
| SKEWED_STRING_LIST        |
| SKEWED_STRING_LIST_VALUES |
| SKEWED_VALUES             |
| SORT_COLS                 |
| TABLE_PARAMS              |
| TAB_COL_STATS             |
| TBLS                      |
| TBL_COL_PRIVS             |
| TBL_PRIVS                 |
| TXNS                      |
| TXN_COMPONENTS            |
| TXN_TO_WRITE_ID           |
| TYPES                     |
| TYPE_FIELDS               |
| VERSION                   |
| WM_MAPPING                |
| WM_POOL                   |
| WM_POOL_TO_TRIGGER        |
| WM_RESOURCEPLAN           |
| WM_TRIGGER                |
| WRITE_SET                 |
+---------------------------+
73 rows in set (0.01 sec)
```

目前，我们已经在 Nomad 上部署了 MySQL 服务器，并在 MySQL 中为 Hive Metastore 创建了 db 模式。

在继续之前，让我们检查一下在 Nomad 上重启 mysql 服务器作业后，MySQL 数据是否丢失。

在 Nomad 上重新提交 MySQL 服务器作业。

```
# stop mysql server.
nomad stop -purge -namespace hive-metastore hive-metastore-mysql-server;# run mysql server job.
nomad job run hive-metastore-mysql-server.nomad;
```

并再次访问 mysql 服务器。

```
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| metastore_db       |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)MySQL [(none)]> use metastore_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -ADatabase changed
MySQL [metastore_db]> show tables;
+---------------------------+
| Tables_in_metastore_db    |
+---------------------------+
| AUX_TABLE                 |
| BUCKETING_COLS            |
| CDS                       |
| COLUMNS_V2                |
| COMPACTION_QUEUE          |
| COMPLETED_COMPACTIONS     |
| COMPLETED_TXN_COMPONENTS  |
...
```

正如所见，Hive Metastore 的 MySQL 服务器中没有数据丢失。

# 在 Nomad 上部署配置单元 Metastore 服务器

要运行 hive metastore，我们需要用于 hadoop 配置的`core-site.xml`和用于 Hive Metastore 配置的`metastore-site.xml`。

`core-site.xml`的 hadoop 配置如下。

```
<configuration>
    <property>
        <name>fs.s3a.connection.ssl.enabled</name>
     <value>true</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>s3a://mykidong</value>
    </property>
    <property>
        <name>fs.s3a.path.style.access</name>
        <value>true</value>
    </property>
    <property>
        <name>fs.s3a.access.key</name>
        <value>cclminio</value>
    </property>
    <property>
        <name>fs.s3a.secret.key</name>
        <value>rhksflja!@#</value>
    </property>
    <property>
        <name>fs.s3a.impl</name>
        <value>org.apache.hadoop.fs.s3a.S3AFileSystem</value>
    </property>
    <property>
        <name>fs.s3a.endpoint</name>
        <value>[https://nginx-test.cloudchef-labs.com](https://nginx-test.cloudchef-labs.com)</value>
    </property>
    <property>
        <name>fs.s3a.fast.upload</name>
        <value>true</value>
    </property>
</configuration>
```

看看与 s3 相关的属性，比如 s3 存储桶、访问密钥、秘密密钥和 S3 端点，应该对它们进行更改以满足您的需求。

`metastore-site.xml`的 hive metastore 配置如下所示。

```
<configuration>
 <property>
  <name>metastore.task.threads.always</name>
  <value>org.apache.hadoop.hive.metastore.events.EventCleanerTask</value>
 </property>
 <property>
  <name>metastore.expression.proxy</name>
  <value>org.apache.hadoop.hive.metastore.DefaultPartitionExpressionProxy</value>
 </property>
 <property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
 </property>
 <property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://hive-metastore-mysql-server.service.consul:3306/metastore_db?useSSL=false</value>
 </property>
 <property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>root</value>
 </property>
 <property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>password</value>
 </property>
 <property>
  <name>metastore.warehouse.dir</name>
  <value>s3a://mykidong/warehouse/</value>
 </property>
 <property>
  <name>metastore.thrift.port</name>
  <value>9083</value>
 </property>
</configuration>
```

看看连接到 MySQL for Hive Metastore 的属性。

让我们来看看 Hive Metastore 服务器的完整作业脚本。

```
job "hive-metastore-server" {
  namespace = "hive-metastore"
  datacenters = ["dc1"]
  type = "service"
  group "hive-metastore" {
    count = 1
    network {
      port "hms" {
        static = 9083
      }
    }
 restart {
      attempts = 10
      interval = "5m"
      delay    = "25s"
      mode     = "delay"
    }
    task "hive-metastore-server" {
      template {
        data        = <<EOF
<configuration>
    <property>
        <name>fs.s3a.connection.ssl.enabled</name>
     <value>true</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>s3a://mykidong</value>
    </property>
    <property>
        <name>fs.s3a.path.style.access</name>
        <value>true</value>
    </property>
    <property>
        <name>fs.s3a.access.key</name>
        <value>cclminio</value>
    </property>
    <property>
        <name>fs.s3a.secret.key</name>
        <value>rhksflja!@#</value>
    </property>
    <property>
        <name>fs.s3a.impl</name>
        <value>org.apache.hadoop.fs.s3a.S3AFileSystem</value>
    </property>
    <property>
        <name>fs.s3a.endpoint</name>
        <value>[https://nginx-test.cloudchef-labs.com](https://nginx-test.cloudchef-labs.com)</value>
    </property>
    <property>
        <name>fs.s3a.fast.upload</name>
        <value>true</value>
    </property>
</configuration>
EOF
        destination = "local/core-site.xml"
        change_mode = "restart"
      }   
   template {
        data        = <<EOF
<configuration>
 <property>
  <name>metastore.task.threads.always</name>
  <value>org.apache.hadoop.hive.metastore.events.EventCleanerTask</value>
 </property>
 <property>
  <name>metastore.expression.proxy</name>
  <value>org.apache.hadoop.hive.metastore.DefaultPartitionExpressionProxy</value>
 </property>
 <property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
 </property>
 <property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://hive-metastore-mysql-server.service.consul:3306/metastore_db?useSSL=false</value>
 </property>
 <property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>root</value>
 </property>
 <property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>password</value>
 </property>
 <property>
  <name>metastore.warehouse.dir</name>
  <value>s3a://mykidong/warehouse/</value>
 </property>
 <property>
  <name>metastore.thrift.port</name>
  <value>9083</value>
 </property>
</configuration>
EOF
        destination = "local/metastore-site.xml"
        change_mode = "restart"
      }   
   env {
        AWS_ACCESS_KEY_ID = "cclminio"
  AWS_SECRET_ACCESS_KEY = "rhksflja!@#"
      }   
      driver = "docker"
      config {
        image = "mykidong/hivemetastore:v3.0.0"
        volumes = [
          "./local/core-site.xml:/opt/hadoop/etc/hadoop/core-site.xml",
    "./local/metastore-site.xml:/opt/hive-metastore/conf/metastore-site.xml"
        ]   
        command = "/opt/hive-metastore/bin/start-metastore" 
        args = ["-p", "9083"] 
  ports = [
          "hms",
        ]
      }   
   resources {
        cpu    = 500
        memory = 256
      }
      service {
        name = "hive-metastore"
        port = "hms"
        check {
          type     = "tcp"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }
}
```

将文件另存为`hive-metastore-server.nomad`，并在 Nomad 上运行 Hive Metastore 作业。

```
nomad job run hive-metastore-server.nomad;
```

如果已成功提交配置单元 metastore 作业，您可以看到作业的状态。

```
nomad status -namespace hive-metastore;
ID                           Type     Priority  Status   Submit Date
hive-metastore-mysql-server  service  50        running  2021-05-13T05:20:53Z
hive-metastore-server        service  50        running  2021-05-13T06:40:28Z
```

您可以在`9083`上检查到 Hive Metastore 服务器侦听的连接。

```
nc -zv hive-metastore.service.consul 9083;
```

就是这样。