# 用 Arquillian 和 WildFly 测试 Jakarta EE 9 应用程序

> 原文：<https://itnext.io/testing-jakarta-ee-9-applications-with-arquillian-and-wildfly-cd108eec57e2?source=collection_archive---------3----------------------->

随着期待已久的 [Jakarta EE 9 支持](https://issues.redhat.com/projects/WFARQ/issues/WFARQ-89)问题在 [WildFly Arquillian 项目](https://github.com/wildfly/wildfly-arquillian)的最新 5.0.0.Alpha1 版本中得到修复，我们终于可以使用最新的 Arquillian 适配器在 WildFly 应用服务器上测试 Jakarta EE 9 应用程序了。

![](img/cb6aacb75e41b6ac6986b91ac79f6872.png)

照片由[叶夫根尼·内尔明](https://unsplash.com/@nelmin?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/china-landscape?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

在本帖中，我们将探索用各种各样的野生 Arquillian 适配器测试 Jakarta EE 9 应用程序。

*   *WildFly Arquillian 托管容器*，在这种情况下，Arquillian 控制器负责通过管理工具启动和停止 WildFly 服务器。
*   *WildFly Arquillian 远程容器*，Arquillian 不启动和停止 WildFly 服务器，它通过可用的远程协议，如 HTTP、JMX 等，将测试档案部署到正在运行的 WildFly 服务器。
*   *WildFly Arquillian 嵌入式容器*，类似于托管适配器，但是嵌入式适配器通过嵌入式 WildFly APIs 控制 WildFly 服务器，通常测试本身和服务器会在同一个线程上运行。
*   *WildFly Arquillian 托管域容器*，使用单个域代替独立服务器。
*   *WildFly Arquillian 远程域容器*，使用单个域代替独立服务器。
*   *wild fly Arquillian Bootable JAR 容器*，它在由 maven bootable jar 插件构建的 Bootable JAR 上部署测试档案。

这里我们集中讨论前三个适配器。

# 先决条件

*   Java 11 ( [OpenJDK](https://openjdk.java.net/install/) 或 [AdoptOpenJDK](https://adoptopenjdk.net/installation.html) )
*   最新[阿帕奇 Maven](http://maven.apache.org/download.cgi)
*   [JUnit 的基础知识 5](https://junit.org/junit5/)
*   了解阿尔奎利亚语的基本知识
*   下载一份[野花](https://www.wildfly.org)服务器，请下载*雅加达 EE 9 tech 预览*版本。

# 代码示例

从我的 Github 获取[源代码，阅读](https://github.com/hantsy/jakartaee9-starter-boilerplate)[文档](https://hantsy.github.io/jakartaee9-starter-boilerplate/)了解项目结构和现有源代码。我们将关注这些野生适配器的阿奎利亚配置。

# 配置 WildFly 管理的容器适配器

将以下依赖项添加到您的项目中，与 Jersey 相关的依赖项用于使用 *JAXRS Client* API 测试 RESTful APIs。WildFly 包括其内置的 JAXRS 实现— RESTEasy(可能由 Jakarta EE 9 迁移工具转化而来)，但目前还没有针对 Jakarta EE 9 发布的公共官方 RESTEasy 版本。

```
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
<dependency>
    <groupId>org.wildfly.arquillian</groupId>
    <artifactId>wildfly-arquillian-container-managed</artifactId>
    <version>${wildfly-arquillian.version}</version>
    <scope>test</scope>
</dependency>
```

在 *arquillian.xml* 文件中创建一个容器配置。

```
<container qualifier="wildfly-managed">
    <configuration>
        <!--<property name="jbossHome">${jbossHome:target/wildfly-18.0.1.Final}</property>-->
        <property name="serverConfig">standalone-full.xml</property>
    </configuration>
</container>
```

您可以在 *arquillian.xml* 文件中定义一个`jbossHome`属性来指定本地系统中现有 WildFly 服务器的位置。

或者，为了获得一个干净的环境来每次运行您的测试，您可以配置一个**依赖 Maven 插件**来下载 WildFly 的副本，并使用它来运行您的测试。这是在您的 CI 服务器中连续运行测试的一个很好的选择。

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
                        <groupId>org.wildfly</groupId>
                        <artifactId>wildfly-preview-dist</artifactId>
                        <version>${wildfly.version}</version>
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
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>${maven-failsafe-plugin.version}</version>
    <configuration>
        <environmentVariables>
            <JBOSS_HOME>${project.build.directory}/wildfly-preview-${wildfly.version}</JBOSS_HOME>
        </environmentVariables>
        <systemPropertyVariables>
            <arquillian.launch>wildfly-managed</arquillian.launch>
            <serverConfig>standalone-full.xml</serverConfig>
            <java.util.logging.manager>org.jboss.logmanager.LogManager</java.util.logging.manager>
        </systemPropertyVariables>
    </configuration>
</plugin>
```

> *注:我们在这里使用* `*wildfly-preview-dist*` *artifactId 下载 WildFly Jakarta EE 9 预览版。*

打开您的终端并切换到项目根文件夹。

运行以下命令对 WildFly 服务器运行测试。

```
mvn clean verify -Parq-wildfly-managed
```

# 配置 WildFly 远程容器适配器

使用 Arquillian WildFly 远程容器适配器，您可以针对正在运行的 WildFly 服务器运行测试。它运行在不同的主机上。

改为添加下面的`wildfly-arquillian-container-remote`依赖项。

```
<dependency>
    <groupId>org.wildfly.arquillian</groupId>
    <artifactId>wildfly-arquillian-container-remote</artifactId>
    <version>${wildfly-arquillian.version}</version>
    <scope>test</scope>
</dependency>
// the jersey client depdenceies are omitted.
```

并在该远程容器的 *arquillian.xml* 文件中添加一个*容器*段，并配置必要的属性，如`username`、`password`、`protocol`等。

```
<container qualifier="wildfly-remote">
    <configuration>
        <property name="managementAddress">127.0.0.1</property>
        <property name="managementPort">9990</property>
        <property name="protocol">http-remoting</property>
        <property name="username">admin</property>
        <property name="password">Admin@123</property>
    </configuration>
</container>
```

在执行测试之前，设置一个在上面代码中配置的管理员用户。转到 WildFly 安装文件夹，运行以下命令来启用用户 *admin* 。

```
<WF_HOME>/bin/add-user.sh admin Admin@123 --silent
```

并确保目标 WildFly 服务器正在运行。

执行以下命令对正在运行的 WildFly 服务器运行测试。

```
mvn clean verfiy -Parq-wildfly-managed
```

# 配置 WildFly 嵌入式容器适配器

WildFly 嵌入式适配器的配置类似于 WildFly 管理的适配器。

改为添加下面的`wildfly-arquillian-container-embedded`依赖项。

```
<dependency>
    <groupId>org.wildfly.arquillian</groupId>
    <artifactId>wildfly-arquillian-container-embedded</artifactId>
    <version>${wildfly-arquillian.version}</version>
    <scope>test</scope>
</dependency>
// the jersey client depdenceies are omitted.
```

在 *arquillian.xml* 文件中为这个 WildFly 嵌入式适配器配置一个`container`。

```
<container qualifier="wildfly-embedded">
    <configuration>
        <!--<property name="jbossHome">${jbossHome:target/wildfly-18.0.1.Final}</property>-->
        <property name="serverConfig">standalone-full.xml</property>
    </configuration>
</container>
```

同样，使用 maven dependency 插件下载 WildFly dist 的副本作为运行时服务器。

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
                        <groupId>org.wildfly</groupId>
                        <artifactId>wildfly-preview-dist</artifactId>
                        <version>${wildfly.version}</version>
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
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>${maven-failsafe-plugin.version}</version>
    <configuration>
        <environmentVariables>
            <JBOSS_HOME>${project.build.directory}/wildfly-preview-${wildfly.version}</JBOSS_HOME>
        </environmentVariables>
        <systemPropertyVariables>
            <arquillian.launch>wildfly-embedded</arquillian.launch>
            <java.util.logging.manager>org.jboss.logmanager.LogManager</java.util.logging.manager>
        </systemPropertyVariables>
    </configuration>
</plugin>
```

对 WildFly 嵌入式适配器运行测试。

```
mvn clean verfiy -Parq-wildfly-embedded
```

## 从我的 Github 中抓取一份[源代码，然后自己探索。](https://github.com/hantsy/jakartaee9-starter-boilerplate)