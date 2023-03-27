# 介绍一个使用 NebulaGraph 作为图形数据库的元数据管理系统

> 原文：<https://itnext.io/introducing-a-metadata-management-system-that-uses-nebulagraph-as-a-graph-database-2ab5777f8d73?source=collection_archive---------4----------------------->

![](img/112c5a9cbce37816bc2dc066e90f7bda.png)

> 为了建立元数据管理和数据沿袭系统，我是否必须创建自己的图形模型和所有东西？感谢许多伟大的开源项目，答案是:不！

今天，我想与一些最好的开源项目分享我坚持的参考数据基础设施堆栈，这些项目包括现代 ETL、仪表板、元数据治理和数据沿袭管理。

# 元数据治理系统

元数据治理系统是一个提供单一视图的系统，该视图显示所有数据在何处以及如何被格式化、生成、转换、消费、呈现和拥有。

元数据治理就像所有数据仓库、数据库、表、仪表板、ETL 作业等的目录。有了元数据治理系统，人们就不必问一些多余的问题，比如“大家好，我可以更改这个表的模式吗？”、“嘿，谁知道我怎么能找到 table-view-foo-bar 的原始数据？”这可能就是为什么我们需要一个成熟数据堆栈中的元数据治理系统来处理相对大规模的数据和团队(或不断增长的数据和团队)。

对于另一个术语，数据沿袭是元数据的一种类型，应该尽可能地进行管理，以确保数据驱动团队中的信任链。数据谱系揭示了数据的生命周期——它旨在显示完整的数据流，从开始到结束。

# 参考溶液

## 动机

元数据和数据沿袭本质上都适合图形模型字段。数据历程中面向关系的查询，例如，“查找每个给定组件(即表)的所有 n 深度数据历程”，本质上是图形数据库中的查找所有路径查询。

这也解释了我作为一个分布式图形数据库 NebulaGraph 的开源软件(OSS)贡献者的一个观察结果:(从他们在讨论中的查询/图形建模，我可以告诉)许多已经在他们的技术栈中利用 NebulaGraph 的团队，正在从头开始建立他们自己的数据血统系统。

元数据治理系统需要以下一些组件:

*   元数据提取器:元数据提取器是数据堆栈的不同部分，如数据库、数据仓库、仪表板、ETL 管道和应用程序，向元数据治理系统推送数据或从中提取数据。
*   元数据存储:存储部分可以是数据库，甚至是大型 JSON 清单文件。
*   元数据目录:这可能是一个提供 API 和/或 GUI 接口来读/写元数据和数据沿袭的系统。

在 NebulaGraph 社区中，我看到许多图形数据库用户构建他们自己的数据谱系系统。目睹这项工作没有被标准化或共同贡献是令人发痒的，因为他们的大部分工作基本上是解析来自知名大数据项目的元数据，并将其存储在图形数据库中。

然后，我创建了这个固执己见的参考数据基础设施堆栈，将一些最好的开源项目放在一起。希望那些打算在 NebulaGraph 上定义和迭代他们自己的图形建模方式，并创建内部元数据和数据线性提取管道的人可以从这个项目中受益，拥有一个相对完善、设计精美的元数据治理系统，开箱即用，具有完全进化的图形模型。

为了使参考项目自包含且可运行，我尝试将数据基础设施层堆叠在一起，而不仅仅是与元数据相关的层。因此，它可能会帮助新的数据工程师，他们想尝试看看开源将现代数据实验室推进到了什么程度。

这是该参考数据堆栈中所有组件的图表，其中我将大多数组件视为元数据源:

# 数据堆栈

首先，我们来介绍一下组件。

数据库和数据仓库

为了处理和消费原始和中间数据，我们需要一个或多个数据库和/或仓库。

它可以是任何 DB/DW，包括 Hive、Apache Delta、TiDB、Cassandra、MySQL 或 Postgres。在这个项目中，我们简单地选择一个最流行的:PostgreSQL。我们的参考实验室提供第一项服务:

✅ —数据仓库:PostgreSQL

数据操作

我们应该有某种类型的数据操作设置，以使管道和环境可重复、可测试并受版本控制。

