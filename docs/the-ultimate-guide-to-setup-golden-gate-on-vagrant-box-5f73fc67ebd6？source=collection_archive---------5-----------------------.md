# 在流浪者盒子上设置金门的终极指南

> 原文：<https://itnext.io/the-ultimate-guide-to-setup-golden-gate-on-vagrant-box-5f73fc67ebd6?source=collection_archive---------5----------------------->

![](img/66f1a38917611144d49d610082653e8b.png)

由 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 的 [Chris Leipelt](https://unsplash.com/@cleipelt?utm_source=medium&utm_medium=referral) 拍摄的“背景中水晶般湛蓝的水面上的旧金山金门大桥的无人机视图”

如果你读过我之前在 [Debezium](https://iamninad.com/how-debezium-kafka-stream-can-help-you-write-cdc/) 上的帖子，我已经提到过，目前我正在开发一个平台，该平台包括从 Oracle 捕获 CDC 事件并将其发布到我们的新数据库。为此，Oracle 提供了类似于 Debezium 的 Golden Gate，将所有数据库更改发布到 Kafka 主题。然而，如果你像我一样是一名开发人员，正在 Mac OS 上工作，却没有为 Mac 安装 Oracle DB，这很令人难过😞那么这篇帖子将帮助你拥有自己的 Oracle 流浪环境。

当我在开发这个新平台时，我的一个同事告诉我，Oracle 有一个可以为在 mac 上工作的开发人员提供 Oracle 12c 的流浪映像。然后我检查了一下，发现 Oracle 已经为 Linux 和 Oracle 11 和 12c 创建了一个 GitHub repo。在那之后，我基本上把这个流浪文件作为我工作的基础，并在 Linux 机器上安装了金门设置和合流包。但是你知道手动安装是一项非常繁琐的任务。如果我写一个包含所有步骤的博客，你们会怎么样？

![](img/52b39a1e6a8fd726897d49cc4d94dab8.png)

因此，我增强了脚本来完成这项艰巨的工作，并安装了 Golden Gate + Confluent +在迁移箱供应期间为 Oracle 数据库启用一些配置。您知道哪个命令可以为您完成所有的设置和配置。遵循帖子中给出的具体步骤，您将获得 Oracle 数据库、金门和合流的神奇力量。😂

![](img/a1bcf2c2ec1ee9f38e389a516af9f413.png)

# 1.克隆流浪者盒子图像

从这里克隆我的神谕流浪叉:[https://github.com/ninadingole/vagrant-boxes.git](https://github.com/ninadingole/vagrant-boxes.git)。让我们祝愿我的 pull 请求被 oracle 贡献者团队接受，这项工作正式成为 Oracle GitHub repository 的一部分🤞🏼).

# 2.下载 Oracle 数据库 Zip

从甲骨文技术网[这里](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html)http://www . Oracle . com/tech network/database/enterprise-edition/downloads/index . html 下载 Oracle database 12c zip 文件。使用 Oracle Database 12c Release 2—Linux x86–64 版本，下载大约 3.2GB 的单个文件 zip

# 3.下载 Oracle GG 和 Oracle GG BD

现在，我们需要 Oracle Golden Gate 和 Golden Gate for Big Data zip 文件。这些文件可以从[http://www . Oracle . com/tech network/middleware/golden gate/downloads/index . html](http://www.oracle.com/technetwork/middleware/goldengate/downloads/index.html)下载。在 Linux x86–64 上下载 Oracle golden gate for Big Data 12 . 3 . 1 . 1 . 1&Oracle golden gate 12 . 3 . 0 . 1 . 4 for Oracle on Linux x86–64。

> 将所有下载的文件保存在同一个目录下。

# 4.开始流浪

至此，所有先决条件都已完成。现在运行`vagrant up`，这将开始在 Linux 机器上安装 Oracle database 12c、oracle golden gate、oracle golden gate for big data 和融合 oss 软件包。

![](img/09a3c6508ebc48176f0833f9f65deee7.png)

等到一切结束。最后，安装程序将打印 Oracle 数据库的密码，请务必复制该密码，因为这将有助于您继续下一步。现在，您已经在 Mac 或任何安装了 vagger 的机器上准备好了 CDC 开发环境。😎

![](img/241470493c4b35bec2499faf1a83a2aa.png)

全部完成

如果您不熟悉 Oracle golden gate，并且想了解如何为 Oracle 数据库配置 golden gate，请遵循本文中的以下步骤。

# 在流浪者盒子上配置金门

我们需要配置两个东西，首先是 oracle golden gate，然后是 oracle golden gate for big data。让我们首先为 Oracle 数据库配置 golden gate。请按照准确的步骤让它运行起来。您可以在 oracle golden gate [文档](https://www.oracle.com/technetwork/middleware/goldengate/documentation/index.html)中找到关于以下命令的更多信息。

## 1.将示例模式加载到 Oracle 数据库。

我们需要一个存在于已安装的 oracle 数据库中的模式，这样我们就可以配置 golden gate 来监听这个模式上发生的变化。Oracle 安装程序附带了安装中的 HR 模式 SQL 文件。我们将加载相同的到我们的数据库中。

1.  以 sysdba 身份使用 SQL shell 登录数据库:
    `sqlplus / as sysdba`
2.  更改会话以允许 Oracle 脚本执行:
    `alter session set "_ORACLE_SCRIPT"=true;`
3.  将会话更改为使用 PDB 数据库(在 SQL shell 中):
    `alter session set container=ORCLPDB1;`
4.  加载模式，并为以下命令要求的所有输入提供适当的响应:
    `@?/demo/schema/human_resources/hr_main.sql`

作为 oracle 用户执行以下命令，使用`sudo su - oracle`将流浪用户切换到 Oracle

## 2.配置 Oracle 金门大桥

1.  转到 Oracle golden gate 安装目录。
    
2.  打开金门控制台。
    `./ggsci`
3.  启动管理器。
    `> start mgr`
4.  登录到数据库。
    `> DBLOGIN USERID [[email protected]](https://iamninad.com/cdn-cgi/l/email-protection):1521/ORCLCDB PASSWORD [password copied while installation]`
5.  注册摘录。
    
6.  为表启用模式级补充日志记录。
    `> ADD SCHEMATRANDATA ORCLPDB1.HR ALLCOLS`
7.  创建提取组。
    `> ADD EXTRACT EXT1, INTEGRATED TRANLOG, BEGIN NOW`
8.  在本地系统上创建用于在线处理的跟踪，并将其与提取组相关联。
    `> ADD EXTTRAIL ./dirdat/lt EXTRACT EXT1`
9.  创建 EXT1 参数文件并粘贴文件中的内容。
    `> EDIT PARAM EXT1`

```
EXTRACT EXT1 
USERID SYSTEM@ORCLCDB, PASSWORD [password copied during installation] 
EXTTRAIL ./dirdat/lt 
SOURCECATALOG ORCLPDB1 
TABLE HR.*;
```

10.开始提取 EXT1。
`> start ext1`

11.查看经理和 ext1 的状态。
`> info all`

```
Program  Status   Group    Lag at Chkpt   Time Since Chkpt 
MANAGER  RUNNING  
EXTRACT  RUNNING   EXT1     00:00:00        00:00:00
```

Oracle 12c 的 Oracle golden gate 已配置。现在，是时候为大数据配置 Oracle golden gate 了，这将把所有的更改/ CDC 事件推送到汇合的 Kafka。

## 3.为大数据配置 Oracle Golden Gate

在执行以下步骤之前，确保将 java 路径添加到`.bashrc`文件中的`$LD_LIBRARY_PATH`环境变量。java 的位置是`/usr/lib/jvm/java-1.8.0-openjdk-XXXXX/jre/lib/amd64/server`。在安装过程中，用安装在上的当前版本替换 XXXXX。

1.  将目录更改为 oggbd
    `cd /u01/oggbd`
2.  打开 bd 控制台的金门。
    `./ggsci`
3.  创建默认目录。
    `> CREATE SUBDIRS`
    完成后使用`exit`命令退出控制台，并在`dirprm`目录下创建以下文件。
4.  使用步骤`2`再次登录 GG 控制台，然后执行以下命令，并将内容添加到参数文件
    `> edit param mgr`
    `dawd `中
5.  启动管理器。
    `> start mgr`
6.  创建复制组`rkafka`。
    `> add replicat rkafka, exttrail /u01/ogg/dirdat/lt`
7.  启动复制机。
    
8.  如果一切配置正确，查看所有状态。
    `> info all`

```
Program  Status   Group   Lag at Chkpt   Time Since Chkpt 
MANAGER  RUNNING 
REPLICAT RUNNING  RKAFKA    00:00:00       00:00:09
```

## 4.验证设置

1.  登录数据库并向任何表中添加行，如:

```
> sqlplus / as sysdba 
SQL> alter session set container=ORCLPDB1; 
SQL> INSERT INTO HR.REGIONS(REGION_ID, REGION_NAME) VALUES(47, 'FOO'); 
SQL> COMMIT;
```

2.检查是否为数据插入表
`> kafka-topics --zookeeper localhost:2181 --list`创建了 kafka 主题

```
__confluent.support.metrics 
__consumer_offsets 
_confluent-ksql-default__command_topic 
_schemas 
connect-configs 
connect-offsets 
connect-statuses 
ora-ogg-HR-REGIONS-avro
```

*   要检查 kafka 主题运行中的数据

```
> kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic ora-ogg-HR-REGIONS-avro --from-beginning
```

您可以从运行 vagger 的主机上运行`kafka-topics`和`kafka-avro-console-consumer`。这将有助于您在编码时连接到存在于流浪者虚拟机中的 Kafka 实例。此外，Oracle 数据库运行在端口“1521”上，您可以从 java 代码或 IDE 连接到该端口。

![](img/fa2e8224e3dd225563a8c9c30ce2dd2d.png)

这个流浪图片对我真的很有帮助，希望它也能帮助你们开发与 CDC 相关的应用程序。对于任何问题，在设置或配置期间，请在帖子上发表评论。

*   动物园管理员端口:2181
*   卡夫卡经纪人:9092
*   汇合架构注册表:8081
*   Oracle 数据库端口:1521

*原载于 2018 年 7 月 22 日*[*iamninad.com*](https://iamninad.com/the-ultimate-guide-to-setup-golden-gate-on-vagrant/)*。*