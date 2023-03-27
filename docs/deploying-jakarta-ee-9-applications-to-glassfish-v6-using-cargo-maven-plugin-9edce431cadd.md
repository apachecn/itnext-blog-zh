# 使用 Cargo maven 插件将 Jakarta EE 9 应用程序部署到 Glassfish v6

> 原文：<https://itnext.io/deploying-jakarta-ee-9-applications-to-glassfish-v6-using-cargo-maven-plugin-9edce431cadd?source=collection_archive---------3----------------------->

从雅加达 EE 邮件订阅来看，我认为雅加达 EE 9 已经为公众准备好了，官方声明将在几天内发布。

![](img/bbe525b66addc9069a161aebd3e74370.png)

在之前的 [Jakarta EE 8 starter 样板文件](https://github.com/hantsy/jakartaee8-starter)中，我已经描述了[如何在 Glassfish v5](https://github.com/hantsy/jakartaee8-starter/blob/master/docs/03run-glassfish-mvn.md) 上部署 Jakarta EE 8 应用程序，在这篇文章中，我将更新它并尝试将 Jakarta EE 9 示例应用程序部署到 Glassfish v6 应用服务器上。

# 先决条件

确保您已经安装了以下软件。

*   Java 8，没错，Glassfish v6 还是只支持 Java 8，下一个 6.1 会重点支持 Java 11。建议使用 Oracle JDK 8 或 AdoptOpenJDK 8。
*   下载并安装 [Glassfish v6](https://github.com/eclipse-ee4j/glassfish/releases) ，当您通过**现有**和远程**运行时**配置部署应用程序时，这是必需的。
*   下载并安装 [Apache Maven](http://maven.apache.org/) 。
*   了解 [Cargo maven 插件](https://codehaus-cargo.github.io/)的基础。

> *在本文中，我们仍然使用货物* `*glassfish5x*` *集装箱来执行部署。在 Glassfish v6 的 1.8.3 版本中，Cargo 将包含一个新的* `*glassfish6x*` *容器。*

# 部署应用程序

将 [Jakarta EE9 starter 样板文件](https://github.com/hantsy/jakartaee9-starter-boilerplate)库克隆到您的本地磁盘中。

打开一个终端并切换到项目根文件夹，执行以下命令来构建 Jakarta EE 9 应用程序并将其部署到 Glassfish v6。

```
mvn clean verify org.codehaus.cargo:cargo-maven2-plugin:1.8.2:run 
-Dcargo.maven.containerId=glassfish5x   
-Dcargo.maven.containerUrl=https://download.eclipse.org/ee4j/glassfish/glassfish-6.0.0-RC2.zip  
-Dcargo.servlet.port=8080
```

该命令将使用 Maven 构建项目，然后调用 cargo maven 插件从指定的 URL 下载 Glassfish v6 的副本，然后尝试为应用程序部署准备新的域配置，然后启动新创建的域，最后部署可部署文件(`target\jakartaee9-starter-boilerplate.war`)。

当部署成功时，您将看到类似如下的信息。

```
[INFO] --- cargo-maven2-plugin:1.8.2:run (default-cli) @ jakartaee9-starter-boilerplate ---
[INFO] [en2.ContainerRunMojo] Resolved container artifact org.codehaus.cargo:cargo-core-container-glassfish:jar:1.8.2 for container glassfish5x
[INFO] [talledLocalContainer] Parsed GlassFish version = [6.0.0.RC2]
[INFO] [talledLocalContainer] GlassFish 6.0.0.RC2 starting...
[INFO] [talledLocalContainer] Using port 4848 for Admin.
[INFO] [talledLocalContainer] Using port 8080 for HTTP Instance.
[INFO] [talledLocalContainer] Using port 7676 for JMS.
[INFO] [talledLocalContainer] Using port 3700 for IIOP.
[INFO] [talledLocalContainer] Using port 8181 for HTTP_SSL.
[INFO] [talledLocalContainer] Using port 3820 for IIOP_SSL.
[INFO] [talledLocalContainer] Using port 3920 for IIOP_MUTUALAUTH.
[INFO] [talledLocalContainer] Using port 8686 for JMX_ADMIN.
[INFO] [talledLocalContainer] Using port 6666 for OSGI_SHELL.
[INFO] [talledLocalContainer] Using port 9009 for JAVA_DEBUGGER.
[INFO] [talledLocalContainer] Distinguished Name of the self-signed X.509 Server Certificate is:
[INFO] [talledLocalContainer] [CN=hantsy-t540p,OU=GlassFish,O=Eclipse.org Foundation Inc,L=Ottawa,ST=Ontario,C=CA]
[INFO] [talledLocalContainer] Distinguished Name of the self-signed X.509 Server Certificate is:
[INFO] [talledLocalContainer] [CN=hantsy-t540p-instance,OU=GlassFish,O=Eclipse.org Foundation Inc,L=Ottawa,ST=Ontario,C=CA]
[INFO] [talledLocalContainer] Domain cargo-domain created.
[INFO] [talledLocalContainer] Domain cargo-domain admin port is 4848.
[INFO] [talledLocalContainer] Domain cargo-domain admin user is "admin".
[INFO] [talledLocalContainer] Command create-domain executed successfully.
[INFO] [talledLocalContainer] Attempting to start cargo-domain.... Please look at the server log for more details.....
[INFO] [talledLocalContainer] remote failure: The jdbc resource [ jdbc/__default ] cannot be deleted as it is required to be configured in the system.
[INFO] [talledLocalContainer] Command delete-jdbc-resource failed.
[INFO] [talledLocalContainer] JDBC Connection pool DerbyPool deleted successfully
[INFO] [talledLocalContainer] Command delete-jdbc-connection-pool executed successfully.
[INFO] [talledLocalContainer] Application deployed with name jakartaee9-starter-boilerplate.
[INFO] [talledLocalContainer] Command deploy executed successfully.
[INFO] [talledLocalContainer] Application deployed with name cargocpc.
[INFO] [talledLocalContainer] Command deploy executed successfully.
[INFO] [talledLocalContainer] GlassFish 6.0.0.RC2 started on port [8080]
[INFO] Press Ctrl-C to stop the container...
```

上述命令相当于项目 *pom.xml* 文件中的以下配置。

```
<properties>
	<cargo.servlet.port>8080</cargo.servlet.port>
	<cargo.zipUrlInstaller.downloadDir>${project.build.directory}/../installs
	</cargo.zipUrlInstaller.downloadDir>
</properties><build>
	<plugins>
		<plugin>
			<groupId>org.codehaus.cargo</groupId>
			<artifactId>cargo-maven2-plugin</artifactId>
			<configuration>
				<container>
					<containerId>glassfish5x</containerId>
					<zipUrlInstaller>
						<url>https://download.eclipse.org/ee4j/glassfish/glassfish-6.0.0-RC2.zip</url>
						<downloadDir>${cargo.zipUrlInstaller.downloadDir}</downloadDir>
					</zipUrlInstaller>
				</container>
				<configuration>
					<home>${project.build.directory}/glassfish6x-home</home>
					<properties>
						<cargo.servlet.port>${cargo.servlet.port}</cargo.servlet.port>
					</properties>
				</configuration>
			</configuration>
		</plugin>
	</plugins>
</build>
```

您可以使用上述配置运行以下命令。

```
mvn clean verify cargo:run
```

这里我们使用`run`目标，它将保持应用服务器运行，直到我们发送一个`CTRL+C`命令来停止它。

如果您已经准备了 Glassfish v6 的副本，您可以重用现有的**Glassfish 和域配置。**

```
<properties>
	<glassfish.home>[ The existing glassfish6 dir]</glassfish.home>
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
					<containerId>glassfish5x</containerId>
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

在上面的代码中，将配置类型设置为**现有的**，并将域名指定为默认的`domain1`，当运行下面的命令时，它将重用现有的`domain1`来部署应用程序。

```
mvn clean verify cargo:run
```

如果您想将应用程序部署到正在运行的 Glassfish v6 服务器(esp。它运行在不同的机器上)，尝试配置一个**运行时**配置，并使用一个**远程**部署器来执行部署。

```
<plugin>
	<groupId>org.codehaus.cargo</groupId>
	<artifactId>cargo-maven2-plugin</artifactId>
	<configuration>
		<container>
			<containerId>glassfish5x</containerId>
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
	<!-- WARNING: deployment client is not available in Glassfish v6.0 -->
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

在上述配置中:

*   `cargo.hostname`是您要部署到的目标服务器
*   `cargo.remote.username`和`cargo.remote.password`是用于部署的管理员账户
*   远程部署者依赖于一个`deployment-client`原型，这里我们使用版本`5.1.0`，Glassfish v6 没有新版本，Jakarta EE 9 中删除了 JSR 88 规范。

对于远程容器，您不能像以前的配置那样控制生命周期的开始和停止，而是使用`deploy`和`undeploy`目标来执行部署和取消部署任务。

```
# deploy applications
mvn clean verify cargo:deploy# undeploy
mvn cargo:undeploy
```

当应用程序部署成功时，使用`curl`验证部署的应用程序是否正在运行。

```
curl [http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting/Hantsy](http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting/Hantsy)
{"message":"Say Hello to Hantsy at 2020-11-14T15:56:10.099"}
```

我已经在 *pom.xml* 中添加了上述不同 Maven 配置文件的配置。查看[源代码](https://github.com/hantsy/jakartaee9-starter-boilerplate/)并探索自己。