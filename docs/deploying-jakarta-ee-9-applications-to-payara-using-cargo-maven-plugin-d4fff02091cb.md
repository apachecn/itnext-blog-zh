# 使用 Cargo maven 插件将 Jakarta EE 9 应用程序部署到 Payara

> 原文：<https://itnext.io/deploying-jakarta-ee-9-applications-to-payara-using-cargo-maven-plugin-d4fff02091cb?source=collection_archive---------0----------------------->

Payara 社区 5.2020.5 引入了技术预览功能，可以在 Payara 服务器和 Micro 上运行 Jakarta EE 9，更多详情请参见[发布说明](https://docs.payara.fish/community/docs/5.2020.6/release-notes/release-notes-2020-5.html#_run_jakarta_ee_9_applications_in_tech_preview)。

![](img/bbe525b66addc9069a161aebd3e74370.png)

最初，Payara 是开源 Glassfish 的一个分支，但是它包含了一系列 Glassfish 中没有的新特性。

*   与现有的 Glassfish 相比，有许多改进和更快的错误修复
*   Java 11(或更高版本)支持
*   内置微配置文件支持，可用于云原生应用
*   许多第三方服务集成。
*   全面的文档和技术指南
*   为付费企业用户提供商业支持。

Payara 社区也是开源的，对于开发者来说，你可以用它作为 Glassfish 的替代来获得更好的开发体验。

# 先决条件

确保您已经安装了以下软件。

*   Java 8 或 Java 11 ( [OpenJDK](https://openjdk.java.net/install/) 或 [AdoptOpenJDK](https://adoptopenjdk.net/installation.html) )
*   [Payara 社区 5.2020.6](https://www.payara.fish/downloads/payara-platform-community-edition/)
*   [阿帕奇 Maven](http://maven.apache.org/)
*   了解 [Cargo maven 插件](https://codehaus-cargo.github.io/)的基础。

# 部署到本地 Payara 服务器

Cargo maven 插件为 payara 服务器提供了一个单独的 **payara** 容器。

假设您已经下载了一份 [Payara Community dist](https://www.payara.fish/downloads/payara-platform-community-edition/) ，并将文件解压到您的本地磁盘中。

在 *pom.xml* 中添加以下配置，以使用现有的 Payara 域配置在本地 Payara 服务器上运行 Jakarta EE 9 应用程序。

```
<properties>
	<glassfish.home>[ Payara install dir]</glassfish.home>
	<glassfish.domainDir>${glassfish.home}/glassfish/domains</glassfish.domainDir>
	<glassfish.domainName>domain1</glassfish.domainName>
</properties>
<build>
	<plugins>
		<plugin>
			<groupId>org.codehaus.cargo</groupId>
			<artifactId>cargo-maven2-plugin</artifactId>
			<configuration>
				<container>
					<containerId>payara</containerId>
					<type>installed</type>
					<home>${glassfish.home}</home>
				</container>
				<configuration>
					<type>existing</type>
					<home>${glassfish.domainDir}</home>
					<properties>
						<cargo.glassfish.domain.name>${glassfish.domainName}</cargo.glassfish.domain.name>
						<cargo.remote.password></cargo.remote.password>
					</properties>
				</configuration>
			</configuration>
		</plugin>
	</plugins>
</build>
```

上面的配置几乎与我们在[之前的 Glassfish 帖子](https://github.com/hantsy/jakartaee9-starter-boilerplate/blob/master/docs/docs/deploy-cargo.md)中讨论的配置相同，但是我们在这里使用了 containerId **payara** 。

打开终端，执行以下命令在 Payara 服务器上运行您的应用程序。

```
mvn clean package cargo:run -Ppayara-local
```

上述命令将执行一系列任务。

*   构建项目，并将其打包到*目标*文件夹中的 war 档案中
*   使用现有的*域 1* 配置启动 Payara 服务器
*   将打包的 war 部署到正在运行的服务器

如果您希望每次都使用刷新域配置来运行您的应用程序，请尝试将配置`type`设置为**独立**。将服务器*配置*部分更改为以下内容。

```
<configuration>
    <type>standalone</type>
    <properties>
        <cargo.remote.password></cargo.remote.password>
    </properties>
</configuration>
```

与现有的配置略有不同，它将在*目标*文件夹中创建一个新的域配置，而不是 Payara 服务器中的默认*域 1* 。

在 CI 服务中，比如 Github actions，我想使用官方的`maven-dependency-plugin`来准备 Payara 服务器的刷新副本。

```
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
```

将`glassfish.home`属性更改为以下内容。

```
<glassfish.home>${project.build.directory}/payara5</glassfish.home>
```

要停止服务器，只需向正在运行的控制台发送一个`CTRL+C`来停止它。

# 部署到正在运行的 Payara 服务器

使用远程部署器和运行时配置，您可以使用 Cargo maven 插件将应用程序部署到正在运行的 Payara，esp。Payara 服务器位于远程服务器中。

```
<plugin>
	<groupId>org.codehaus.cargo</groupId>
	<artifactId>cargo-maven2-plugin</artifactId>
	<configuration>
		<container>
			<containerId>payara</containerId>
			<type>remote</type>
		</container>
		<configuration>
			<type>runtime</type>
			<properties>
				<cargo.remote.username>admin</cargo.remote.username>
				<cargo.remote.password></cargo.remote.password>
				<cargo.glassfish.admin.port>4848</cargo.glassfish.admin.port>
				<cargo.hostname>localhost</cargo.hostname>
			</properties>
		</configuration>
	</configuration>
	<!-- provides JSR88 client API to deploy on Glassfish/Payara Server -->
	<dependencies>
		<dependency>
			<groupId>org.glassfish.main.deployment</groupId>
			<artifactId>deployment-client</artifactId>
			<version>5.1.0</version>
			<!--<version>${glassfish.version}</version>-->
		</dependency>
	</dependencies>
</plugin>
```

上述配置与 [Glassfish 文档](https://github.com/hantsy/jakartaee9-starter-boilerplate/blob/master/docs/docs/deploy-cargo.md)中的配置几乎相同。

对于远程服务器，您不能启动和停止 Payara 服务器。在部署应用程序之前，请确保它正在运行并且可以从您的本地环境访问。

使用`deploy`和`undeploy`目标执行部署和取消部署任务。

```
# deploy applications
mvn clean package cargo:deploy -Ppayara-remote# undeploy
mvn cargo:undeploy -Ppayara-remote
```

# 验证部署的应用程序

当应用程序成功部署后，使用`curl`来验证部署的应用程序是否。

```
curl [http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting/Hantsy](http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting/Hantsy)
{"message":"Say Hello to Hantsy at 2020-11-14T15:56:10.099"}
```

> *注:事实上，您可以使用* `*glassfish5x*` *containerId 将 Jakarta EE 9 应用程序部署到 Payara 服务器，反过来，我认为使用* `*payara*` *containerId 将应用程序部署到 Glassfish v6 也是可以的。*

从我的 github 中抓取[源代码](https://github.com/hantsy/jakartaee9-starter-boilerplate/)。