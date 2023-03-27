# 捕手 e2e 测试工具的初学者

> 原文：<https://itnext.io/catcher-e2e-tests-tool-for-beginners-359413721e28?source=collection_archive---------9----------------------->

![](img/557be12294ad322b1b66cd9753f57b83.png)

它是一个端到端的测试工具，专门设计用于测试包含许多组件的系统。最初是作为端到端微服务测试工具开发的，它非常适合数据管道测试的需求。这里我们描述基本的[捕捉器](https://github.com/comtihon/catcher)概念。

Catcher 是一个模块化系统。它有捕捉器自带的核心模块和可根据需要单独安装的[附加](https://github.com/comtihon/catcher_modules)模块。

# 步伐

在捕手术语模块和步骤是相同的。
每个捕手测试都可以有下一个步骤类型:

*   内置岩心捕捉器步骤。包括[检查](https://catcher-test-tool.readthedocs.io/en/latest/source/internal_modules.html#module-catcher.steps.check)、 [sh](https://catcher-test-tool.readthedocs.io/en/latest/source/internal_modules.html#sh-run-shell-command) 、[循环](https://catcher-test-tool.readthedocs.io/en/latest/source/internal_modules.html#loop-loop-over-the-data)、 [http](https://catcher-test-tool.readthedocs.io/en/latest/source/internal_modules.html#http-perform-http-request) 、[等待](https://catcher-test-tool.readthedocs.io/en/latest/source/internal_modules.html#wait-delay-testcase-execution)和[其他](https://catcher-test-tool.readthedocs.io/en/latest/source/internal_modules.html)等基本步骤；
*   延伸捕手步骤。它们的实现更复杂，有时需要在系统中安装依赖项或驱动程序。比如:[卡夫卡](https://catcher-modules.readthedocs.io/en/latest/source/catcher_modules.mq.html#catcher_modules.mq.kafka.Kafka)，[弹力](https://catcher-modules.readthedocs.io/en/latest/source/catcher_modules.service.html#catcher-modules-service-elastic-module)， [docker](https://catcher-modules.readthedocs.io/en/latest/source/catcher_modules.service.html#catcher-modules-service-docker-module) ， [mongo](https://catcher-modules.readthedocs.io/en/latest/source/catcher_modules.database.html#catcher-modules-database-mongo-module) ， [s3](https://catcher-modules.readthedocs.io/en/latest/source/catcher_modules.service.html#catcher-modules-service-s3-module) 等。
*   您的[自定义](https://catcher-test-tool.readthedocs.io/en/latest/source/modules.html)步骤，用 Python、Java、sh 或任何您喜欢的语言编写。

测试的简单示例如下:

```
steps: 
  - http: # load answers via post body 
      post: 
        url: 'http://127.0.0.1:8080/load' 
        body_from_file: "data/answers.json" 
  - elastic: # check in the logs that answers were loaded 
      search: 
        url: 'http://127.0.0.1:9200' 
        index: test 
        query: 
          match: {payload : "answers loaded successfully"}
```

# 变量

具有硬编码值的步骤没有那么有用。变量是捕手的关键特征之一。完全支持 Jinja2 模板。尽可能多地使用变量，以避免代码重复，并使事情变得灵活。

## 测试局部变量

通过在`variables`部分定义变量，每一步都可以使用现有的变量。它们可以是静态的(如`token`)或计算的(如`user_email`)。

```
variables: 
  user_email: '{{ random("email") }}' 
  token: 'my_secret_token' 
steps: 
  - http: 
      post: 
      headers: {Content-Type: 'application/json', Authorization: '{{ token }}'} 
      url: 'http://127.0.0.1:8080?user={{ user_email }}' 
      body: {'foo': bar}
```

每个步骤还可以将其输出(或部分输出)注册为一个新变量:

```
variables: 
  user_email: '{{ random("email") }}' 
  token: 'my_secret_token' 
steps: 
  - mongo: # search MongoDb for user 
      request: 
        conf: 
          database: test 
          username: test 
          password: test 
          host: localhost 
          port: 27017 
        collection: 'users' 
        find_one: {'user_id': '{{ user_email }}'} 
      register: {found_user: '{{ OUTPUT }}'} 
  - check: # check token was saved 
      equals: {the: '{{ found_user["token"] }}', is: "{{ token }}"}
```

让我们仔细看看这条线:`register: {found_user: '{{ OUTPUT }}'}`。这里`OUTPUT`系统变量存储 mongo step 的输出。我们将其注册为一个新变量`found_user`。

OUTPUT 是系统变量，用于存储每个步骤的输出**。**

## 系统变量

Catcher 还可以访问您的系统环境变量。F.e .运行`export MY_ENV_HOST=localhost && catcher my_test.yml`,您的环境变量将被选取。您可以通过运行带有 **-s** 标志:`catcher -s false my_test.yml`的 Catcher 来禁用此行为。

```
steps: 
  - http: # load answers via post body 
      post: 
        url: 'http://{{ MY_ENV_HOST }}/load'
```

运行测试时，使用 **-e** 参数覆盖变量:

```
catcher -e var=value -e other_var=other_value tests/
```

覆盖优先级是:

1.  命令行变量覆盖下面的所有内容
2.  新注册的变量会覆盖下面的所有内容
3.  测试局部变量覆盖下面的所有内容
4.  库存变量覆盖下面的所有内容
5.  环境变量不覆盖任何东西

# 存货

正如您可能注意到的，无论是在测试中还是在测试变量中，硬编码服务配置都不是那么灵活，因为它们对于每个环境都是不同的。Catcher 使用库存文件进行特定于环境的配置。库存储存在`inventory`文件夹中。它们也支持模板。

您可以创建库存档案:`local.yml`并在其中设置变量:

```
airflow_web: 'http://127.0.0.1:8080' 
s3_config: 
  url: [http://127.0.0.1:9001](http://127.0.0.1:9001) 
  key_id: minio 
  secret_key: minio123
```

并且用变量创建`develop_aws.yml`:

```
airflow_web: 'http://my_airflow:8080' 
s3_config: 
  key_id: my_real_aws_key_id 
  secret_key: my_real_aws_secrey
```

运行 Catcher 时，您可以通过 **-i** 参数指定库存:

```
catcher -i inventory/local.yml test
```

# 报告

有时，当您编写测试时，您需要在每一步之后查看系统中正在发生什么。目前，Catcher 只支持 json 输出。使用 **-p** `json`选项运行:

```
catcher -p json my_test.yml
```

并查看 `reports`目录中的 json 分步报告。它将包含每一步的所有输入和输出变量。

# 装置

## 码头工人

为了快速开始，你可以使用捕手 Docker [图像](https://hub.docker.com/repository/docker/comtihon/catcher)。它包括 catcher-core、所有扩展模块、驱动程序和依赖项。它随时可以使用——您所需要做的就是用您的测试、清单等挂载您的本地目录。
运行它的完整命令是:

```
docker run -v $(pwd)/inventory:/opt/catcher/inventory 
           -v $(pwd)/resources:/opt/catcher/resources 
           -v $(pwd)/test:/opt/catcher/tests 
           -v $(pwd)/steps:/opt/catcher/steps 
           -v $(pwd)/reports:/opt/catcher/reports catcher 
           -i inventory/my_inventory.yml tests
```

Docker 中的 Catcher 通常用于 CI，而开发人员更喜欢在本地安装 Catcher 来编写测试。

## 当地的

对于本地安装，请确保您拥有 Python 3.5 以上版本。没有的话可以用 [miniconda](https://docs.conda.io/en/latest/miniconda.html) 。

运行`pip install catcher`安装捕捉器芯。它将安装捕捉器和基本[步骤](https://catcher-test-tool.readthedocs.io/en/latest/source/internal_modules.html)。如果您需要任何[扩展](https://catcher-modules.readthedocs.io/en/latest/genindex.html)步骤，您需要单独安装它们。对于卡夫卡和波斯特格雷，你必须运行`pip install catcher_modules[kafka,postgres]`

一些扩展步骤还要求先安装驱动程序和系统要求。用于 Oracle 数据库的库。