# 将 Jakarta EE 9 应用程序部署到 Open Liberty

> 原文：<https://itnext.io/deploying-jakarta-ee-9-applications-to-open-liberty-dac4529f48c6?source=collection_archive---------5----------------------->

![](img/bbe525b66addc9069a161aebd3e74370.png)

在过去的开发迭代中，Open Liberty 已经为 Jakarta EE 9 提供了一个独立的预览版。在我写这篇文章的时候，最新的版本是 21.0.0.1-beta。Open Liberty 遵循每月出版周期，我认为陈旧版本将在未来几个月提供。

进入[下载](https://openliberty.io/downloads)页面，切换到 *Beta* 选项卡，表格中有两个封装选项。

*   雅加达 EE 9 测试版功能
*   所有测试版功能

并且确保你正在下载 Jakarta EE 9 测试版的特性。

# 先决条件

确保您已经安装了以下软件。

*   Java 8 或 Java 11
*   Apache Maven 3.6
*   开放自由 21.0.0.1 测试版

# 手动打开自由号

从我的 github 账户克隆[源代码](https://github.com/hantsy/jakartaee9-starter-boilerplate)，然后构建项目。

```
mvn clean package
```

完成后，在*目标*文件夹中会打包一个*jakartaee 9-starter-boilerplate . war*文件。

进入开放自由文件夹，启动开放自由服务器。

```
# cd wlp\bin# server start // `server run` to show logs in the fontend console
Starting server defaultServer.
CWWKE0953W: This version of Open Liberty is an unsupported early release version.
Server defaultServer started.
```

如果你是第一次运行 server，它会为你创建一个*服务器配置文件*。它将在 *usr/servers* 文件夹中创建一个名为 *defaultServer* 的新文件夹来准备一个`jakartaee-9.0` feature pack 的所有资源，详见*usr/servers/default server/server . XML*文件。

> *注意:服务器概要类似于其他应用服务器中的域概念，比如 Glassfish、WildFly 等。*

要部署我们的应用程序，只需将 war 复制到*WLP/usr/servers/default server/drop ins*文件夹中。

在*usr/servers/default server/logs/messages . log*文件中，您会看到部署进度。

```
[11/24/20, 16:00:07:304 CST] 00000046 com.ibm.ws.app.manager.AppMessageHelper                      I CWWKZ0018I: Starting application jakartaee9-starter-boilerplate.
[11/24/20, 16:00:07:304 CST] 00000046 bm.ws.app.manager.war.internal.WARDeployedAppInfoFactoryImpl I CWWKZ0133I: The jakartaee9-starter-boilerplate application at the D:\appsvr\wlp\usr\servers\defaultServer\dropins\jakartaee9-starter-boilerplate.war location is being expanded to the D:\appsvr\wlp\usr\servers\defaultServer\apps\expanded\jakartaee9-starter-boilerplate.war directory.
[11/24/20, 16:00:09:629 CST] 00000046 org.jboss.weld.Version                                       I WELD-000900: 4.0.0 (Alpha3)
[11/24/20, 16:00:10:709 CST] 00000046 org.jboss.weld.Event                                         I WELD-000411: Observer method [BackedAnnotatedMethod] org.apache.myfaces.config.annotation.CdiAnnotationProviderExtension.processAnnotatedType(@Observes ProcessAnnotatedType<T>) receives events for all annotated types. Consider restricting events using @WithAnnotations or a generic type with bounds.
[11/24/20, 16:00:10:726 CST] 00000046 org.jboss.weld.Event                                         I WELD-000411: Observer method [BackedAnnotatedMethod] org.apache.myfaces.cdi.JsfArtifactProducerExtension.processAnnotatedType(@Observes ProcessAnnotatedType<T>, BeanManager) receives events for all annotated types. Consider restricting events using @WithAnnotations or a generic type with bounds.
[11/24/20, 16:00:11:770 CST] 00000046 com.ibm.ws.webcontainer.osgi.webapp.WebGroup                 I SRVE0169I: Loading Web Module: jakartaee9-starter-boilerplate.
[11/24/20, 16:00:11:786 CST] 00000046 com.ibm.ws.webcontainer                                      I SRVE0250I: Web Module jakartaee9-starter-boilerplate has been bound to default_host.
[11/24/20, 16:00:11:786 CST] 00000046 com.ibm.ws.http.internal.VirtualHostImpl                     A CWWKT0016I: Web application available (default_host): [http://localhost:9080/jakartaee9-starter-boilerplate/](http://localhost:9080/jakartaee9-starter-boilerplate/)
[11/24/20, 16:00:11:786 CST] 0000004e com.ibm.ws.session.WASSessionCore                            I SESN0176I: A new session context will be created for application key default_host/jakartaee9-starter-boilerplate
[11/24/20, 16:00:11:786 CST] 0000004e com.ibm.ws.util                                              I SESN0172I: The session manager is using the Java default SecureRandom implementation for session ID generation.
[11/24/20, 16:00:11:864 CST] 00000046 com.ibm.ws.webcontainer.osgi.mbeans.PluginGenerator          I SRVE9103I: A configuration file for a web server plugin was automatically generated for this server at D:\appsvr\wlp\usr\servers\defaultServer\logs\state\plugin-cfg.xml.
[11/24/20, 16:00:11:896 CST] 0000004e org.apache.myfaces.ee.MyFacesContainerInitializer            I Using org.apache.myfaces.ee.MyFacesContainerInitializer
[11/24/20, 16:00:12:005 CST] 0000004e com.ibm.ws.app.manager.AppMessageHelper                      A CWWKZ0001I: Application jakartaee9-starter-boilerplate started in 4.701 seconds.
[11/24/20, 16:00:12:370 CST] 0000004e org.jboss.resteasy.resteasy_jaxrs.i18n                       I RESTEASY002225: Deploying jakarta.ws.rs.core.Application: class com.example.JaxrsActivator$Proxy$_$$_WeldClientProxy
[11/24/20, 16:00:12:433 CST] 0000004e com.ibm.ws.webcontainer.servlet                              I SRVE0242I: [jakartaee9-starter-boilerplate] [/jakartaee9-starter-boilerplate] [com.example.JaxrsActivator]: Initialization successful.
```

要取消部署应用程序，只需从*WLP/usr/servers/default server/drop ins*文件夹中删除 war 文件。

检查*usr/servers/default server/logs/messages . log*文件，您会看到这样的日志。

```
[11/24/20, 16:04:13:245 CST] 00000066 com.ibm.ws.http.internal.VirtualHostImpl                     A CWWKT0017I: Web application removed (default_host): [http://localhost:9080/jakartaee9-starter-boilerplate/](http://localhost:9080/jakartaee9-starter-boilerplate/)
[11/24/20, 16:04:13:248 CST] 00000066 com.ibm.ws.webcontainer.servlet                              I SRVE0253I: [jakartaee9-starter-boilerplate] [/jakartaee9-starter-boilerplate] [com.example.JaxrsActivator]: Destroy successful.
[11/24/20, 16:04:13:281 CST] 00000066 com.ibm.ws.app.manager.AppMessageHelper                      A CWWKZ0009I: The application jakartaee9-starter-boilerplate has stopped successfully.
[11/24/20, 16:04:13:359 CST] 00000027 com.ibm.ws.webcontainer.osgi.mbeans.PluginGenerator          I SRVE9103I: A configuration file for a web server plugin was automatically generated for this server at D:\appsvr\wlp\usr\servers\defaultServer\logs\state\plugin-cfg.xml.
```

# 使用 Liberty Maven 插件

在项目 *pom.xml* 文件的`build/plugins`节下声明一个`liberty-maven-plugin`配置。

```
<plugin>
    <groupId>io.openliberty.tools</groupId>
    <artifactId>liberty-maven-plugin</artifactId>
    <version>${liberty-maven-plugin.version}</version>
</plugin>
```

当简单地运行`liberty:run`时，它将检索最新的`io.openliberty:openliberty-kernel`并安装在您的项目特定的`src/main/liberty/config/server.xml`文件中定义的所需特性，然后启动服务器并将您的应用程序部署到该服务器。

要使用最新的 Open Liberty Jakarta EE 9 beta 特性包来运行我们的应用程序，请在`liberty-maven-plugin`配置中配置`runtimeArtifact`来替换默认的`openlibety-kernel`。

```
<runtimeArtifact>
    <groupId>io.openliberty.beta</groupId>
    <artifactId>openliberty-jakartaee9</artifactId>
    <version>${liberty.runtime.version}</version>
</runtimeArtifact>
```

在`properties`部分定义了`liberty.runtime.version`属性。

```
<liberty.runtime.version>21.0.0.1-beta</liberty.runtime.version>
```

运行以下命令，将我们的应用程序部署到 Open Liberty 服务器上。

```
mvn clean liberty:run
```

您将在控制台中看到以下消息。

```
[INFO] --- liberty-maven-plugin:3.3.1:run (default-cli) @ jakartaee9-starter-boilerplate ---
[INFO] The runtimeArtifact version 21.0.0.1-beta is overwritten by the liberty.runtime.version value 21.0.0.1-beta.
[INFO] CWWKM2102I: Using artifact based assembly archive : io.openliberty.beta:openliberty-jakartaee9:null:21.0.0.1-beta:zip.
[INFO] CWWKM2102I: Using installDirectory : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp.
[INFO] CWWKM2102I: Using serverName : defaultServer.
[INFO] CWWKM2102I: Using serverDirectory : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer.
[INFO] Running maven-compiler-plugin:compile
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 4 source files to D:\hantsylabs\jakartaee9-starter-boilerplate\target\classes
[INFO] Running maven-resources-plugin:resources
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] Running liberty:create
[INFO] The runtimeArtifact version 21.0.0.1-beta is overwritten by the liberty.runtime.version value 21.0.0.1-beta.
[INFO] CWWKM2102I: Using artifact based assembly archive : io.openliberty.beta:openliberty-jakartaee9:null:21.0.0.1-beta:zip.
[INFO] CWWKM2102I: Using installDirectory : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp.
[INFO] CWWKM2102I: Using serverName : defaultServer.
[INFO] CWWKM2102I: Using serverDirectory : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer.
[INFO] Installing assembly...
[INFO] Expanding: C:\Users\hantsy\.m2\repository\io\openliberty\beta\openliberty-jakartaee9\21.0.0.1-beta\openliberty-jakartaee9-21.0.0.1-beta.zip into D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty
[INFO] CWWKM2143I: Server defaultServer does not exist. Now creating...
[INFO] CWWKM2001I: Invoke command is ["D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\bin\server.bat", create, defaultServer].
[INFO] Server defaultServer created.
[INFO] CWWKM2129I: Server defaultServer has been created at D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer.
[INFO] Copying 1 file to D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer
[INFO] CWWKM2144I: Update server configuration file server.xml from D:\hantsylabs\jakartaee9-starter-boilerplate\src\main\liberty\config\server.xml.
[INFO] Running liberty:install-feature
[INFO] The runtimeArtifact version 21.0.0.1-beta is overwritten by the liberty.runtime.version value 21.0.0.1-beta.
[INFO] CWWKM2102I: Using artifact based assembly archive : io.openliberty.beta:openliberty-jakartaee9:null:21.0.0.1-beta:zip.
[INFO] CWWKM2102I: Using installDirectory : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp.
[INFO] CWWKM2102I: Using serverName : defaultServer.
[INFO] CWWKM2102I: Using serverDirectory : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer.
[WARNING] Features that are not included with the beta runtime cannot be installed. Features that are included with the beta runtime can be enabled by adding them to your server.xml file.
[INFO] Running liberty:deploy
[INFO] The runtimeArtifact version 21.0.0.1-beta is overwritten by the liberty.runtime.version value 21.0.0.1-beta.
[INFO] CWWKM2102I: Using artifact based assembly archive : io.openliberty.beta:openliberty-jakartaee9:null:21.0.0.1-beta:zip.
[INFO] CWWKM2102I: Using installDirectory : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp.
[INFO] CWWKM2102I: Using serverName : defaultServer.
[INFO] CWWKM2102I: Using serverDirectory : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer.
[INFO] Copying 1 file to D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer
[INFO] CWWKM2144I: Update server configuration file server.xml from D:\hantsylabs\jakartaee9-starter-boilerplate\src\main\liberty\config\server.xml.
[INFO] CWWKM2185I: The liberty-maven-plugin configuration parameter "appsDirectory" value defaults to "dropins".
[INFO] CWWKM2160I: Installing application jakartaee9-starter-boilerplate.war.xml.
[INFO] Copying 1 file to D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer
[INFO] CWWKM2144I: Update server configuration file server.xml from D:\hantsylabs\jakartaee9-starter-boilerplate\src\main\liberty\config\server.xml.
[INFO] CWWKM2001I: Invoke command is ["D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\bin\server.bat", run, defaultServer].
[INFO] Launching defaultServer (Open Liberty 21.0.0.1-beta/wlp-1.0.47.cl201220201111-0736) on OpenJDK 64-Bit Server VM, version 11.0.7+10 (en_US)
[INFO] CWWKE0953W: This version of Open Liberty is an unsupported early release version.
[INFO] [AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
[INFO] [WARNING ] CWWKS3103W: There are no users defined for the BasicRegistry configuration of ID com.ibm.ws.security.registry.basic.config[basic].
[INFO] [AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
[INFO] [AUDIT   ] CWPKI0820A: The default keystore has been created using the 'keystore_password' environment variable.
[INFO] [AUDIT   ] CWWKS4104A: LTPA keys created in 1.056 seconds. LTPA key file: D:/hantsylabs/jakartaee9-starter-boilerplate/target/liberty/wlp/usr/servers/defaultServer/resources/security/ltpa.keys
[INFO] [AUDIT   ] CWWKT0016I: Web application available (default_host): [http://localhost:9080/ibm/api/](http://localhost:9080/ibm/api/)
[INFO] [AUDIT   ] CWWKT0016I: Web application available (default_host): [http://localhost:9080/IBMJMXConnectorREST/](http://localhost:9080/IBMJMXConnectorREST/)
[INFO] [AUDIT   ] CWWKT0016I: Web application available (default_host): [http://localhost:9080/jakartaee9-starter-boilerplate/](http://localhost:9080/jakartaee9-starter-boilerplate/)
[INFO] [AUDIT   ] CWWKZ0001I: Application jakartaee9-starter-boilerplate started in 6.360 seconds.
[INFO] [AUDIT   ] CWWKF0012I: The server installed the following features: [appClientSupport-2.0, appSecurity-4.0, beanValidation-3.0, cdi-3.0, concurrent-2.0, connectors-2.0, connectorsInboundSecurity-2.0, distributedMap-1.0, enterpriseBeans-4.0, enterpriseBeansHome-4.0, enterpriseBeansLite-4.0, enterpriseBeansPersistentTimer-4.0, enterpriseBeansRemote-4.0, expressionLanguage-4.0, faces-3.0, jacc-2.0, jakartaee-9.0, jaspic-2.0, jaxb-3.0, jdbc-4.2, jndi-1.0, json-1.0, jsonb-2.0, jsonp-2.0, mail-2.0, managedBeans-2.0, mdb-4.0, messaging-3.0, messagingClient-3.0, messagingSecurity-3.0, messagingServer-3.0, pages-3.0, persistence-3.0, persistenceContainer-3.0, restConnector-2.0, restfulWS-3.0, restfulWSClient-3.0, servlet-5.0, ssl-1.0, webProfile-9.0, websocket-2.0].
[INFO] [AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 52.202 seconds.
[INFO] [AUDIT   ] CWPKI0803A: SSL certificate created in 8.997 seconds. SSL key file: D:/hantsylabs/jakartaee9-starter-boilerplate/target/liberty/wlp/usr/servers/defaultServer/resources/security/key.p12
[INFO] [AUDIT   ] CWWKI0001I: The CORBA name server is now available at corbaloc:iiop:localhost:2809/NameService.
```

> *注意:beta 特性包在 Maven 原型中使用了不同的****groupId****(*`*io.openliberty.beta*` *)。*

要取消部署应用程序并停止服务器，只需向控制台发送一个`CTRL+C`，您将看到以下信息。

```
[INFO] [AUDIT   ] CWWKE1100I: Waiting for up to 30 seconds for the server to quiesce.
[INFO] [AUDIT   ] CWWKT0017I: Web application removed (default_host): [http://localhost:9080/jakartaee9-starter-boilerplate/](http://localhost:9080/jakartaee9-starter-boilerplate/)
[INFO] [AUDIT   ] CWWKT0017I: Web application removed (default_host): [http://localhost:9080/ibm/api/](http://localhost:9080/ibm/api/)
[INFO] [AUDIT   ] CWWKT0017I: Web application removed (default_host): [http://localhost:9080/IBMJMXConnectorREST/](http://localhost:9080/IBMJMXConnectorREST/)
[INFO] [AUDIT   ] CWWKZ0009I: The application jakartaee9-starter-boilerplate has stopped successfully.
[INFO] [AUDIT   ] CWWKI0002I: The CORBA name server is no longer available at corbaloc:iiop:localhost:2809/NameService.
```

您还可以在插件*配置*中指定一个`installDirectory`属性，以使用现有的 Open Liberty 服务器。

在 CI 服务器中，要从头开始准备 Open Liberty 服务器，需要添加一个`dependency:unpack`来直接从 Maven Central 检索 Open Liberty 档案。

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>${maven-dependency-plugin.version}</version>
    <executions>
        <execution>
            <id>default-unpack</id>
            <phase>process-resources</phase>
            <goals>
                <goal>unpack</goal>
            </goals>
            <configuration>
                <artifactItems>
                    <artifactItem>
                        <groupId>io.openliberty.beta</groupId>
                        <artifactId>openliberty-jakartaee9</artifactId>
                        <version>${liberty.runtime.version}</version>
                        <type>zip</type>
                        <overWrite>false</overWrite>
                        <outputDirectory>${project.build.directory}/liberty
                        </outputDirectory>
                    </artifactItem>
                </artifactItems>
            </configuration>
        </execution>
    </executions>
</plugin>
<!-- Enable liberty-maven-plugin -->
<plugin>
    <groupId>io.openliberty.tools</groupId>
    <artifactId>liberty-maven-plugin</artifactId>
    <version>${liberty-maven-plugin.version}</version>
    <configuration>
        <installDirectory>${project.build.directory}/liberty/wlp</installDirectory>
    </configuration>
</plugin>
```

现在运行以下命令，将我们的应用程序部署到 Open Liberty。

```
mvn clean package liberty:start liberty:deploy
```

> *注意:这里我们不使用* `*run*` *目标，因为它会在初始阶段清理* ***目标*** *文件夹，这样会删除下载的 Open Liberty dist。*

当执行`start`和`deploy`目标时，您将看到以下消息。

```
[INFO] --- liberty-maven-plugin:3.3.1:start (default-cli) @ jakartaee9-starter-boilerplate ---
[INFO] CWWKM2102I: Using pre-installed assembly : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp.
[INFO] CWWKM2102I: Using serverName : defaultServer.
[INFO] CWWKM2102I: Using serverDirectory : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer.
[INFO] CWWKM2107I: Installation type is pre-existing; skipping installation.
[INFO] Copying 1 file to D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer
[INFO] CWWKM2144I: Update server configuration file server.xml from D:\hantsylabs\jakartaee9-starter-boilerplate\src\main\liberty\config\server.xml.
[INFO] CWWKM2001I: Invoke command is ["D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\bin\server.bat", start, defaultServer].
[INFO] Starting server defaultServer.
[INFO] CWWKE0953W: This version of Open Liberty is an unsupported early release version.
[INFO] Server defaultServer started.
[INFO] Waiting up to 30 seconds for server confirmation:  CWWKF0011I to be found in D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer\logs\messages.log
[INFO] CWWKM2010I: Searching for CWWKF0011I in D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer\logs\messages.log. This search will timeout after 30 seconds.
[INFO] CWWKM2015I: Match number: 1 is [11/24/20, 17:38:08:783 CST] 00000022 com.ibm.ws.kernel.feature.internal.FeatureManager            A CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 53.584 seconds..
[INFO]
[INFO] --- liberty-maven-plugin:3.3.1:deploy (default-cli) @ jakartaee9-starter-boilerplate ---
[INFO] CWWKM2102I: Using pre-installed assembly : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp.
[INFO] CWWKM2102I: Using serverName : defaultServer.
[INFO] CWWKM2102I: Using serverDirectory : D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer.
[INFO] Copying 1 file to D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer
[INFO] CWWKM2144I: Update server configuration file server.xml from D:\hantsylabs\jakartaee9-starter-boilerplate\src\main\liberty\config\server.xml.
[INFO] CWWKM2185I: The liberty-maven-plugin configuration parameter "appsDirectory" value defaults to "dropins".
[INFO] CWWKM2160I: Installing application jakartaee9-starter-boilerplate.war.xml.
[INFO] CWWKM2010I: Searching for CWWKZ0001I.*jakartaee9-starter-boilerplate in D:\hantsylabs\jakartaee9-starter-boilerplate\target\liberty\wlp\usr\servers\defaultServer\logs\messages.log. This search will timeout after 40 seconds.
[INFO] CWWKM2015I: Match number: 1 is [11/24/20, 17:38:16:858 CST] 00000028 com.ibm.ws.app.manager.AppMessageHelper                      A CWWKZ0001I: Application jakartaee9-starter-boilerplate started in 4.990 seconds..
```

在启动阶段，它将使用您的项目中的配置更新 Open Liberty server 配置(如果存在)。一个示例 *server.xml* 文件是这样的。

```
<?xml version="1.0" encoding="UTF-8"?>
<server description="new server"> <!-- Enable features -->
    <featureManager>
        <feature>jakartaee-9.0</feature>
    </featureManager> <!-- This template enables security. To get the full use of all the capabilities, a keystore and user registry are required. -->

    <!-- For the keystore, default keys are generated and stored in a keystore. To provide the keystore password, generate an 
         encoded password using bin/securityUtility encode and add it below in the password attribute of the keyStore element. 
         Then uncomment the keyStore element. -->
    <!--
    <keyStore password=""/> 
    -->

    <!--For a user registry configuration, configure your user registry. For example, configure a basic user registry using the
        basicRegistry element. Specify your own user name below in the name attribute of the user element. For the password, 
        generate an encoded password using bin/securityUtility encode and add it in the password attribute of the user element. 
        Then uncomment the user element. -->
    <basicRegistry id="basic" realm="BasicRealm"> 
        <!-- <user name="yourUserName" password="" />  --> 
    </basicRegistry>

    <!-- To access this server from a remote client add a host attribute to the following element, e.g. host="*" -->
    <httpEndpoint id="defaultHttpEndpoint"
                  httpPort="9080"
                  httpsPort="9443" />

    <!-- Automatically expand WAR files and EAR files -->
    <applicationManager autoExpand="true"/></server>
```

要取消部署并停止 Open Liberty 服务器，请执行以下命令。

```
mvn liberty:undeploy 
mvn liberty:stop
```

您还可以配置`installDirectory`属性来使用本地 Open Liberty 服务器的位置。

```
<configuration>
     <installDirectory>D:/appsvr/wlp</installDirectory>
     ...
</configuration>
```

`liberty-maven-plugin`不支持将应用部署到正在运行的服务器上，详见[open liberty/ci . maven # 16](https://github.com/OpenLiberty/ci.maven/issues/16)。