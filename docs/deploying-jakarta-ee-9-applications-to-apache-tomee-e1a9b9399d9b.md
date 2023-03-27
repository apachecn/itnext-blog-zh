# 将 Jakarta EE 9 应用程序部署到 Apache TomEE

> 原文：<https://itnext.io/deploying-jakarta-ee-9-applications-to-apache-tomee-e1a9b9399d9b?source=collection_archive---------2----------------------->

## Apache TomEE(又名 Apache Tomcat + Java EE/Jakarta EE)是一个轻量级的开源 Jakarta EE 应用服务器。

![](img/bbe525b66addc9069a161aebd3e74370.png)

目前它在[下载页面](http://tomee.apache.org/download-ng.html)中提供了一个 Jakarta EE 9 预览版，这个版本是由 Eclipse Transformer 工具项目从 TomEE 8 转换而来的[。](https://github.com/apache/tomee-jakarta)

TomEE 9 在下载页面有几个选项，根据您的需求选择一个变体，关于这些变体的详细信息，请前往[功能对比](http://tomee.apache.org/comparison.html)页面。

> *注意:Apache TomEE 是 Web Profile 兼容的，不是完整 Profile 的实现。*

**从我的 Github 中获取** [**源代码**](https://github.com/hantsy/jakartaee9-starter-boilerplate) **。**

# 先决条件

确保您已经安装了以下软件。

*   Java 11 ( [OpenJDK](https://openjdk.java.net/install/) 或 [AdoptOpenJDK](https://adoptopenjdk.net/installation.html) )
*   [阿帕奇 Maven 3.6.3](http://maven.apache.org/download.cgi)
*   [阿帕奇托米(羽)9.0.0-M3](http://tomee.apache.org/download-ng.html)

# 手动部署

作为一个例子，我们选择了 *plume* 版本，它包含了一个 Web Profile 的超级特征集合。

将文件从下载的归档文件解压到本地磁盘。TomEE 文件结构类似于 Apache Tomcat。

要启动 TomEE 服务器，打开终端窗口，进入 TomEE 根文件夹，切换到 *bin* 文件夹，执行以下命令，在单独的窗口中启动 TomEE 服务器。

```
# Windows
startup.bat
# or 
catalina.bat start# Linux like system.
sh ./startup.sh
```

或者通过执行以下命令在同一窗口中运行 TomEE 服务器。

```
# Windows
catalina.bat run# Linux like system.
sh ./catalina.sh run
```

您将在 TomEE 服务器控制台中看到启动进度。

```
Using CATALINA_BASE:   "D:\appsvr\apache-tomee-plume-9.0.0-M3"
Using CATALINA_HOME:   "D:\appsvr\apache-tomee-plume-9.0.0-M3"
Using CATALINA_TMPDIR: "D:\appsvr\apache-tomee-plume-9.0.0-M3\temp"
Using JRE_HOME:        "D:\jdk11"
Using CLASSPATH:       "D:\appsvr\apache-tomee-plume-9.0.0-M3\bin\bootstrap.jar;D:\appsvr\apache-tomee-plume-9.0.0-M3\bin\tomcat-juli.jar"
Using CATALINA_OPTS:   ""
...
```

在项目根文件夹中执行以下命令，将项目构建到*目标*文件夹中的 *war* 包中。

```
mvn clean package
```

要部署应用程序，只需将*target/jakartaee 9-starter-boilerplate . war*复制到*【tomee 安装目录】/webapps* 文件夹。

在控制台窗口中，您将看到部署进度自动开始。

```
29-Nov-2020 15:40:33.673 INFO [Catalina-utility-1] jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke Deploying web application archive [D:\appsvr\apache-tomee-plume-9.0.0-M3\webapps\jakartaee9-starter-boilerplate.war]
29-Nov-2020 15:40:33.673 INFO [Catalina-utility-1] org.apache.tomee.catalina.TomcatWebAppBuilder.init ------------------------- localhost -> /jakartaee9-starter-boilerplate...29-Nov-2020 15:40:37.955 INFO [Catalina-utility-1] org.apache.openejb.server.cxf.rs.CxfRsHttpListener.logEndpoints REST Application: [http://localhost:8080/jakartaee9-starter-boilerplate/api](http://localhost:8080/jakartaee9-starter-boilerplate/api)                 -> com.example.JaxrsActivator@69c1f1fe
29-Nov-2020 15:40:37.955 INFO [Catalina-utility-1] org.apache.openejb.server.cxf.rs.CxfRsHttpListener.logEndpoints      Service URI: [http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting](http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting)        -> Pojo com.example.GreetingResource
29-Nov-2020 15:40:37.955 INFO [Catalina-utility-1] org.apache.openejb.server.cxf.rs.CxfRsHttpListener.logEndpoints               GET [http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting/{name}](http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting/{name}) ->      Response greeting(String)
29-Nov-2020 15:40:38.003 INFO [Catalina-utility-1] jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke Deployment of web application archive [D:\appsvr\apache-tomee-plume-9.0.0-M3\webapps\jakartaee9-starter-boilerplate.war] has finished in [4,330] ms
```

一旦部署成功，使用`curl`命令验证部署结果。

```
curl [http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting/Hantsy](http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting/Hantsy)
{"message":"Say Hello to Hantsy at 2020-11-29T15:44:29.252675900"}
```

要取消部署应用程序，只需从 *[tomee 安装目录]/webapps* 文件夹中删除 **war** 文件。请稍等，您将在 TomEE 控制台中看到取消部署进度。

要停止正在运行的 TomEE 服务器，只需在 TomEE 控制台发送一个`CTRL+C`信号，或者在另一个终端窗口执行 shutdown 命令。

```
# Linux like system.
[tomee install dir]/bin/shutdown # Windows
[tomee install dir]\bin\shutdown.bat
```

# 使用 TomEE Maven 插件

Apache TomEE 提供了一个官方的 maven 插件，供开发者在应用开发阶段部署和取消部署应用到 TomEE 服务器。

在 *pom.xml* 文件中声明`plugins`下的`tomee-maven-plugin`。

```
<plugin>
    <groupId>org.apache.tomee.maven</groupId>
    <artifactId>tomee-maven-plugin</artifactId>
    <version>${tomee-maven-plugin.version}</version>
</plugin>
```

定义一个`tomee-maven-plugin.version`属性来指定要使用的`tomee-maven-plugin`的版本。

```
<tomee-maven-plugin.version>8.0.5</tomee-maven-plugin.version>
```

> *注意:目前* `*tomee-maven-plugin*` *没有为 Jakarta EE 9/TomEE 9 提供预览版，但是稳定版(8.0.5)与最新的 TomEE 9 预览版配合得非常好。*

默认情况下，`tomee-maven-plugin`将检索最新的稳定版本来服务您的应用程序。在`tomee-maven-plugin`的`configuration`中添加以下属性，强制其使用最新的 TomEE 9.0.0-M3。

```
<!-- download and run on a managed Apache TomEE server -->
<tomeeVersion>${tomee.version}</tomeeVersion>
<tomeeArtifactId>apache-tomee</tomeeArtifactId>
<tomeeGroupId>org.apache.tomee.jakarta</tomeeGroupId>
<tomeeClassifier>plume</tomeeClassifier>
```

定义一个`tomeeVersion`来指定 TomEE 版本。

```
<tomee.version>9.0.0-M3</tomee.version>
```

> *注:* `*tomeeGroupId*` *为****org . Apache . TomEE . Jakarta****，与稳定版不同，TomEE 8.0 使用的是 groupId****org . Apache . TomEE****。*

好了，现在运行下面的命令来启动 TomEE 9 服务器并部署应用程序。

```
mvn clean package tomee:run -Ptomee
```

您将在 TomEE 控制台中看到以下信息。

```
[INFO] --- tomee-maven-plugin:8.0.5:run (default-cli) @ jakartaee9-starter-boilerplate ---
[INFO] TomEE was unzipped in 'D:\hantsylabs\jakartaee9-starter-boilerplate\target\apache-tomee'
[INFO] Removed not mandatory default webapps
[INFO] Installed 'D:\hantsylabs\jakartaee9-starter-boilerplate\target\jakartaee9-starter-boilerplate.war' in D:\hantsylabs\jakartaee9-starter-boilerplate\target\apache-tomee\webapps\ja
kartaee9-starter-boilerplate.war
[INFO] TomEE will run in development mode
[INFO] Running 'org.apache.openejb.maven.plugin.run'. Configured TomEE in plugin is localhost:8080 (plugin shutdown port is 8005 and https port is null)
// ...
29-Nov-2020 14:50:20.179 INFO [main] jdk.internal.reflect.NativeMethodAccessorImpl.invoke Server version name:   Apache Tomcat (TomEE)/9.0.39 (8.0.5)
29-Nov-2020 14:50:20.179 INFO [main] jdk.internal.reflect.NativeMethodAccessorImpl.invoke Server built:          Oct 6 2020 14:11:46 UTC
29-Nov-2020 14:50:20.179 INFO [main] jdk.internal.reflect.NativeMethodAccessorImpl.invoke Server version number: 9.0.39.0
29-Nov-2020 14:50:20.179 INFO [main] jdk.internal.reflect.NativeMethodAccessorImpl.invoke OS Name:               Windows 10
29-Nov-2020 14:50:20.179 INFO [main] jdk.internal.reflect.NativeMethodAccessorImpl.invoke OS Version:            10.0
29-Nov-2020 14:50:20.179 INFO [main] jdk.internal.reflect.NativeMethodAccessorImpl.invoke Architecture:          amd64
29-Nov-2020 14:50:20.179 INFO [main] jdk.internal.reflect.NativeMethodAccessorImpl.invoke Java Home:             D:\jdk11
29-Nov-2020 14:50:20.194 INFO [main] jdk.internal.reflect.NativeMethodAccessorImpl.invoke JVM Version:           11.0.9.1+1
29-Nov-2020 14:50:20.194 INFO [main] jdk.internal.reflect.NativeMethodAccessorImpl.invoke JVM Vendor:            AdoptOpenJDK
29-Nov-2020 14:50:20.194 INFO [main] jdk.internal.reflect.NativeMethodAccessorImpl.invoke CATALINA_BASE:         D:\hantsylabs\jakartaee9-starter-boilerplate\target\apache-tomee
29-Nov-2020 14:50:20.194 INFO [main] jdk.internal.reflect.NativeMethodAccessorImpl.invoke CATALINA_HOME:         D:\hantsylabs\jakartaee9-starter-boilerplate\target\apache-tomee//...29-Nov-2020 14:50:28.371 INFO [main] org.apache.openejb.server.cxf.rs.CxfRsHttpListener.logEndpoints REST Application: [http://localhost:8080/jakartaee9-starter-boilerplate/api](http://localhost:8080/jakartaee9-starter-boilerplate/api)
        -> com.example.JaxrsActivator@2ab8589a
29-Nov-2020 14:50:28.386 INFO [main] org.apache.openejb.server.cxf.rs.CxfRsHttpListener.logEndpoints      Service URI: [http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting](http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting)
        -> Pojo com.example.GreetingResource
29-Nov-2020 14:50:28.386 INFO [main] org.apache.openejb.server.cxf.rs.CxfRsHttpListener.logEndpoints               GET [http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting](http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting)
/{name} ->      Response greeting(String)
...
29-Nov-2020 14:50:29.325 WARNING [main] org.apache.catalina.util.SessionIdGeneratorBase.createSecureRandom Creation of SecureRandom instance for session ID generation using [SHA1PRNG]
took [829] milliseconds.
29-Nov-2020 14:50:29.341 INFO [main] jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke Starting ProtocolHandler ["http-nio-8080"]
29-Nov-2020 14:50:29.348 INFO [main] jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke Server startup in [6177] milliseconds
```

您还可以使用`tomee-maven-plugin`将应用程序部署到正在运行的 TomEE 服务器，尤其是 esp。运行在远程服务器上。

打开*[tomee install dir]/conf/Tomcat-users . XML*文件，取消注释以下配置。

```
<!-- Activate those lines to get access to TomEE GUI if added (tomee-webaccess) --><role rolename="tomee-admin" />
<user username="tomee" password="tomee" roles="tomee-admin,manager-gui" />
```

打开*【tomee 安装目录】/conf/system.properties* 文件，取消注释并修改以下项目。

```
tomee.remote.support = true
tomee.serialization.class.blacklist = -
tomee.serialization.class.whitelist = com.example.
openejb.system.apps = true
openejb.servicemanager.enabled = true
```

让我们返回到项目 *pom.xml* 文件，用以下内容替换之前的`tomee-maven-plugin`配置。

```
<configuration>
    <tomeeHost>localhost</tomeeHost>
    <user>tomee</user>
    <password>tomee</password>
    <path>${project.build.directory}/${project.build.finalName}.war</path>
</configuration>
```

确保 TomEE 服务器正在运行，在项目根文件夹中运行以下命令。

```
mvn clean package tomee:deploy -Ptomee
```

您可以在 TomEE 控制台中查看部署进度信息。

```
...
29-Nov-2020 16:56:23.550 INFO [http-nio-8080-exec-8] org.apache.openejb.server.cxf.rs.CxfRsHttpListener.logEndpoints REST Application: [http://localhost:8080/jakartaee9-starter-boilerplate/api](http://localhost:8080/jakartaee9-starter-boilerplate/api)                 -> com.example.JaxrsActivator@affb16f
29-Nov-2020 16:56:23.550 INFO [http-nio-8080-exec-8] org.apache.openejb.server.cxf.rs.CxfRsHttpListener.logEndpoints      Service URI: [http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting](http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting)        -> Pojo com.example.GreetingResource
29-Nov-2020 16:56:23.550 INFO [http-nio-8080-exec-8] org.apache.openejb.server.cxf.rs.CxfRsHttpListener.logEndpoints               GET [http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting/{name}](http://localhost:8080/jakartaee9-starter-boilerplate/api/greeting/{name}) ->      Response greeting(String)
```

要取消部署应用程序，请执行以下命令。

```
mvn tomee:undeploy -Ptomee
```