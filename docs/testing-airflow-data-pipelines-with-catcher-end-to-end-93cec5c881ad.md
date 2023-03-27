# 用捕集器端到端测试气流数据管道

> 原文：<https://itnext.io/testing-airflow-data-pipelines-with-catcher-end-to-end-93cec5c881ad?source=collection_archive---------5----------------------->

本文是关于为数据管道编写端到端测试的。它将涵盖气流，作为当今最流行的数据管道调度程序之一，也是最复杂的测试之一。对于不耐烦的人来说——这里是已经设置好一切的[库](https://github.com/comtihon/catcher_airflow_example)。

# 什么是数据管道，为什么测试它很重要？

这是一种开发模式，当我们从一个或几个数据源获取数据，处理它(不总是)并把它转移到另一个地方。可以是实时的，也可以是批量的。它可以通过不同的框架和工具(如气流，火花，弹簧批，手工制作)来完成。
但它有一个共同点:

![](img/d010795a47797ba3293e9eddfb8820e8.png)

任何数据管道都非常难以测试，因为您总是需要一个完全部署的系统，预先准备好数据集和外部服务的模拟。

假设您有一个标准的业务案例:您的后端正在将结果发送到 Postgres，您需要在 Salesforce 中更新商家的状态，以便您的客户支持代理能够即时回答客户的问题。

要测试它，你必须通过这些复杂的步骤:

*   从生产中下载数据，并对其进行个性化处理；
*   设置沙箱或直接在代码中使用模拟；

# 什么是捕手，它如何帮助你？

Catcher 是一个端到端工具 l，专门设计用于测试包含许多组件的系统。最初是作为端到端微服务测试工具开发的，它非常适合数据管道测试的需求。

主要捕集器的特点是:

*   模块化架构。它有许多不同需求的模块——从[卡夫卡](https://kafka.apache.org/)到[S3](https://aws.amazon.com/s3/)；
*   模板。它完全支持 [Jinja2](https://jinja.palletsprojects.com/en/2.11.x/) 模板，让您轻松生成数据集；
*   不同环境的不同清单文件。在本地编写您的测试，并通过更改库存文件在云环境中运行；

你可以在这里阅读[。](/catcher-e2e-tests-tool-for-beginners-359413721e28)

如果您是 Catcher 新手，您会发现这篇文章很有用。

# 管道

想象一下，您有一个处理用户注册的后端。所有新注册的用户都存储在 MySQL 中。你也有另一个后端工程与 GDPR 和退订。业务需求是—第二个后端应该以某种方式知道新创建的用户，因为它需要这些信息来进行适当的取消订阅事件匹配。最后一点是——你的后端开发人员不知道 Kafka/Rabbit，所以唯一的方法是编写一个管道，将数据从 MySQL 上传到 Postgres。

管道将:

1.  从 MySQL 获取数据并加载到 S3
2.  把 S3 的数据放到 Postgres 中

```
start >> mysql_to_s3 >> s3_to_psql >> end
```

在现实世界中，该管道的第 2 步和第 3 步最有可能被加入到自定义操作符 MySQLtoPostgresViaS3Operator 中。但是这里我们把它分开，以展示一个比一个实际的步骤管道更长的例子:)。

start 和 end 都是伪运算符。我添加它是因为它是一个定制通知 slack 等的好地方。
`mysql_to_s3`是一个 python 运算符:

```
mysql_to_s3 = PythonOperator(task_id='mysql_to_s3',
                             python_callable=mysql_to_s3,
                             retries=0,
                             dag=dag,
                             provide_context=True)
```

它只是调用`mysql_to_s3`函数:

```
def mysql_to_s3(**context): 
  mysql_hook = MySqlHook(mysql_conn_id=mysql_conn_id) 
  s3_hook = S3Hook(aws_conn_id=aws_conn_id) 
  sql = f'Select * from {mysql_tbl_name} order by email' 
  df: DataFrame = mysql_hook.get_pandas_df(sql=sql) 
  with NamedTemporaryFile(newline='', mode='w+') as f: 
    key_file = f"data/{mysql_tbl_name}/year={datetime.date.today().year}/" \ 
               f"month={datetime.date.today().strftime('%m')}/" \ 
               f"day={datetime.date.today().strftime('%d')}/" \ 
               f"{mysql_tbl_name}.csv" 
    df.to_csv(path_or_buf=f, 
              sep=",", 
              columns=df.columns.values.tolist(), 
              index=False ) 
    f.flush() 
    s3_hook.load_file(filename=f.name, 
                      key=key_file, 
                      bucket_name=bucket_name)  
    context["ti"].xcom_push(key=key_str, value=key_file) 
    f.close()
```

在这个函数中，我们通过 MySQL hook 从给定的 SQL 查询中检索 Pandas 数据帧(请注意，确保该查询不会读取太多数据，也不会使内存过载，否则会成块读取)，并将该数据帧作为 CSV 文件存储在 S3 上。

S3 文件被加载后的下一个任务:`s3_to_psql`被调用:

```
s3_to_psql = PythonOperator(task_id='s3_to_psql', 
                            python_callable=s3_to_psql, 
                            retries=0, 
                            dag=dag, 
                            provide_context=True)
```

它也是一个调用`s3_to_psql`函数的 python 操作符:

```
def s3_to_psql(**context): 
  ti = context["ti"] 
  key_file = ti.xcom_pull(dag_id='simple_example_pipeline', 
                          task_ids='mysql_to_s3', 
                          key=key_str) 
  psql_hook = PostgresHook(postgres_conn_id=postgres_conn_id) 
  s3_hook = S3Hook(aws_conn_id=aws_conn_id) 
  lines = s3_hook.read_key(key=key_file, bucket_name=bucket_name).split("\n") 
  lines = [tuple(line.split(',')) for line in lines if line != ''] 
  df = DataFrame.from_records(data=lines[1:], columns=lines[0]) 
  df.to_sql(name=psql_tbl_name, 
            con=psql_hook.get_sqlalchemy_engine(), 
            if_exists="replace", 
            index=False)
```

在这个函数中，我们将文件从 S3 读入工作内存，从中构建 Pandas 数据帧，并将其存储到 Postgres 中。

所有气流连接 id 都是在文件开头硬编码的:

```
postgres_conn_id = 'psql_conf' 
mysql_conn_id = 'mysql_conf' 
aws_conn_id = 's3_config'
```

您无需费心将其添加到气流测试环境中——Catcher 将在运行测试期间为您处理。

仅此而已。现在是检验它的时候了。

# 测试本身

让我们从定义 Catcher 测试局部变量开始:

```
variables: 
  users: 
    - uuid: '{{ random("uuid4") }}' 
      email: 'bar@test.com' 
    - uuid: '{{ random("uuid4") }}' 
      email: 'baz@test.com' 
    - uuid: '{{ random("uuid4") }}' 
      email: 'foo@test.com' 
  pipeline: 'simple_example_pipeline' 
  mysql_tbl_name: 'my_table'
```

我们设置了 Airflow 的管道名、mysql 表名和 3 个将被导出的用户。

我们在这里提供两种不同的库存:一种用于本地运行，一种用于码头运行。哪些存货由你决定，取决于具体情况。
本地[库存](https://github.com/comtihon/catcher_airflow_example/blob/master/inventory/local.yml)是:

```
mysql_conf: 'root:test@localhost:3307/test' 
psql_conf: 'postgres:postgres@localhost:5432/postgres' 
airflow_db: 'airflow:airflow@localhost:5433/airflow' 
airflow_web: 'http://127.0.0.1:8080' 
s3_config: 
  url: [http://127.0.0.1:9001](http://127.0.0.1:9001) 
  key_id: minio 
  secret_key: minio123
```

如果您已经有了`stage`或`dev` 环境设置，您可以为其添加清单，就像我们为 local 所做的一样，但是指定 DNS 名称而不是 localhost ip 地址。

Docker [inventory](https://github.com/comtihon/catcher_airflow_example/blob/master/inventory/docker.yml) 是一样的，但是用域名代替了 localhost。

```
mysql_conf: 'mysql://root:test@mysql:3306/test' 
psql_conf: 'postgresql://postgres:postgres@custom_postgres_1:5432/postgres' 
airflow_db: 'airflow:airflow@postgres:5432/airflow' 
airflow_web: 'http://webserver:8080' 
airflow_fernet: 'zp8kV516l9tKzqq9pJ2Y6cXbM3bgEWIapGwzQs6jio4=' s3_config: 
  url: [http://minio:9000](http://minio:9000) 
  key_id: 
  minio secret_key: minio123
```

## 步伐

**第一步**应该填充测试数据:它创建 MySQL 和 Postgres 表并生成数据。作为一名数据工程师，它可以让你避免枯燥的工作，简化你的生活。忘记测试数据集手动构建和生产数据导出到 csv 并复制到测试环境。以及与之相关的各种问题:数据匿名化、sql 中的正则表达式和生成器。

为了准备您的测试数据[准备](https://catcher-modules.readthedocs.io/en/latest/source/prepare_expect.html)步骤最适合:

```
prepare: 
  populate: 
    mysql: 
      conf: '{{ mysql_conf }}' 
      schema: my_table.sql 
      data: 
        my_table: my_table.csv 
    postgres: 
      conf: '{{ psql_conf }}' 
      schema: psql_tbl.sql
```

正如您在`prepare`中看到的，我们已经为 MySQL 和 Postgres 数据源定义了`populate`。对于两个数据源，这一步遵循相同的逻辑:提供配置并从指定的 SQL 文件运行 DDL 代码。`mysql_conf`和`psql_conf`值都来自当前的库存文件(您正在使用它运行测试)。

唯一不同的是，对于`mysql`，我们指定了用于填充`my_table`的输入数据。我们没有为 Postgres 指定输入数据，因为它应该在执行过程中由我们的气流管道填充。让我们更深入地了解 mysql populate 语句是如何定义的。

[my_table.sql](https://github.com/comtihon/catcher_airflow_example/blob/master/resources/my_table.sql) 是包含 create table 语句的 sql 文件。在现实世界中，您也可以在这里使用 grant access 语句，添加索引等:

```
CREATE TABLE if not exists test.my_table( 
                     user_id varchar(36) primary key, 
                     email varchar(36) NOT NULL 
                     );
```

[my_table.csv](https://github.com/comtihon/catcher_airflow_example/blob/master/resources/my_table.csv) 是一个数据文件，与一般测试方法的主要区别是——我们在这里不指定实际数据。我们从一开始就应用 Jinja2 模板来基于我们的`users`变量生成 csv 文件。所以 Catcher 最酷的特性之一:它到处支持 Jinja2 模板。

```
user_id,email
{%- for user in users -%}
{{ user.uuid }},{{ user.email }}
{%- endfor -%}
```

[psql_tbl.sql](https://github.com/comtihon/catcher_airflow_example/blob/master/resources/psql_tbl.sql) 与`my_table.sql`几乎相同，但具有另一个表名。

当所有的数据都准备好了，我们应该触发我们的管道。这是第**第二**步:

```
- airflow: 
    run: 
      config: 
        db_conf: '{{ airflow_db }}' 
        url: '{{ airflow_web }}' 
        populate_connections: true 
        fernet_key: '{{ airflow_fernet }}'  
      dag_id: '{{ pipeline }}' 
      sync: true 
      wait_timeout: 150
```

它将运行气流管道`simple_example_pipeline`，并等待它完成(或在 150 秒内失败)。并且它还将[基于您的捕集器库存文件创建](https://catcher-modules.readthedocs.io/en/latest/source/airflow.html)气流连接。

这里有一点很重要——Catcher 将在气流中创建连接，并将它们命名为库存文件中的名称:

对于库存中的`psql_conf: 'postgres:[postgres@localhost](mailto:postgres@localhost):5432/postgres'`,它将在气流中创建连接`psql_conf`。因此，为了进行工作测试，管道中的连接 id 应该与库存文件中的连接 id 相同:`postgres_conn_id = 'psql_conf'`。名字本身并不重要。

**第三步**检查 S3 文件是否已创建并下载:

```
- s3: 
    get: 
      config: '{{ s3_config }}' 
      path: 'my_awesome_bucket/data/{{ mysql_tbl_name }}/year={{ now()[:4] }}/month={{ now()[5:7] }}/day={{ now()[8:10] }}/my_table.csv' 
    register: {s3_csv: '{{ OUTPUT }}'}
```

正如我们上面所说的，Catcher 可以在任何地方应用 jinja 模板，在这里你可以看到一个例子，如何构建到我们的 S3 资源的路径。我们在原始管道中的路径是动态构建的，取决于 execution_date。我们现在使用内置函数()，它以字符串形式返回当前日期时间，并应用一些 python 字符串操作，如`[5:7]`来仅检索字符串的一部分。我们提取资源并将步骤的输出注册为一个新变量`s3_csv`。

**接下来的**两步是从资源文件中加载内容并与`s3_csv`进行比较(我们在原始气流管道中的最后一步):

```
- echo: {from_file: 'my_table.csv', register: {expect_csv: '{{ OUTPUT }}'}} 
- check: 
    equals: {the: '{{ s3_csv.strip() }}', is: '{{ expect_csv.strip() }}'} 
    name: 'Check data in s3 expected'
```

[echo](https://catcher-test-tool.readthedocs.io/en/latest/source/catcher.steps.html?highlight=echo#catcher.steps.echo.Echo) 步骤也可用于写入或读取文件。这里我们读取相同的资源`my_table.csv`，它用于填充 MySQL 并将 step 的输出保存到变量`expect_csv`。Echo step 还将运行 Jinja2 模板并生成适当的内容。

[检查](https://catcher-test-tool.readthedocs.io/en/latest/source/internal_modules.html?highlight=check#module-catcher.steps.check)等于[步骤用于比较`expect_csv`和`s3_csv`变量值。因为它们的内容是字符串，我们使用 python 的 string](https://catcher-test-tool.readthedocs.io/en/latest/source/internal_modules.html?highlight=check#catcher.steps.check.Equals) [strip](https://docs.python.org/3.7/library/stdtypes.html#str.strip) ()方法来删除最后的空格和结束行。

最后一个步骤是检查 Postgres 中实际写入了什么。[期望](https://catcher-modules.readthedocs.io/en/latest/source/prepare_expect.html#expect)步骤最适合我们:

```
- expect: 
    compare: 
      postgres: 
        conf: '{{ psql_conf }}' 
        data: 
          psql_tbl: 'my_table.csv' 
        strict: true 
    name: 'Postgres data match expected'
```

## 竣工清理

我们需要在测试后添加清理，以消除任何副作用，以便在干净的环境下运行其他测试。

因此，我们将块`finally`添加到测试的根:

```
finally: 
  - mysql: 
      request: 
        conf: '{{ mysql_conf }}' 
        query: 'drop table my_table' 
      name: 'Clean up mysql' 
      ignore_errors: true 
  - postgres: 
      request: 
        conf: '{{ psql_conf }}' 
        query: 'drop table psql_tbl' 
      name: 'Clean up postgres' 
      ignore_errors: true 
  - s3: 
      delete: 
        config: '{{ s3_config }}' 
        path: 'my_awesome_bucket/data/{{ mysql_tbl_name }}/year={{ now()[:4] }}/month={{ now()[5:7] }}/day={{ now()[8:10] }}/my_table.csv' 
      name: 'Clean up s3' 
      ignore_errors: true
```

我们从 mysql 和 postgres 删除所有数据，并从 S3 删除文件。`ignore_errors`意味着我们不关心操作是否失败(如果数据库中的 S3 或表上没有这样的文件)。顺便说一下，这里的好方法是将 S3 文件路径移到 Catcher 变量中，并在`S3:get`(步骤#3)和删除步骤中重用它，以减少代码重复。

# 怎么跑

## 在 docker 本地

如果你没有任何环境，你可以使用这个 docker-compose 在本地创建一个。它基于 [puckel](https://github.com/puckel/docker-airflow) -airflow docker 库。

```
docker-compose up -d
```

在您启动 docker-compose 之后，您需要在**相同的**网络中运行 catcher docker [image](https://hub.docker.com/repository/docker/comtihon/catcher) 挂载您的测试、资源并指定清单:

```
docker run -it --volume=$(pwd)/test:/opt/catcher/test \ 
               --volume=$(pwd)/resources:/opt/catcher/resources \ 
               --volume=$(pwd)/inventory:/opt/catcher/inventory \ 
               --network catcherairflowexample_default \ 
                 comtihon/catcher -i inventory/docker.yml test
```

网络非常重要，因为如果您要在本地运行 catcher，您可能会对所有服务主机 127.0.0.1 使用本地清单。您和 Catcher 都可以访问它们，**但是** Catcher 将使用您本地库存中的 127.0.0.1 填充 Airflow 连接，因此您的管道将失败，因为 docker 中的 Airflow 将无法通过 127.0.0.1 访问数据库/minio。

点击查看更多 docker 运行指令[。](https://catcher-test-tool.readthedocs.io/en/latest/source/run.html#docker)

## 远程环境中

自动化你的测试是一个很好的观点。您可以让您的配置项在每次部署到每个环境后运行 catcher。可以从 CI 代理使用 Catcher-in-docker 或 Catcher cli。只要使用适当的库存。

## 输出

如果你运行你的测试，你会看到漂亮的彩色输出。它将被分成两部分。

首先是逐步运行总结。它帮助您理解测试是如何运行的:

```
INFO:catcher:Step Create table and populate initial data OK INFO:catcher:Step Trigger pipeline simple_example_pipeline OK INFO:catcher:Step Get file from s3 OK INFO:catcher:user_id,email ea1d710b-0a7b-45f6-a1c4-52a5f7d91bce,bar@test.com cf0a3043-6958-412d-a7b0-924092e7e95b,baz@test.com e7a53958-f4aa-45e7-892e-69514990c992,foo@test.com INFO:catcher:Step echo OK INFO:catcher:Step Check data in s3 expected OK INFO:catcher:Step Postgres data match expected OK INFO:catcher:Test test/test.yml passed.
```

和以下清理部分:

```
INFO:catcher:Step Clean up mysql OK 
INFO:catcher:Step Clean up postgres OK 
INFO:catcher:Step Clean up s3 OK 
INFO:catcher:Test test/test.yml [cleanup] passed.
```

在多个测试的情况下，它们将相互跟随。

第二部分是运行总结，针对所有测试。它显示了每个测试运行的统计数据和状态。如果失败，它将显示步骤号。在我们的案例中:

```
INFO:catcher:Test run 1\. Success: 1, Fail: 0\. Total: 100% 
Test test: pass
```

# 结论

当开发人员听说端到端测试时，他们通常会想到复杂的 BDD 框架和他们需要编写的大量代码，以使一切正常工作。这不是关于捕手。创建测试不需要了解任何编程语言。

当 QA 工程师听说端到端测试时，他们通常会想到很多手动操作，在测试时很容易错过或出错。这不是关于捕手。做一次你的手动动作，并把它们放入捕手的脚本中，在每次部署时重复。

前端开发人员听到端到端测试，通常会想——“我们是前端开发人员，为什么需要端到端测试”。这不是关于捕手。在我的下一篇文章中，我将向您展示如何在一个 Catcher 脚本中自动化前端+后端测试。

测试愉快！