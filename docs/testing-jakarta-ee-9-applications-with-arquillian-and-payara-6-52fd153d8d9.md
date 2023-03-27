# 用 Arquillian 和 Payara 6 测试 Jakarta EE 9 应用程序

> 原文：<https://itnext.io/testing-jakarta-ee-9-applications-with-arquillian-and-payara-6-52fd153d8d9?source=collection_archive---------5----------------------->

在最新稳定的 Payara 5 中，它加入了早期的 Jakarta EE 9 支持。Payara 6.2021.Alpha1 默认带来了完整的 Jakarta EE 9/9.1 支持。随着对 Payara 6 问题的 Arquillian 支持被修复，可以在 Payara 服务器上测试 Jakarta EE 9 应用程序了。

![](img/482708efea9c3ad4437bb106d4db389a.png)

照片由[蒙](https://unsplash.com/@ideasboom?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/china-landscape?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

Payara Arquillian 项目为 Payara 服务器提供了几个 Arquillian 容器适配器。

*   Payara 管理的容器适配器
*   Payara 远程容器适配器
*   Payara 嵌入式容器适配器
*   Payara 微管理容器适配器

# 示例代码

[示例代码](https://github.com/hantsy/jakartaee9-starter-boilerplate)通过我的 Github 共享。这 4 个适配器的 Arquillian 配置通过 4 个 Maven 配置文件提供。

# 先决条件

*   Java 11 ( [OpenJDK](https://openjdk.java.net/install/) 或 [AdoptOpenJDK](https://adoptopenjdk.net/installation.html) )
*   最新[阿帕奇 Maven](http://maven.apache.org/download.cgi)
*   [JUnit 的基础知识 5](https://junit.org/junit5/)
*   了解[剑术的基础](http://arquillian.org/guides/)

# 配置 Payara 托管容器适配器

将`arquillian-payara-server-managed`依赖项添加到您的项目中。

```
<dependency>
    <groupId>fish.payara.arquillian</groupId>
    <artifactId>payara-client-ee9</artifactId>
    <version>${arquillian-payara.version}</version>
    <scope>test</scope>
</dependency>
<!-- Payara Server Container adaptor -->
<dependency>
    <groupId>fish.payara.arquillian</groupId>
    <artifactId>arquillian-payara-server-managed</artifactId>
    <version>${arquillian-payara.version}</version>
    <scope>test</scope>
</dependency>
```

在 *arquillian.xml* 文件中创建一个容器配置。

```
<container qualifier="payara-managed">
    <configuration>
        <property name="allowConnectingToRunningServer">false</property>
        <property name="adminHost">localhost</property>
        <property name="adminPort">4848</property>
        <property name="enableH2">${enableDerby:true}</property>
        <property name="outputToConsole">true</property>
    </configuration>
</container>
```

配置`maven-dependency-plugin`下载一份 Payara 发行版，并使用它来运行您的测试。这是在您的 CI 服务器中连续运行测试的一个很好的选择。

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
                        <groupId>fish.payara.distributions</groupId>
                        <artifactId>payara</artifactId>
                        <version>${payara.version}</version>
                        <type>zip</type>
                        <overWrite>false</overWrite>
                        <outputDirectory>${project.build.directory}</outputDirectory>
                    </artifactItem>
                </artifactItems>
            </configuration>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>${maven-failsafe-plugin.version}</version>
    <configuration>
        <systemPropertyVariables>
            <payara.home>${project.build.directory}/payara6</payara.home>
            <arquillian.launch>payara-managed</arquillian.launch>
        </systemPropertyVariables>
    </configuration>
</plugin>
```

运行以下命令来运行测试。

```
mvn clean verify -Parq-payara-managed
```

在这种情况下，Arquillian 负责启动和停止 Payara 容器。

# 配置 Payara 远程容器适配器

使用 Arquillian Payara 远程容器，您可以针对正在运行的 Payara 服务器运行测试。它运行在不同的主机上。

改为添加下面的`arquillian-payara-server-remote`依赖项。

```
<dependency>
    <groupId>fish.payara.arquillian</groupId>
    <artifactId>payara-client-ee9</artifactId>
    <version>${arquillian-payara.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>fish.payara.arquillian</groupId>
    <artifactId>arquillian-payara-server-remote</artifactId>
    <version>${arquillian-payara.version}</version>
    <scope>test</scope>
</dependency>
```

并在该远程容器的 *arquillian.xml* 文件中添加一个*容器*段，如果与默认值不同，则配置`adminHost` `adminPort`、`adminUser`、`adminPassword`。

```
<container qualifier="payara-remote">
        <configuration>
            <property name="adminHost">localhost</property>
            <property name="adminPort">4848</property>
            <property name="adminUser">admin</property>
            <!-- if https is enabled via `asadmin enable-secure-admin` on a remote server -->
            <!-- <property name="adminHttps">true</property>-->
            <!-- if admin password is changed via `asadmin change-admin-password` -->
            <!--<property name="adminPassword">adminadmin</property>-->
            <!-- default is empty -->
            <property name="adminPassword"></property>
        </configuration>
    </container>
```

在执行测试之前，确保目标 Payara 服务器正在运行。

运行以下命令，对正在运行的 Payara 服务器运行测试。

```
mvn clean verfiy -Parq-payara-managed
```

# 配置 Payara 嵌入式容器适配器

使用 Payara 嵌入式容器适配器，您可以在嵌入式 Payara 服务器上运行测试。

将以下依赖项添加到项目中。

```
<dependency>
     <groupId>fish.payara.extras</groupId>
     <artifactId>payara-embedded-all</artifactId>
     <version>${payara.version}</version>
     <scope>test</scope>
</dependency>
<dependency>
    <groupId>fish.payara.arquillian</groupId>
    <artifactId>arquillian-payara-server-embedded</artifactId>
    <version>${arquillian-payara.version}</version>
    <scope>test</scope>
</dependency>
```

然后运行以下命令来运行测试。

```
mvn clean verfiy -Parq-payara-embdded
```

# 配置 Payara 微管理容器适配器

使用 Payara Micro managed container 适配器，您可以在 Payara Micro fat JAR 上运行测试。

将以下依赖项添加到项目中。

```
<dependency>
      <groupId>fish.payara.arquillian</groupId>
      <artifactId>payara-client-ee9</artifactId>
      <version>${arquillian-payara.version}</version>
      <scope>test</scope>
</dependency><!-- Payara Micro Managed Container Adaptor -->
<dependency>
    <groupId>fish.payara.arquillian</groupId>
    <artifactId>arquillian-payara-micro-managed</artifactId>
    <version>${arquillian-payara.version}</version>
    <scope>test</scope>
</dependency>
```

通过 maven 依赖插件下载一份 Payara Micro，设置一个系统属性`payara.microJar`指向下载的 Payara Micro JAR 的位置。

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
                <goal>copy</goal>
            </goals>
            <configuration>
                <artifactItems>
                    <artifactItem>
                        <groupId>fish.payara.extras</groupId>
                        <artifactId>payara-micro</artifactId>
                        <version>${payara.version}</version>
                        <type>jar</type>
                        <overWrite>false</overWrite>
                        <outputDirectory>${project.build.directory}</outputDirectory>
                        <destFileName>payara-micro.jar</destFileName>
                    </artifactItem>
                </artifactItems>
            </configuration>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>${maven-failsafe-plugin.version}</version>
    <configuration>
        <systemPropertyVariables>
            <payara.microJar>${project.build.directory}/payara-micro.jar</payara.microJar>
        </systemPropertyVariables>
    </configuration>
</plugin>
```

然后运行以下命令来运行测试。

```
mvn clean verfiy -Parq-payara-micro
```

从我的 Github 中获取[源代码。](https://github.com/hantsy/jakartaee9-starter-boilerplate)