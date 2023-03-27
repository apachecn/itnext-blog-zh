# AWS Elastic Beanstalk —使用 Glassfish 预配置的 docker 设置 Mysql 数据源

> 原文：<https://itnext.io/aws-elastic-beanstalk-setting-up-a-mysql-datasource-with-glassfish-preconfigured-docker-d8d91cdcfe62?source=collection_archive---------2----------------------->

![](img/022cefed4e48c1e28b686d73cdaa1d03.png)

如果您愿意将 java web 应用程序部署到 AWS，而不必处理基础设施配置，那么您可能会考虑使用 Elastic Beanstalk 来实现这一点。Amazon Elastic Beanstalk 服务提供了两个预配置的 java web 应用程序平台供选择，一个是 Tomcat 自带的，另一个是 Glassfish 自带的。如果您想使用 Glassfish，这篇文章可能对您有用。

Glassfish 平台是使用 docker 设置的，因此，如果您需要在您的环境中做进一步的配置，比如添加 JDBC 连接，那么 docker 文件将是定制基本映像所必需的。也就是说，让我们开始展示在 Glassfish Elastic Beanstalk 平台上设置 Mysql 数据源所需的步骤。

基本上，我们要做的是使用下面的命令编写一个 docker 文件:

```
# Use the AWS Elastic Beanstalk Glassfish image
FROM        amazon/aws-eb-glassfish:5.0-al-onbuild-2.11.1# Exposes port 8080
EXPOSE      8080# Install Datasource dependencies and configure datasource in Glassfish
RUN         curl -L -o /usr/local/glassfish5/glassfish/domains/domain1/lib/mysql-connector-java-8.0.13.jar [http://central.maven.org/maven2/mysql/mysql-connector-java/8.0.13/mysql-connector-java-8.0.13.jar](http://central.maven.org/maven2/mysql/mysql-connector-java/8.0.13/mysql-connector-java-8.0.13.jar) && \
            asadmin start-domain domain1 && \
            asadmin create-jdbc-connection-pool --datasourceclassname com.mysql.cj.jdbc.MysqlDataSource --restype javax.sql.DataSource --property user=your_username:password=your_password:serverName=your_server_name:portNumber=3306:databaseName=ebdb:useSSL=false MysqlConnPool && \
            asadmin create-jdbc-resource --connectionpoolid MysqlConnPool jdbc/your_jdni
```

我会试着解释文件的每一部分。

首先，我们有一个 FROM 部分，它是必需的，定义了将用于构建定制 docker 容器映像的基础映像。你可以在这里找到你正在使用的平台的基础镜像名:h[ttps://docs . AWS . Amazon . com/elastic beanstalk/latest/platforms/platforms-supported . html # platforms-supported . docker preconfig](https://docs.aws.amazon.com/elasticbeanstalk/latest/platforms/platforms-supported.html#platforms-supported.dockerpreconfig)。

对于这个例子，我们使用“Amazon/AWS-e b-glassfish:5.0-al-onbuild-2 . 11 . 1”图像。

```
# Use the AWS Elastic Beanstalk Glassfish image
FROM        amazon/aws-eb-glassfish:5.0-al-onbuild-2.11.1
```

之后，我们有包含一个或多个端口号的 EXPOSE 部分。Elastic Beanstalk 使用此端口在主机和 docker 容器之间建立连接，因此可以路由公共 internet 请求。对于 Glassfish，我们需要使用 8080。

```
# Exposes port 8080
EXPOSE      8080
```

最后，我们在这篇文章中有我们想要做的最重要的部分。跑步部分。在这里，我们可以发出命令，根据需要定制预配置的映像。请注意，每个命令都需要用字符串“&&”分隔。

```
# Install Datasource dependencies and configure datasource in Glassfish
RUN         curl -L -o /usr/local/glassfish5/glassfish/domains/domain1/lib/mysql-connector-java-8.0.13.jar [http://central.maven.org/maven2/mysql/mysql-connector-java/8.0.13/mysql-connector-java-8.0.13.jar](http://central.maven.org/maven2/mysql/mysql-connector-java/8.0.13/mysql-connector-java-8.0.13.jar) && \
            asadmin start-domain domain1 && \
            asadmin create-jdbc-connection-pool --datasourceclassname com.mysql.cj.jdbc.MysqlDataSource --restype javax.sql.DataSource --property user=your_username:password=your_password:serverName=your_server_name:portNumber=3306:databaseName=ebdb:useSSL=false MysqlConnPool && \
            asadmin create-jdbc-resource --connectionpoolid MysqlConnPool jdbc/your_jdni
```

让我们一个命令一个命令地看看 Docker 文件在做什么。

```
curl -L -o /usr/local/glassfish5/glassfish/domains/domain1/lib/mysql-connector-java-8.0.13.jar [http://central.maven.org/maven2/mysql/mysql-connector-java/8.0.13/mysql-connector-java-8.0.13.jar](http://central.maven.org/maven2/mysql/mysql-connector-java/8.0.13/mysql-connector-java-8.0.13.jar)
```

在上面的代码片段中，我们使用 curl 将 mysql jdbc 连接器从 maven 资源库下载到 Glassfish lib 文件夹。我们将需要它来设置我们的数据源。

```
asadmin start-domain domain1
```

在 lib 下载之后，我们调用 asadmin，glassfish 命令行工具，使用 start-domain 启动由弹性 Beanstalk 映像配置的默认域，该域称为“domain1”。

在下面的命令中，我们使用 create-jdbc-connection-pool 命令在 glassfish 中为应用程序创建一个 jdbc 连接池。我们需要在这里传递一些参数:

*   data source class name:Mysql lib 中数据源的类名。在本例中为“com . MySQL . CJ . JDBC . MySQL data source”
*   restype:连接池资源类型。在本例中是“javax.sql.DataSource”
*   属性:数据库特定的属性，如用户、密码、端口号、数据库名称。在我的例子中，我需要用 useSSL=false 禁用 SSL。这是可选的。所有属性都用冒号分隔。

在命令的末尾，我们提供了连接池的名称。这里的名称被设置为“MysqlConnPool”。

```
asadmin create-jdbc-connection-pool --datasourceclassname com.mysql.cj.jdbc.MysqlDataSource --restype javax.sql.DataSource --property user=your_username:password=your_password:serverName=your_server_name:portNumber=3306:databaseName=ebdb:useSSL=false MysqlConnPool
```

为了完成文件内容，我们使用了 create-jdbc-resource 命令，该命令用于配置与之前通过参数 connectionpoolid 创建的连接池相关联的 JNDI 资源。

```
asadmin create-jdbc-resource --connectionpoolid MysqlConnPool jdbc/your_jdni
```

在命令的最后一部分，我们给资源命名。在本例中是“jdbc/your_jdni”。

最后，准备好 Dockerfile 后，您所要做的就是将其命名为“Dockerfile ”,并将其打包到 war 文件的顶层目录结构中。如果一切正常，那么当您将 war 文件部署到服务器时，配置就会发生。

感谢你的阅读。:)