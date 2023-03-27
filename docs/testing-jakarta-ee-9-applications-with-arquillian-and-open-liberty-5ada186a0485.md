# 用 Arquillian 和 Open Liberty 测试 Jakarta EE 9 应用程序

> 原文：<https://itnext.io/testing-jakarta-ee-9-applications-with-arquillian-and-open-liberty-5ada186a0485?source=collection_archive---------4----------------------->

[open liberty/arquillian-liberty](https://github.com/OpenLiberty/liberty-arquillian)已经开始[添加 Jakarta EE 9 支持](https://github.com/OpenLiberty/liberty-arquillian/issues/71)。对于不耐烦的用户，你现在可以在你的项目中品尝当前的工作。

![](img/bbe525b66addc9069a161aebd3e74370.png)

在本文中，我们将尝试使用托管和远程适配器在 Open Liberty 容器上运行我们的测试。

# 先决条件

*   Java 8 或 Java 11 ( [OpenJDK](https://openjdk.java.net/install/) 或 [AdoptOpenJDK](https://adoptopenjdk.net/installation.html) )
*   最新[阿帕奇 Maven](http://maven.apache.org/download.cgi)
*   [JUnit 的基础知识 5](https://junit.org/junit5/)
*   了解[Arquillian 的基础](http://arquillian.org/guides/)

在将 Open Liberty 和 Aquilian 集成配置添加到您的项目之前，请确保您已经添加了 [Arquillian Jarkarta EE 9 和 JUnit 5 依赖项](https://github.com/hantsy/jakartaee9-starter-boilerplate/blob/master/docs/docs/arq-weld.md)。

# 配置 OpenLiberty 托管容器适配器

将`arquillian-liberty-managed-jakarta`依赖项添加到您的项目中。

```
<dependency>
    <groupId>io.openliberty.arquillian</groupId>
    <artifactId>arquillian-liberty-managed-jakarta</artifactId>
    <version>${arquillian-liberty-jakarta.version}</version>
</dependency>
```

在*属性*部分定义`arquillian-liberty-jakarta.version`属性。

```
<arquillian-liberty-jakarta.version>2.0.0-M1</arquillian-liberty-jakarta.version>
```

> *注:对于 Jakarta EE 9，*[*arquillian-liberty*](https://github.com/OpenLiberty/liberty-arquillian)*项目使用了新的名称空间，请注意 groupId(* `*arquillian-liberty-managed-jakarta*` *)，它有一个****-Jakarta****后缀。*

如果您在测试代码中使用 Jakarta Restful WS Client 与 Restful Web 服务握手，您应该将 Restful WS Client 实现添加到**测试**范围中。但是我发现 Open Liberty 21.0.0.1-beta 与之前的 Jakarta EE 8 兼容版本有点不同，Restful WS 实现改成了 Resteasy。但是 Resteasy 仍然没有为 Jakarta EE 9 提供公共版本。到目前为止，只有泽西岛完成了雅加达 EE 9 的改造。

所以添加 **jersey-client** 相关的依赖项。

```
<!-- Jersey -->
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-sse</artifactId>
    <version>${jersey.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-binding</artifactId>
    <version>${jersey.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.inject</groupId>
    <artifactId>jersey-hk2</artifactId>
    <version>${jersey.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-client</artifactId>
    <version>${jersey.version}</version>
    <scope>test</scope>
</dependency>
```

在 *arquillian.xml* 文件中创建一个容器配置。

```
<container qualifier="liberty-managed">
    <configuration>
        <property name="wlpHome">target/wlp/</property>
        <property name="serverName">defaultServer</property>
        <property name="httpPort">9080</property>
        <property name="serverStartTimeout">300</property>
    </configuration>
</container>
```

您可以配置一个 *wlpHome* 来使用本地系统中现有的 *Open Liberty* 服务器。

为了获得一个干净的环境来运行您的测试，我使用一个**依赖 Maven 插件**来下载一份 Open Liberty，并使用它来运行您的测试。在 CI 服务器中自动进行是很好的。

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>${maven-dependency-plugin.version}</version>
    <executions>
        <execution>
            <id>unpack</id>
            <phase>pre-integration-test</phase>
            <goals>
                <goal>unpack</goal>
            </goals>
            <configuration>
                <artifactItems>
                    <artifactItem>
                        <groupId>io.openliberty.beta</groupId>
                        <artifactId>openliberty-runtime</artifactId>
                        <version>${liberty.runtime.version}</version>
                        <type>zip</type>
                        <overWrite>false</overWrite>
                        <outputDirectory>${project.build.directory}</outputDirectory>
                    </artifactItem>
                    <artifactItem>
                        <groupId>io.openliberty.arquillian</groupId>
                        <artifactId>arquillian-liberty-support-jakarta</artifactId>
                        <version>${arquillian-liberty-jakarta.version}</version>
                        <type>zip</type>
                        <classifier>feature</classifier>
                        <overWrite>false</overWrite>
                        <outputDirectory>${project.build.directory}/wlp/usr</outputDirectory>
                    </artifactItem>
                </artifactItems>
            </configuration>
        </execution>
    </executions>
</plugin>
<plugin>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>${maven-failsafe-plugin.version}</version>
    <configuration>
        <systemPropertyVariables>
            <arquillian.launch>liberty-managed</arquillian.launch>
            <java.util.logging.config.file>${project.build.testOutputDirectory}/logging.properties
            </java.util.logging.config.file>
        </systemPropertyVariables>
    </configuration>
</plugin>
```

使用开放的 Liberty *server.xml* 进行测试。

```
<?xml version="1.0" encoding="UTF-8"?>
<server description="new server"> <!-- Enable features -->
    <featureManager>
        <feature>jakartaee-9.0</feature>
        <feature>usr:arquillian-support-jakarta-2.0</feature>
        <feature>localConnector-1.0</feature>
    </featureManager>

    <!-- To access this server from a remote client add a host attribute to the following element, e.g. host="*" -->
    <httpEndpoint id="defaultHttpEndpoint"
                  httpPort="9080"
                  httpsPort="9443" />

    <!-- Automatically expand WAR files and EAR files -->
    <applicationManager updateTrigger="mbean"  autoExpand="true"/></server>
```

为了让我们的 Open Liberty arquillian 适配器工作，你必须配置一个`localConnector`特性，`arquillian-support-jakarta-2.0`有助于开发，用于在运行测试时从 Open Liberty 收集更简洁的异常信息。

执行以下命令来运行测试。

```
mvn clean verify -Parq-libery-managed
```

# 配置 Open Liberty 远程容器适配器

使用 Open Liberty 远程容器适配器，您可以针对正在运行的 Open Liberty 服务器运行测试。它运行在不同的服务器上。

改为添加下面的`arquillian-liberty-remote-jakarta`依赖项。

```
<dependency>
    <groupId>io.openliberty.arquillian</groupId>
    <artifactId>arquillian-liberty-remote-jakarta</artifactId>
    <version>${arquillian-liberty-jakarta.version}</version>
</dependency>
// the jersey client depdenceies are omitted.
```

并在这个远程容器的 *arquillian.xml* 文件中添加一个*容器*部分。

```
<container qualifier="liberty-remote">
        <configuration>
            <property name="hostName">localhost</property>
            <property name="serverName">testServer</property>
            <property name="username">admin</property>
            <property name="password">admin</property>
            <property name="httpPort">9080</property>
            <property name="httpsPort">9443</property>
        </configuration>
    </container>
```

默认情况下，当您第一次启动 Open Liberty 时，它会创建一个新的 *defaultServer* 配置。

出于测试目的，我们创建了一个新的*测试服务器*。

```
server create testServer
```

用以下内容替换*[Open Liberty dir]/usr/servers/testServer/server.xml*中生成的 server . XML。

```
<?xml version="1.0" encoding="UTF-8"?>
<server description="Jakarta EE 9 test server"> <!-- Enable features -->
    <featureManager>
        <feature>jakartaee-9.0</feature>
        <feature>restConnector-2.0</feature>
    </featureManager> <quickStartSecurity userName="admin" userPassword="admin" /> <!-- Default SSL configuration enables trust for default certificates from the Java runtime -->
    <ssl id="defaultSSLConfig" trustDefaultCerts="true"/> <keyStore id="defaultKeyStore" password="password" location="key.jks" type="JKS"/> <httpEndpoint id="defaultHttpEndpoint"
                  httpPort="9080"
                  httpsPort="9443" host="*"/> <!-- Automatically expand WAR files and EAR files -->
    <applicationManager autoExpand="true" updateTrigger="mbean" />

    <logging consoleLogLevel="INFO" />
        <writeDir>${server.config.dir}/dropins</writeDir>
    </remoteFileAccess>

</server>
```

启动服务器，它将准备所需的资源定义在 *server.xml* 文件中。

```
# start server testServer
server start testServer
```

对于远程连接，它依赖于`restConnector-2.0`功能，并且它启用了 SSL 连接。

为了避免在运行我们的测试时出现 SSL 证书异常，您应该从正在运行的 OpenLiberty 服务器导出证书，并将其导入客户端 JVM 证书，就像我们在使用 Arquillian 和 Glassfish 进行的[测试中所做的那样](https://github.com/hantsy/jakartaee9-starter-boilerplate/blob/master/docs/docs/arq-glassfish.md)。

```
keytool -export -alias default -file testwlp.crt -keystore [Open Liberty install dir]/usr/servers/testServer/reources/security/key.jkskeytool -import -trustcacerts -keystore ${JAVA_HOME}/jre/lib/security/cacerts -storepass changeit -alias testwlp -file testwlp.crt -noprompt
```

> 在执行测试之前，确保目标 Open Liberty 服务器正在运行。

执行以下命令来运行测试。

```
mvn clean verfiy -Parq-liberty-managed
```

从我的 Github 中抓取一份[源代码，自己探索。](https://github.com/hantsy/jakartaee9-starter-boilerplate)