在这里，我们使用 GitLab 创建的 [Meltano](https://gitlab.com/meltano/meltano) 。

Meltano 是一个工作正常的 DataOps 平台，它以优雅的方式连接了作为提取和加载(EL)的[歌手](https://singer.io/)和作为转换(T)的 [dbt](https://getdbt.com/) 。它还连接到其他一些数据基础设施实用程序，如 Apache Superset 和 Apache Airflow。

现在，我们已经准备好数据操作了。

✅——吉托普斯:梅尔塔诺

抽取、转换、加载至目的端（extract-transform-load 的缩写）

在引擎盖下，我们将通过利用 [Singer](https://singer.io/) 和 Meltano 将来自许多不同数据源的数据 E(提取)和 L(加载)到数据目标，并用 [dbt](https://getdbt.com/) 进行 T(转换)。

✅-艾尔:歌手

✅ —时间:dbt

数据可视化

创建仪表板、图表和表格来获得所有数据的洞察力怎么样？

Apache 超集是我们可以选择的最好的可视化平台之一。现在让我们把它添加到我们的包中吧！

![](img/33d7bc8db5ed5e3e964a36253fef5c72.png)

✅-仪表板:Apache 超集

作业编排

在大多数情况下，我们的数据操作作业增长到需要协调的长时间执行的规模，这里出现了 [Apache 气流](https://airflow.apache.org/)。

✅ — DAG:阿帕奇气流

元数据治理

随着越来越多的组件和数据被引入到数据基础架构中，数据库、表、模式、仪表板、Dag、应用程序的所有生命周期中都将存在大量元数据。他们的管理员和团队可以被集中管理、连接和发现。

[Linux 基金会 Amundsen](https://www.amundsen.io/amundsen/) 就是解决这个问题的最好项目之一。

![](img/7add872f7455cfb901b0b5515f39115c.png)

✅ —数据发现:Linux 基金会阿蒙森

使用图形数据库作为事实的来源来加速多跳查询，并使用 Elasticsearch 作为全文搜索引擎，Amundsen 在下一个级别中平滑而漂亮地索引了所有元数据及其谱系。

默认情况下， [Neo4j](https://neo4j.com/) 被用作图形数据库。然而，我将在这个项目中使用[星云图](http://nebula-graph.io/)，因为我更熟悉它。

✅ —全文搜索:弹性搜索

✅——图形数据库:星云图

现在，随着堆栈中组件的出现，让我们将它们组装起来。

# 环境引导，组件概述

参考 runnable 项目是开源的，您可以在下面找到它:

*   [https://github.com/wey-gu/data-lineage-ref-solution](https://github.com/wey-gu/data-lineage-ref-solution)

我会尽力让事情变得干净和孤立。假设您运行在一个类似 UNIX 的系统上，该系统有互联网连接并安装了 Docker Compose。

> *请参考* [*此处*](https://docs.docker.com/compose/install/) *安装 Docker 和 Docker Compose 后再前进。*

我在 Ubuntu 20.04 LTS X86_64 上运行它，但在 Linux 的其他发行版或版本上应该不会有任何问题。

## 运行数据仓库/数据库

首先，让我们安装 Postgres 作为我们的数据仓库。

这个 oneliner 将帮助用 docker 创建一个在后台运行的 Postgres，当它被停止时将被清理掉(— rm)。

```
docker run --rm --name postgres \
    -e POSTGRES_PASSWORD=lineage_ref \
    -e POSTGRES_USER=lineage_ref \
    -e POSTGRES_DB=warehouse -d \
    -p 5432:5432 postgres
```

然后我们可以用 Postgres CLI 或 GUI 客户机来验证它。

> *提示:您可以使用 VS 代码扩展:* [*SQL 工具*](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools) *快速连接到多个 RDBMS(MariaDB、Postgres 等。)甚至像 Cassandra 这样的非 SQL DBMS。*

## 为 ETL 设置数据操作工具链

然后，让我们用 Singler 得到 Meltano 并安装 dbt。

Meltano 帮助我们管理 ETL 工具(作为插件)和它们的所有配置(管道)。这些元信息位于 Meltano 配置及其[系统数据库](https://docs.meltano.com/concepts/project#system-database)中，其中的配置是基于文件的(可以用 git 管理)。默认情况下，系统数据库是 SQLite。

安装 Meltano

使用 Meltano 的工作流程是启动一个 meltano 项目，并开始将 E、L 和 T 添加到配置文件中。项目的启动只需要一个 CLI 命令调用:meltano init yourprojectname，为此，我们可以使用 Python 的 package manager: pip 或通过 Docker 映像安装 meltano:

*   在 python 虚拟环境中安装 Meltano 和 pip:

```
mkdir .venv
# example in a debian flavor Linux distro
sudo apt-get install python3-dev python3-pip python3-venv python3-wheel -y
python3 -m venv .venv/meltano
source .venv/meltano/bin/activate
python3 -m pip install wheel
python3 -m pip install meltano# init a project
mkdir meltano_projects && cd meltano_projects
# replace <yourprojectname> with your own one
touch .env
meltano init <yourprojectname>
```

*   通过 Docker 安装 Meltano

```
docker pull meltano/meltano:latest
docker run --rm meltano/meltano --version# init a project
mkdir meltano_projects && cd meltano_projects# replace <yourprojectname> with your own one
touch .env
docker run --rm -v "$(pwd)":/projects \
             -w /projects --env-file .env \
             meltano/meltano init <yourprojectname>
```

除了 meltano init，还有一些其他的命令，比如 meltano etl 来执行 etl 执行，meltano 调用<plugin>来调用插件的命令，总是检查 [cheatsheet](https://docs.meltano.com/reference/command-line-interface) 来快速参考。</plugin>

梅尔塔诺界面

Meltano 还带有一个基于网络的用户界面，要启动它，只需运行:

```
meltano ui
```

它在监听 [http://localhost:5000。](http://localhost:5000.)

对于 Docker，只需运行暴露了 5000 端口的容器，这里我们最终没有提供 ui，因为容器的默认命令已经是 meltano ui 了。

```
docker run -v "$(pwd)":/project \
             -w /project \
             -p 5000:5000 \
             meltano/meltano
```

梅尔塔诺项目示例

在写这篇文章的时候，我注意到 [Pat Nadolny](https://github.com/pnadolny13) 已经在一个带有 dbt 的 Meltano 示例数据集上创建了[优秀示例](https://github.com/pnadolny13/meltano_example_implementations/tree/main/meltano_projects/singer_dbt_jaffle)(还带有 [Airflow](https://github.com/pnadolny13/meltano_example_implementations/tree/main/meltano_projects/dbt_orchestration) 和[超集](https://github.com/pnadolny13/meltano_example_implementations/tree/main/meltano_projects/jaffle_superset))!).我们不会重新创建的例子，而是使用帕特的伟大的。

> *注意，Andrew Stewart 创建了另一个版本，它的配置文件版本稍旧。*

您可以跟随[到这里](https://github.com/pnadolny13/meltano_example_implementations/tree/main/meltano_projects/singer_dbt_jaffle)来运行一个管道:

*   [tap-CSV](https://hub.meltano.com/taps/csv) (Singer)，从 CSV 文件中提取数据
*   [target-postgres](https://hub.meltano.com/targets/postgres) (Singer)，加载数据到 postgres
*   [dbt](https://hub.meltano.com/transformers/dbt) ，将数据转换成聚合表或视图

> *您应该省略用 docker 运行本地 Postgres 的步骤，因为我们已经创建了一个。请确保在您的中更改 Postgres 用户和密码。环境文件。*

基本上是这样的(如上安装 meltano):

```
git clone https://github.com/pnadolny13/meltano_example_implementations.git
cd meltano_example_implementations/meltano_projects/singer_dbt_jaffle/
meltano install
touch .env
echo PG_PASSWORD="lineage_ref" >> .env
echo PG_USERNAME="lineage_ref" >> .env
# Extract and Load(with Singer)
meltano run tap-csv target-postgres# Trasnform(with dbt)
meltano run dbt:run
# Generate dbt docs
meltano invoke dbt docs generate
# Serve generated dbt docs
meltano invoke dbt docs to serve
# Then visit [http://localhost:8080](http://localhost:8080)
```

现在，我假设您已经按照 singer_dbt_jaffle 的 [README.md](https://github.com/pnadolny13/meltano_example_implementations/tree/main/meltano_projects/singer_dbt_jaffle) 完成了对它的测试，我们可以连接到 Postgres 实例来查看加载和转换后的数据，如下所示，截图来自 VS 代码的 SQLTool:

![](img/4a9a389f8fb4074d850a140b792ec247.png)

## 为仪表板设置 BI 平台

现在，我们将数据存储在数据仓库中，通过 ETL 工具链将不同的数据源导入其中。如何使用这些数据？

像仪表板这样的 BI 工具可能是帮助我们从数据中获得洞察力的一种方式。

有了 Apache 超集，基于这些数据源的仪表板和图表可以被流畅而漂亮地创建和管理。

这个项目的焦点不在 Apache 超集本身，因此，我们简单地重用了 Pat Nadolny 在 meltano 示例中创建的实用程序[中的示例。](https://github.com/pnadolny13/meltano_example_implementations/tree/main/meltano_projects/jaffle_superset)

自举梅尔塔诺和超集

创建一个安装了 Meltano 的 python venv:

```
mkdir .venv
python3 -m venv .venv/meltano
source .venv/meltano/bin/activate
python3 -m pip install wheel
python3 -m pip install meltano
```

遵循 [Pat 的指南](https://github.com/pnadolny13/meltano_example_implementations/tree/main/meltano_projects/jaffle_superset)，稍作修改:

*   克隆 repo，进入 jaffle_superset 项目

```
git clone https://github.com/pnadolny13/meltano_example_implementations.git
cd meltano_example_implementations/meltano_projects/jaffle_superset/
```

*   修改 meltano 配置文件，让 Superset 连接到我们创建的 Postgres:

```
vim meltano_projects/jaffle_superset/meltano.yml
```

在我的例子中，我将主机名改为 10.1.1.111，这是我当前主机的 IP 地址。但是，如果您在 macOS 机器上运行它，这应该没问题，更改前后的差异是:

```
--- a/meltano_projects/jaffle_superset/meltano.yml
+++ b/meltano_projects/jaffle_superset/meltano.yml
@@ -71,7 +71,7 @@ plugins:
               A list of database driver dependencies can be found here https://superset.apache.org/docs/databases/installing-database-drivers
     config:
       database_name: my_postgres
-      sqlalchemy_uri: postgresql+psycopg2://${PG_USERNAME}:${PG_PASSWORD}@host.docker.internal:${PG_PORT}/${PG_DATABASE}
+      sqlalchemy_uri: postgresql+psycopg2://${PG_USERNAME}:${PG_PASSWORD}@10.1.1.168:${PG_PORT}/${PG_DATABASE}
       tables:
       - model.my_meltano_project.customers
       - model.my_meltano_project.orders
```

*   将 Postgres 凭据添加到。环境文件:

```
echo PG_USERNAME=lineage_ref >> .env
echo PG_PASSWORD=lineage_ref >> .env
```

*   安装 Meltano 项目，运行 ETL 管道

```
meltano install
meltano run tap-csv target-postgres dbt:run
```

*   启动超集，请注意 ui 不是 meltano 命令，而是配置文件中的用户自定义动作。

```
meltano invoke superset:ui
```

*   在另一个终端中，运行定义的命令 load_datasources

```
meltano invoke superset:load_datasources
```

*   通过 [http://localhost:8088/](http://localhost:8088/) 在网络浏览器中访问超集

我们现在应该看到超集 Web 界面:

![](img/9856c29ebbb81182f21e878b3b0032d1.png)

创建仪表板！

让我们试着在这个 Meltano 项目中定义的 Postgres 中创建一个关于 ETL 数据的仪表板:

*   单击+仪表板，填写仪表板名称，然后单击保存，然后单击+创建新图表

![](img/5d0ea587810394f3e7ba029983f643a1.png)

*   在新的图表视图中，我们应该选择一个图表类型和数据集。在这里，我选择了 orders 表作为数据源和饼图图表类型:

![](img/70817c3c5460220747515ff137ebf96b.png)

*   单击“CREATE NEW CHART ”(创建新图表)后，我们将进入图表定义视图，在该视图中，我选择了“Query of status ”(状态查询)作为维度，选择了“COUNT(数量)”作为指标。因此，我们可以看到每个订单状态分布的饼图。

![](img/7993f30ab04a59848d2a8f517d250f8f.png)

*   单击保存，它将询问该图表应添加到哪个仪表板，选择后，单击保存并转到仪表板。

![](img/8d1222769f98b1261e6b7c4a177ee7b6.png)

*   然后，在仪表板中，我们可以看到所有的图表。您可以看到，我还添加了另一个显示客户订单数量分布的图表:

![](img/905ae53a08bfdbe2c91db4a80b86d79d.png)

*   我们可以设置刷新间隔，或者按您的意愿点击按钮下载仪表板。

![](img/de685f1b087917ac6e8205260661950e.png)

挺爽的啊？目前，我们有一个简单但典型的数据堆栈，就像任何爱好数据实验室一样，一切都是开源的！

假设我们有 100 个 CSV 格式的数据集，数据仓库中有 200 个表，几个数据工程师运行不同的项目，使用和生成不同的应用程序、仪表板和数据库。当有人想要发现其中的一些表、数据集、仪表板和管道，然后甚至修改其中的一些时，这在通信工程中被证明是相当昂贵的。

接下来是我们参考项目的主要部分:元数据发现。

## 元数据发现

然后，我们正在逐步部署 Amundsen 的 NebulaGraph 和 Elasticsearch。

> *注:暂时作为阿蒙森后端* *的* [*PR NebulaGraph 还没有合并，我是*](https://github.com/amundsen-io/amundsen/pull/1817) [*和阿蒙森团队*](https://github.com/amundsen-io/rfcs/pull/48) *一起努力让它实现的。*

有了 Amundsen，我们可以在一个地方发现和管理整个数据堆栈的所有元数据。阿蒙森主要有两部分:

*   元数据摄取
*   [阿蒙森数据生成器](https://www.amundsen.io/amundsen/databuilder/)
*   元数据目录
*   [阿蒙森前端服务](https://www.amundsen.io/amundsen/frontend/)
*   [阿蒙森元数据服务](https://www.amundsen.io/amundsen/metadata/)
*   阿蒙森搜索服务公司

我们将利用 Data builder 从不同来源提取元数据，并将元数据保存到元服务的后端存储和搜索服务的后端存储中，然后我们可以从前端服务或通过元数据服务的 API 来搜索、发现和管理它们。

部署阿蒙森

```
Metadata service
```

我们将部署一个 Amundsen 集群及其 docker-compose 文件。由于 NebulaGraph 后端支持尚未合并，所以我们引用我的 fork。

首先，让我们克隆带有所有子模块的 repo:

```
git clone -b amundsen_nebula_graph --recursive git@github.com:wey-gu/amundsen.git
cd amundsen
```

然后，启动所有目录服务及其后端存储:

```
docker-compose -f docker-amundsen-nebula.yml up
```

> *您可以添加-d 来让容器在后台运行:*

```
docker-compose -f docker-amundsen-nebula.yml up -d
```

这将停止群集:

```
docker-compose -f docker-amundsen-nebula.yml stop
```

这将删除群集:

```
docker-compose -f docker-amundsen-nebula.yml down
```

由于这个 docker-compose 文件是供开发人员轻松使用和破解 Amundsen 的，而不是用于生产部署，所以它是从代码库构建图像，这在第一次会花费一些时间。

部署完成后，请稍等片刻，我们将使用 Data builder 将一些虚拟数据加载到其存储中。

```
Data builder
```

Amundsen Data builder 就像 Meltano，但用于元数据到元数据服务和搜索服务后端存储的 ETL:Nebula Graph 和 Elasticsearch。这里的数据构建器只是一个 python 模块，ETL 作业既可以作为脚本运行，也可以与 Apache Airflow 等 DAG 平台协调运行。

随着[阿蒙森数据生成器](https://github.com/amundsen-io/amundsen/tree/main/databuilder)的安装:

```
cd databuilder
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install wheel
python3 -m pip install -r requirements.txt
python3 setup.py install
```

让我们调用这个示例数据构建器 ETL 脚本来填充一些虚拟数据。

```
python3 example/scripts/sample_data_loader_nebula.pyVerify Amundsen
```

在访问 Amundsen 之前，我们需要创建一个测试用户:

```
# run a container with curl attached to amundsenfrontend
docker run -it --rm --net container:amundsenfrontend nicolaka/netshoot# Create a user with id test_user_id
curl -X PUT -v http://amundsenmetadata:5002/user \
    -H "Content-Type: application/json" \
    --data \
    '{"user_id":"test_user_id","first_name":"test","last_name":"user", "email":"test_user_id@mail.com"}'exit
```

然后我们可以在 [http://localhost:5000](http://localhost:5000/) 查看 UI，并尝试搜索 test，它应该会返回一些结果。

![](img/871f7925daf543f1da8707fd7684b2a2.png)

然后，您可以自己点击并浏览在 sample_data_loader_nebula.py 期间加载到 Amundsen 的那些虚拟元数据。

此外，您可以使用 Nebula Studio 访问图形数据库(http://localhost:7001)。

> *注意在 Nebula Studio 中，登录的默认字段是:*

*   主机:graphd:9669
*   用户:root
*   密码:星云

该图显示了 Amundsen 组件的更多细节:

```
┌────────────────────────┐ ┌────────────────────────────────────────┐
       │ Frontend:5000          │ │ Metadata Sources                       │
       ├────────────────────────┤ │ ┌────────┐ ┌─────────┐ ┌─────────────┐ │
       │ Metaservice:5001       │ │ │        │ │         │ │             │ │
       │ ┌──────────────┐       │ │ │ Foo DB │ │ Bar App │ │ X Dashboard │ │
  ┌────┼─┤ Nebula Proxy │       │ │ │        │ │         │ │             │ │
  │    │ └──────────────┘       │ │ │        │ │         │ │             │ │
  │    ├────────────────────────┤ │ └────────┘ └─────┬───┘ └─────────────┘ │
┌─┼────┤ Search searvice:5002   │ │                  │                     │
│ │    └────────────────────────┘ └──────────────────┼─────────────────────┘
│ │    ┌─────────────────────────────────────────────┼───────────────────────┐
│ │    │                                             │                       │
│ │    │ Databuilder     ┌───────────────────────────┘                       │
│ │    │                 │                                                   │
│ │    │ ┌───────────────▼────────────────┐ ┌──────────────────────────────┐ │
│ │ ┌──┼─► Extractor of Sources           ├─► nebula_search_data_extractor │ │
│ │ │  │ └───────────────┬────────────────┘ └──────────────┬───────────────┘ │
│ │ │  │ ┌───────────────▼────────────────┐ ┌──────────────▼───────────────┐ │
│ │ │  │ │ Loader filesystem_csv_nebula   │ │ Loader Elastic FS loader     │ │
│ │ │  │ └───────────────┬────────────────┘ └──────────────┬───────────────┘ │
│ │ │  │ ┌───────────────▼────────────────┐ ┌──────────────▼───────────────┐ │
│ │ │  │ │ Publisher nebula_csv_publisher │ │ Publisher Elasticsearch      │ │
│ │ │  │ └───────────────┬────────────────┘ └──────────────┬───────────────┘ │
│ │ │  └─────────────────┼─────────────────────────────────┼─────────────────┘
│ │ └────────────────┐   │                                 │
│ │    ┌─────────────┼───►─────────────────────────┐ ┌─────▼─────┐
│ │    │ Nebula Graph│   │                         │ │           │
│ └────┼─────┬───────┴───┼───────────┐     ┌─────┐ │ │           │
│      │     │           │           │     │MetaD│ │ │           │
│      │ ┌───▼──┐    ┌───▼──┐    ┌───▼──┐  └─────┘ │ │           │
│ ┌────┼─►GraphD│    │GraphD│    │GraphD│          │ │           │
│ │    │ └──────┘    └──────┘    └──────┘  ┌─────┐ │ │           │
│ │    │ :9669                             │MetaD│ │ │  Elastic  │
│ │    │ ┌────────┐ ┌────────┐ ┌────────┐  └─────┘ │ │  Search   │
│ │    │ │        │ │        │ │        │          │ │  Cluster  │
│ │    │ │StorageD│ │StorageD│ │StorageD│  ┌─────┐ │ │  :9200    │
│ │    │ │        │ │        │ │        │  │MetaD│ │ │           │
│ │    │ └────────┘ └────────┘ └────────┘  └─────┘ │ │           │
│ │    ├───────────────────────────────────────────┤ │           │
│ └────┤ Nebula Studio:7001                        │ │           │
│      └───────────────────────────────────────────┘ └─────▲─────┘
└──────────────────────────────────────────────────────────┘
```

# 连接点，元数据发现

随着基础环境的建立，让我们把一切放在一起。

还记得我们将一些数据像这样输出到 PostgreSQL 吗？

![](img/3b7bbb89b2484d9fe88bc0b021f218b8.png)

我们如何让 Amundsen 发现关于这些数据和 ETL 的元数据？

## 提取 Postgres 元数据

我们从数据源开始:首先是 Postgres。

我们为 python3 安装了 Postgres 客户端:

```
sudo apt-get install libpq-dev
pip3 install Psycopg2
```

Postgres 元数据 ETL 的执行

运行脚本来解析 Postgres 元数据:

```
export CREDENTIALS_POSTGRES_USER=lineage_ref
export CREDENTIALS_POSTGRES_PASSWORD=lineage_ref
export CREDENTIALS_POSTGRES_DATABASE=warehousepython3 example/scripts/sample_postgres_loader_nebula.py
```

如果您查看将 Postgres 元数据加载到 Nebula 的示例脚本的代码，您会发现主要代码行非常简单:

```
# part 1: PostgressMetadata --> CSV --> Nebula Graph
job = DefaultJob(
      conf=job_config,
      task=DefaultTask(
          extractor=PostgresMetadataExtractor(),
          loader=FsNebulaCSVLoader()),
      publisher=NebulaCsvPublisher())...
# part 2: Metadata stored in NebulaGraph --> Elasticsearch
extractor = NebulaSearchDataExtractor()
task = SearchMetadatatoElasticasearchTask(extractor=extractor)job = DefaultJob(conf=job_config, task=task)
```

第一项工作是在以下路径加载数据:postgresmetadata→CSV→Nebula Graph

*   PostgresMetadataExtractor 用于从 Postgres 中提取/拉取元数据，参见此处的[获取其文档。](https://www.amundsen.io/amundsen/databuilder/#postgresmetadataextractor)
*   FsNebulaCSVLoader 用于将提取的数据以 CSV 文件的形式直接存储
*   NebulaCsvPublisher 用于将元数据以 CSV 格式发布到 NebulaGraph

第二项工作是加载路径:存储在 NebulaGraph 中的元数据→ Elasticsearch

*   NebulaSearchDataExtractor 用于获取存储在星云图中的元数据
*   searchmetadatatoelastic asearchtask 用于使用 Elasticsearch 索引元数据。

> *注意，在生产中，我们可以在脚本中或通过像 Apache Airflow 这样的编排平台来触发这些作业。*

验证 Postgres 提取

搜索支付或直接访问[http://localhost:5000/table _ detail/warehouse/postgres/public/payments，](http://localhost:5000/table_detail/warehouse/postgres/public/payments,)您可以看到来自我们 Postgres 的元数据，如:

![](img/011c313ab8c2e6643399ba375da7f03e.png)

然后，像添加标签、所有者和描述这样的元数据管理操作也可以像上面的屏幕截图一样轻松完成。

## 正在提取 dbt 元数据

实际上，我们也可以从 [dbt](https://www.getdbt.com/) 本身提取元数据。

Amundsen [DbtExtractor](https://www.amundsen.io/amundsen/databuilder/#dbtextractor) 将解析 catalog.json 或 manifest.json 文件，以将元数据加载到 Amundsen 存储中(NebulaGraph 和 Elasticsearch)。

在上面的 meltano 章节中，我们已经用 meltano invoke dbt docs generate 生成了该文件，如下所示的输出告诉我们 catalog.json 文件:

```
14:23:15  Done.
14:23:15  Building catalog
14:23:15  Catalog written to /home/ubuntu/ref-data-lineage/meltano_example_implementations/meltano_projects/singer_dbt_jaffle/.meltano/transformers/dbt/target/catalog.json
```

dbt 元数据 ETL 的执行

下面是一个带有示例 dbt 输出文件的示例脚本:

示例 dbt 文件:

```
$ ls -l example/sample_data/dbt/
total 184
-rw-rw-r-- 1 w w   5320 May 15 07:17 catalog.json
-rw-rw-r-- 1 w w 177163 May 15 07:17 manifest.json
```

我们可以用以下代码加载这个示例 dbt 清单:

```
python3 example/scripts/sample_dbt_loader_nebula.py
```

从这几行 python 代码中，我们可以将这些过程描述为:

```
# part 1: Dbt manifest --> CSV --> Nebula Graph
job = DefaultJob(
      conf=job_config,
      task=DefaultTask(
          extractor=DbtExtractor(),
          loader=FsNebulaCSVLoader()),
      publisher=NebulaCsvPublisher())...
# part 2: Metadata stored in NebulaGraph --> Elasticsearch
extractor = NebulaSearchDataExtractor()
task = SearchMetadatatoElasticasearchTask(extractor=extractor)job = DefaultJob(conf=job_config, task=task)
```

与 Postgres meta ETL 的唯一区别是 extractor=DbtExtractor()，它提供了以下配置来获取有关 dbt 项目的以下信息:

*   数据库名称
*   目录 _json
*   清单 _json

```
job_config = ConfigFactory.from_dict({
  'extractor.dbt.database_name': database_name,
  'extractor.dbt.catalog_json': catalog_file_loc,  # File
  'extractor.dbt.manifest_json': json.dumps(manifest_data),  # JSON Dumped objecy
  'extractor.dbt.source_url': source_url})
```

验证 dbt 提取

搜索 dbt_demo 或访问[http://localhost:5000/table _ detail/dbt _ demo/snow flake/public/raw _ inventory _ value](http://localhost:5000/table_detail/dbt_demo/snowflake/public/raw_inventory_value)查看:

![](img/713b76421b5afee052b3d7aa47e8e1ae.png)

> *提示:我们可以有选择地启用调试日志来查看发送给 Elasticsearch 和 Nebula Graph 的内容！*

```
- logging.basicConfig(level=logging.INFO)
+ logging.basicConfig(level=logging.DEBUG)
```

或者，在 Nebula Studio 中浏览导入的数据:

首先点击“从顶点开始”，填写顶点 id:雪花://dbt _ demo . public/fact _ warehouse _ inventory

![](img/b510d166a7876a3c6dc61edb44277ec1.png)

然后，我们可以看到顶点显示为粉红色的点。让我们使用以下命令修改扩展选项:

*   方向:双向
*   步骤:单用 3 个

![](img/0ecbdffca16b71bfbf7e88198784c72f.png)

双击顶点(点)，它将双向扩展 3 步:

![](img/4d768032a04d86ba8a36a992eda6bf61.png)

从这个图来看，元数据的洞察力非常容易被探索，对吗？

> *提示，您可以点击👁图标来选择要显示的一些属性，这是我在捕获上述屏幕之前完成的。*

此外，我们在 Nebula Studio 中看到的内容也与 Amundsen 元数据服务的数据模型相呼应:

![](img/3af9cd19e35faa1ff646813db71adbf4.png)

最后，记住我们已经利用 dbt 在 meltano 中转换了一些数据，清单文件路径是。melt ano/transformers/dbt/target/catalog . JSON，可以尝试创建一个数据构建器作业来导入它。

## 提取超集元数据

[仪表板](https://www.amundsen.io/amundsen/databuilder/databuilder/extractor/dashboard/apache_superset/apache_superset_metadata_extractor.py)、[图表](https://www.amundsen.io/amundsen/databuilder/databuilder/extractor/dashboard/apache_superset/apache_superset_chart_extractor.py)和[与表格](https://www.amundsen.io/amundsen/databuilder/databuilder/extractor/dashboard/apache_superset/apache_superset_table_extractor.py)的关系可以通过 Amundsen data builder 提取，因为我们已经设置了一个超集仪表板，让我们尝试获取它的元数据。

超集元数据 ETL 的执行

示例超集脚本将从超集获取数据，并将元数据加载到 Nebula Graph 和 Elasticsearch 中。

```
python3 sample_superset_data_loader_nebula.py
```

如果我们将日志记录级别设置为 DEBUG，我们实际上可以看到这样的行:

```
# fetching metadata from superset
DEBUG:urllib3.connectionpool:http://localhost:8088 "POST /api/v1/security/login HTTP/1.1" 200 280
INFO:databuilder.task.task:Running a task
DEBUG:urllib3.connectionpool:Starting new HTTP connection (1): localhost:8088
DEBUG:urllib3.connectionpool:http://localhost:8088 "GET /api/v1/dashboard?q=(page_size:20,page:0,order_direction:desc) HTTP/1.1" 308 374
DEBUG:urllib3.connectionpool:http://localhost:8088 "GET /api/v1/dashboard/?q=(page_size:20,page:0,order_direction:desc) HTTP/1.1" 200 1058
...# insert DashboardDEBUG:databuilder.publisher.nebula_csv_publisher:Query: INSERT VERTEX `Dashboard` (`dashboard_url`, `name`, published_tag, publisher_last_updated_epoch_ms) VALUES  "superset_dashboard://my_cluster.1/3":("http://localhost:8088/superset/dashboard/3/","my_dashboard","unique_tag",timestamp());
...# insert a DASHBOARD_WITH_TABLE relationship/edgeINFO:databuilder.publisher.nebula_csv_publisher:Importing data in edge files: ['/tmp/amundsen/dashboard/relationships/Dashboard_Table_DASHBOARD_WITH_TABLE.csv']
DEBUG:databuilder.publisher.nebula_csv_publisher:Query:
INSERT edge `DASHBOARD_WITH_TABLE` (`END_LABEL`, `START_LABEL`, published_tag, publisher_last_updated_epoch_ms) VALUES "superset_dashboard://my_cluster.1/3"->"postgresql+psycopg2://my_cluster.warehouse/orders":("Table","Dashboard","unique_tag", timestamp()), "superset_dashboard://my_cluster.1/3"->"postgresql+psycopg2://my_cluster.warehouse/customers":("Table","Dashboard","unique_tag", timestamp());
```

验证超集仪表板提取

通过在 Amundsen 中搜索，我们现在可以查看仪表板信息。我们也可以从星云工作室核实。

![](img/93b0e063cb0fe1ee9101fb95298f0d2c.png)

> *注意，也可从* [*仪表板摄取指南*](https://www.amundsen.io/amundsen/databuilder/docs/dashboard_ingestion_guide/) *:* 中查看 Amundsen 中的仪表板型号

![](img/6e07a1dccf62e2031b967a564c61184b.png)

## 用超集预览数据

超集可以用来预览这样的表数据。相应的文档可以参考[这里的](https://www.amundsen.io/amundsen/frontend/docs/configuration/#preview-client)，其中/superset/sql_json/的 API 将被 Amundsen Frontend 调用。

![](img/a831b789d393998584945ea340ca747b.png)

## 启用数据沿袭

默认情况下，不启用数据沿袭，我们可以通过以下方式启用它:

1.  转到 Amundsen repo，这也是我们运行 docker-compose-f docker-Amundsen-nebula . yml up 命令的地方

```
cd amundsen
```

1.  修改前端 JS 配置:

```
--- a/frontend/amundsen_application/static/js/config/config-default.ts
+++ b/frontend/amundsen_application/static/js/config/config-default.ts
   tableLineage: {
-    inAppListEnabled: false,
-    inAppPageEnabled: false,
+    inAppListEnabled: true,
+    inAppPageEnabled: true,
     externalEnabled: false,
     iconPath: 'PATH_TO_ICON',
     isBeta: false,
```

1.  现在让我们再次为 docker 映像运行 build，其中前端映像将被重新构建。

```
docker-compose -f docker-amundsen-nebula.yml build
```

然后，重新运行 up -d 以确保使用新配置重新创建前端容器:

```
docker-compose -f docker-amundsen-nebula.yml up -d
```

我们可以看到这样的情况:

```
$ docker-compose -f docker-amundsen-nebula.yml up -d
...
Recreating amundsenfrontend           ... done
```

之后，我们可以访问[http://localhost:5000/Lineage/table/gold/hive/test _ schema/test _ table 1](http://localhost:5000/lineage/table/gold/hive/test_schema/test_table1)来查看沿袭显示为:

![](img/f2db10f51fa9a4a2ac9931783d4bdd5c.png)

我们可以单击下游(如果有)来查看此表的下游资源:

![](img/35bda01ebadf94c73050042876f10444.png)

或者单击沿袭查看图表:

![](img/d245231c1adf5aa2edfa709b92d06c0f.png)

也有用于血统查询的 API。这里有一个使用 cURL 进行查询的例子，我们像以前创建用户一样利用 netshoot 容器。

```
docker run -it --rm --net container:amundsenfrontend nicolaka/netshootcurl "http://amundsenmetadata:5002/table/snowflake://dbt_demo.public/raw_inventory_value/lineage?depth=3&direction=both"
```

上面的 API 调用是在上游和下游两个方向上查询 linage，深度为 3 的表雪花://dbt _ demo . public/raw _ inventory _ value。

结果应该是这样的:

```
{
  "depth": 3,
  "downstream_entities": [
    {
      "level": 2,
      "usage": 0,
      "key": "snowflake://dbt_demo.public/fact_daily_expenses",
      "parent": "snowflake://dbt_demo.public/fact_warehouse_inventory",
      "badges": [],
      "source": "snowflake"
    },
    {
      "level": 1,
      "usage": 0,
      "key": "snowflake://dbt_demo.public/fact_warehouse_inventory",
      "parent": "snowflake://dbt_demo.public/raw_inventory_value",
      "badges": [],
      "source": "snowflake"
    }
  ],
  "key": "snowflake://dbt_demo.public/raw_inventory_value",
  "direction": "both",
  "upstream_entities": []
}
```

事实上，这个血统数据是在我们的 [DbtExtractor](https://github.com/amundsen-io/amundsen/blob/main/databuilder/databuilder/extractor/dbt_extractor.py) 执行期间提取和加载的，其中 extractor.dbt.{DbtExtractor。EXTRACT_LINEAGE}默认为 True，因此创建了世系元数据并加载到 Amundsen。

获得星云图中的血统

使用图形数据库作为元数据存储的两个优点是:

*   图形查询本身是一个灵活的 DSL for lineage API，例如，此查询帮助我们执行 Amundsen 元数据 API 的等效查询以获取 lineage:

```
MATCH p=(t:Table) -[:HAS_UPSTREAM|:HAS_DOWNSTREAM *1..3]->(x)
WHERE id(t) == "snowflake://dbt_demo.public/raw_inventory_value" RETURN p
```

*   我们现在甚至可以在 Nebula Graph Studio 的控制台中查询它，然后单击 View Subgraphs 使它呈现在图形视图中。

![](img/a322fbca254a00be4a61fc725c4ed4ff.png)![](img/86358f6539135e66bd452de2d4b5df94.png)

提取数据沿袭

```
Dbt
```

如上所述， [DbtExtractor](https://www.amundsen.io/amundsen/databuilder/#dbtextractor) 将提取表级血统，以及 dbt ETL 管道中定义的其他信息。

```
Open Lineage
```

Amundsen 的另一个开箱即用的 LineageExtractor 是[openlinegetablelineageextractor](https://www.amundsen.io/amundsen/databuilder/#openlineagetablelineageextractor)。

[Open lineage](https://openlineage.io/) 是一个在一个地方收集不同来源的 Lineage 数据的开放框架，它可以将 Lineage 信息输出为 JSON 文件，由[openlinegetablelineageextractor](https://www.amundsen.io/amundsen/databuilder/#openlineagetablelineageextractor)提取:

```
dict_config = {
    # ...
    f'extractor.openlineage_tablelineage.{OpenLineageTableLineageExtractor.CLUSTER_NAME}': 'datalab',
    f'extractor.openlineage_tablelineage.{OpenLineageTableLineageExtractor.OL_DATASET_NAMESPACE_OVERRIDE}': 'hive_table',
    f'extractor.openlineage_tablelineage.{OpenLineageTableLineageExtractor.TABLE_LINEAGE_FILE_LOCATION}': 'input_dir/openlineage_nd.json',
}
...task = DefaultTask(
    extractor=OpenLineageTableLineageExtractor(),
    loader=FsNebulaCSVLoader())
```

# 概述

元数据治理/发现的整体思想是:

*   将所有组件作为元数据源放入堆栈(从任何 DB 或 DW 到 dbt、Airflow、Openlineage、超集等。)
*   使用 Databuilder(作为脚本或 DAG)运行元数据 ETL，使用 NebulaGraph(或其他图形数据库)和 Elasticsearch 进行存储和索引
*   从前端 UI(带有预览的超集)或 API 消费、管理和发现元数据
*   从查询和 UI 中获得更多可能性、灵活性和对 NebulaGraph 的洞察力

# 上游项目

本参考项目中使用的所有项目按字典顺序排列如下。

*   阿蒙森
*   阿帕奇气流
*   Apache 超集
*   dbt
*   弹性搜索
*   梅尔塔诺
*   [星云图](https://nebula-graph.io)
*   开放血统
*   歌手