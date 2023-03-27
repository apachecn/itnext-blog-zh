# 使用 Cargo maven 插件将 Jakarta EE 9 应用程序部署到 Apache TomEE

> 原文：<https://itnext.io/deploying-jakarta-ee-9-applications-to-apache-tomee-using-cargo-maven-plugin-164644dc9828?source=collection_archive---------8----------------------->

除了[将 Jakarta EE 9 应用程序部署到 Apache TomEE 之外，](/deploying-jakarta-ee-9-applications-to-apache-tomee-e1a9b9399d9b)在本文中使用 Cargo Maven 插件。

![](img/bbe525b66addc9069a161aebd3e74370.png)

我们已经讨论了使用官方 tomee-maven-plugin 对 Apache TomEE 的[部署。使用 tomee-maven-plugin，很容易下载 Apache tomee 发行版的副本，启动 TomEE 服务器，然后将应用程序部署到其上。或者将您的应用程序部署到正在运行的 TomEE 服务器上。官方的 maven 插件在大多数情况下都很棒，但是它缺乏将应用程序部署到具有细粒度配置的本地安装服务器的能力。](https://github.com/hantsy/jakartaee9-starter-boilerplate/blob/master/docs/deploy-tomee.md)

Cargo maven 插件 1.8.3 为 TomEE 9.0 带来了更新，这与 Jakarta EE 9 一致，有几个部署选项。

*   使用独立配置部署到本地 TomEE 服务器
*   使用现有配置部署到本地 TomEE 服务器
*   使用运行时配置部署到正在运行的 TomEE 服务器

# 先决条件

确保您已经安装了以下软件。

*   Java 8 或 Java 11 ( [OpenJDK](https://openjdk.java.net/install/) 或 [AdoptOpenJDK](https://adoptopenjdk.net/installation.html) )
*   [阿帕奇托米(羽)9.0.0-M3](http://tomee.apache.org/download-ng.html)
*   [阿帕奇 Maven](http://maven.apache.org/)
*   了解 [Cargo maven 插件](https://codehaus-cargo.github.io/)的基础。

# 部署到本地 TomEE 服务器

以下配置使用本地安装的服务器，带有**独立**配置。

```
<profile>
    <id>tomee-local</id>
    <properties>
        <tomee.home>${project.build.directory}/apache-tomee-plume-${tomee.version}</tomee.home>
    </properties>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <id>unpack</id>
                        <phase>process-resources</phase>
                        <goals>
                            <goal>unpack</goal>
                        </goals>
                        <configuration>
                            <artifactItems>
                                <artifactItem>
                                    <groupId>org.apache.tomee.jakarta</groupId>
                                    <artifactId>apache-tomee</artifactId>
                                    <version>${tomee.version}</version>
                                    <classifier>plume</classifier>
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
                <groupId>org.codehaus.cargo</groupId>
                <artifactId>cargo-maven2-plugin</artifactId>
                <configuration>
                    <container>
                        <containerId>tomee9x</containerId>
                        <type>installed</type>
                        <home>${tomee.home}</home>
                    </container>
                    <configuration>
                        <type>standalone</type>
                        <properties>
                        </properties>
                    </configuration>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile>
```

在这个配置中，我们使用依赖 maven 插件为你准备一个 TomEE 服务器，当然你也可以使用本地现有的服务器来代替。

在`container`部分，使用了**安装**类型，并指定 *home* 到 TomEE 服务器的**位置。**

在`configuration`部分，我们在这里使用一个**独立**配置。

> *关于 TomEE 属性的更多细节，参见* [*货物 TomEE 9.x 集装箱页面*](https://codehaus-cargo.github.io/cargo/TomEE+9.x.html) *。*

运行以下命令来部署我们的应用程序。

```
mvn clean package cargo:run -Ptomee-local
```

它将执行一系列任务，包括:

*   构建项目并将其打包到目标文件夹中的 war 中。
*   下载 Apache TomEE 的副本，并将文件解压缩到目标文件夹。
*   然后在目标文件夹中创建一个新的**独立**配置，该配置仅用于该应用程序。
*   使用创建的**独立**配置启动 TomEE 服务器。
*   最后将 war 部署到正在运行的服务器上。

> *默认情况下，Apache Tomcat 和 Apache TomEE 不包含多域/实例的配置，但有可能，* [*Google 搜索结果*](https://www.google.com/search?client=firefox-b-d&q=tomcat+multi+instance) *中有很多文章。货物生成* ***单机*** *配置是配置新实例的一个例子。*

如果您想重新使用 TomEE 服务器中的默认配置，请将`configuraiton`部分更改为以下内容。

```
<configuration>
    <type>existing</type>
    <home></home>
</configuration>
```

在这个配置中，将 type 值设置为现有的中的**，并且 *home* 与`container`部分中的 home 相同，因为对于 Glassfish 这样的域实例没有特定的配置。**

要取消应用程序的部署，只需向控制台发送一个`CTRL+C`。

# 部署到正在运行的 TomEE 服务器

部署到正在运行的 TomEE 服务器，尤其是。它位于不同的主机中，使用以下配置。

```
<profile>
    <id>tomee-remote</id>
    <!-- Add `manager-script` role to the tomee user in tomee/conf/tomcat-users.xml -->
    <!-- Run `mvn clean cargo:deploy -Ptomee-remote` to the running TomEE-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.cargo</groupId>
                <artifactId>cargo-maven2-plugin</artifactId>
                <configuration>
                    <container>
                        <containerId>tomee9x</containerId>
                        <type>remote</type>
                    </container>
                    <configuration>
                        <type>runtime</type>
                        <properties>
                            <cargo.remote.username>tomee</cargo.remote.username>
                            <cargo.remote.password>tomee</cargo.remote.password>
                        </properties>
                    </configuration>
                </configuration>
            </plugin>
        </plugins>
    </build>
        </profile>
```

在此配置中，在`container`部分，将类型设置为**远程**，在`configuration`部分将类型设置为`runtime`，并且不要忘记设置用于连接远程服务器的用户名和密码。

与 Apache Tomcat 类似，Cargo 也要求配置的用户拥有一个**管理器-脚本**角色。打开*tomee dir/conf/Tomcat-users . XML*文件，确保在角色中设置了**管理器脚本**。

```
<user username="tomee" password="tomee" roles="tomee-admin,manager-gui,manager-script" />
```

运行以下命令来部署和取消部署应用程序。

```
# deploy applications
mvn clean package cargo:deploy -Ptomee-remote# undeploy
mvn cargo:undeploy -Ptomee-remote
```

**从我的 Github** 中抓取 [**源代码。**](https://github.com/hantsy/jakartaee9-starter-boilerplate)