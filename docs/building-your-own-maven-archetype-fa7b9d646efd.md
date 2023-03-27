# 构建您自己的 Maven 原型

> 原文：<https://itnext.io/building-your-own-maven-archetype-fa7b9d646efd?source=collection_archive---------5----------------------->

Github 很容易与他人共享代码，例如我创建了[jakartaee 9-starter-boilerplate](https://github.com/hantsy/jakartaee9-starter-boilerplate)作为 jakartaee 开发人员的项目模板。对于那些熟悉 Github 的人来说，直接分叉或克隆这个项目就可以很容易地开始他们的新项目。但是很明显，对于一个普通的 Jakarta EE 应用程序，你不需要所有应用服务器的配置，比如 Glassfish/Payara、WildFly、OpenLiberty、Apache TomEE 等等。对于大多数 Java 开发人员，尤其是。Maven 用户，简单干净的 Maven 原型仍然是生成新项目框架的首选，您可以在以后添加所需的配置。

![](img/24cef39b9a00eb0f9fe57e917ca07494.png)

在过去的几年里，我自己也创建了一些 maven 原型，比如 toys，但是我从来没有通过 Maven Central repository 公开过它们。在这篇文章中，我将重放为 Jakarta EE 9 创建 [Maven 原型的步骤，以及随后将它发布到 Maven 中央存储库并向全世界公开的步骤。](https://github.com/hantsy/maven-archetype-jakartaee9)

## 一个更好的格式版本发布在 dev.to 上，[https://dev . to/hantsy _ 26/building-your-own-maven-archetype-gm8](https://dev.to/hantsy_26/building-your-own-maven-archetype-gm8)

# 生成原型项目框架

Maven **原型**是用于在 Maven 构建系统中生成新项目的 Maven 特定模板。Maven 本身提供了一个官方原型(`maven-archetype-archetype`)，供你构建自己的 Maven 原型。

[官方原型介绍页面](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html)列出了所有官方原型。

运行以下命令，以交互模式生成原型项目。

```
mvn archetype:generate
```

当您按下 *Enter* 键时，需要几秒钟来列出所有可用的原型。每个原型都有一个数字。

键入`maven-archetype-archetype`的编号，然后它会要求您填写您的项目的`groupId`、`archetypeId`、`package`和`version`。在所有字段都按预期填充后，它将在几秒钟内生成项目。

或者，您可以在单个命令中生成项目，也称为批处理模式。

```
mvn archetype:generate -B \
-DarchetypeGroupId=org.apache.maven.archetypes \
-DarchetypeArtifactId=maven-archetype-archetype \
-DarchetypeVersion=1.4 \
-DgroupId=io.github.hantsy \
-DartifactId=maven-archetype-jakartaee9 \
-Dversion=1.0-SNAPSHOT
```

注意这里我们使用命令行中的`-B`参数来跳过交互式问卷调查步骤，并以**批处理**模式运行该命令。

更多关于 Maven 原型和`maven-archetype-plugin`的信息，请查看 [Maven 原型](http://maven.apache.org/archetype/)。

# 探索项目结构

将项目导入到您的 ide 中，如 Apache NetBeans、IntelliJ IDEA 等。

您将看到下面的项目文件结构。

```
├── pom.xml
└── src
    ├── main
    │   └── resources
    │       ├── archetype-resources
    │       │   ├── pom.xml
    │       │   └── src
    │       │       ├── main
    │       │       │   └── java
    │       │       │       └── App.java
    │       │       └── test
    │       │           └── java
    │       │               └── AppTest.java
    │       └── META-INF
    │           └── maven
    │               └── archetype-metadata.xml
    └── test
        └── resources
            └── projects
                └── it-basic
                    ├── archetype.properties
                    └── goal.txt
```

其结构类似于一般的 Maven 项目，但有一些不同。

1.  *src/main/resources/architect-resources*文件夹存储了用于从这个原型生成项目的所有模板资源。
2.  *src/main/resources/META-INF/maven-archetype-metadata . XML*是原型描述文件，包括属性、文件集等。
3.  *src/test/resources/projects*包括测试这个原型的资源，包括应用于生成新项目的属性和在主机`integration-test`阶段运行的目标。

# 准备原型资源

为了简化工作，我重用了来自[jakartaee 9-starter-boilerplate](https://github.com/hantsy/jakartaee9-starter-boilerplate)的样本代码。

1.  将`GreetingMessage`、`GreetingService`、`GreetingService`和`JaxrsActivator`复制到*src/main/resources/architect-resources/src/main/Java*文件夹中。将包声明更改为以下内容。

```
package $package; 
…
```

2.与上一步类似，将`GreetingResourceTest`和`GreetingServiceTest`复制到*src/main/resources/architect-resources/src/test/Java*文件夹中。请记住更改包声明。

3.将空的`beans.xml`文件添加到*src/main/resources/architect-resources/src/main/resources/META-INF/*文件夹中。

```
<beans xmlns=”https://jakarta.ee/xml/ns/jakartaee" xmlns:xsi=”http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation=”https://jakarta.ee/xml/ns/jakartaee https://jakarta.ee/xml/ns/jakartaee/beans_3_0.xsd" version=”3.0"> </beans>
```

4.将一个简单的`arquillian.xml`添加到*src/main/resources/architect-resources/src/test/resources*文件夹中。

```
<arquillian xmlns:xsi=”http://www.w3.org/2001/XMLSchema-instance" xmlns=”http://jboss.org/schema/arquillian" xsi:schemaLocation=”http://jboss.org/schema/arquillian http://jboss.org/schema/arquillian/arquillian_1_0.xsd"> <defaultProtocol type=”Servlet 5.0"/> <engine> 
  <property name=”deploymentExportPath”>target/</property> </engine> 
<container qualifier=”glassfish” default=”true”> 
<configuration> 
  <property name=”adminHost”>localhost</property>
  <property name=”adminPort”>4848</property> 
  <property name=”adminUser”>admin</property> <! — if https is enabled via `asadmin enable-secure-admin` on a remote server →
  <! — <property name=”adminHttps”>true</property> →
  <! — if admin password is changed via `asadmin change-admin-password` → 
  <! — <property name=”adminPassword”>adminadmin</property> →
  <! — default is empty → <property name=”adminPassword”></property> 
</configuration> 
</container> 
</arquillian>
```

5.将`pom.xml`模板(*src/main/resources/archetype-resources/POM . XML*)更改为以下内容。

```
<project  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>${groupId}</groupId>
  <artifactId>${artifactId}</artifactId>
  <version>${version}</version>
  <packaging>war</packaging>

  <name>${artifactId}</name>
  <description>A Jakarta EE starter boilerplate for Jakarta EE 9</description>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>

    <!-- Official Maven Plugins -->
    <maven-compiler-plugin.version>3.8.1</maven-compiler-plugin.version>
    <maven-war-plugin.version>3.3.1</maven-war-plugin.version>
    <maven-dependency-plugin.version>3.1.2</maven-dependency-plugin.version>
    <maven-surefire-plugin.version>3.0.0-M5</maven-surefire-plugin.version>
    <maven-failsafe-plugin.version>3.0.0-M5</maven-failsafe-plugin.version>
    <maven-surefire-report-plugin.version>3.0.0-M5</maven-surefire-report-plugin.version>

    <!-- Cargo maven plugin -->
    <!-- Since 1.9.0, cargo-maven3-plugin is the default for cargo prefix. -->
    <cargo-maven3-plugin.version>1.9.0</cargo-maven3-plugin.version>

    <!-- Jakarta EE API -->
    <jakartaee-api.version>9.0.0</jakartaee-api.version>

    <!-- Arquillian BOM -->
    <arquillian-bom.version>1.7.0.Alpha9</arquillian-bom.version>
    <junit-jupiter.version>5.7.1</junit-jupiter.version>

    <!-- Glassfish server -->
    <glassfish.version>6.0.0</glassfish.version>
    <arquillian-glassfish6.version>1.0.0.Alpha1</arquillian-glassfish6.version>
    <jersey.version>3.0.1</jersey.version>

    <!-- by default skip tests -->
    <skipTests>true</skipTests>
  </properties>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>jakarta.platform</groupId>
        <artifactId>jakarta.jakartaee-api</artifactId>
        <version>${jakartaee-api.version}</version>
        <scope>provided</scope>
      </dependency>
      <dependency>
        <groupId>org.jboss.arquillian</groupId>
        <artifactId>arquillian-bom</artifactId>
        <version>${arquillian-bom.version}</version>
        <scope>import</scope>
        <type>pom</type>
      </dependency>
      <dependency>
        <groupId>org.junit</groupId>
        <artifactId>junit-bom</artifactId>
        <version>${junit-jupiter.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
      <groupId>jakarta.platform</groupId>
      <artifactId>jakarta.jakartaee-api</artifactId>
    </dependency>
    <dependency>
      <groupId>org.jboss.arquillian.junit5</groupId>
      <artifactId>arquillian-junit5-container</artifactId>
      <scope>test</scope>
    </dependency>
    <!-- see: https://github.com/arquillian/arquillian-core/issues/248 -->
    <!-- and https://github.com/arquillian/arquillian-core/pull/246/files -->
    <dependency>
      <groupId>org.jboss.arquillian.protocol</groupId>
      <artifactId>arquillian-protocol-servlet-jakarta</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <finalName>${project.artifactId}</finalName>
    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>org.codehaus.cargo</groupId>
          <artifactId>cargo-maven3-plugin</artifactId>
          <version>${cargo-maven3-plugin.version}</version>
        </plugin>
      </plugins>
    </pluginManagement>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>${maven-compiler-plugin.version}</version>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
        <version>${maven-war-plugin.version}</version>
      </plugin>
    </plugins>
  </build>
  <profiles>
    <profile>
      <id>glassfish</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <build>
        <plugins>
          <plugin>
            <groupId>org.codehaus.cargo</groupId>
            <artifactId>cargo-maven3-plugin</artifactId>
            <configuration>
              <container>
                <containerId>glassfish6x</containerId>
                <artifactInstaller>
                  <groupId>org.glassfish.main.distributions</groupId>
                  <artifactId>glassfish</artifactId>
                  <version>${glassfish.version}</version>
                </artifactInstaller>
              </container>
              <configuration>
                <!-- the configuration used to deploy -->
                <home>${project.build.directory}/glassfish6x-home</home>
                <properties>
                  <cargo.remote.password></cargo.remote.password>
                </properties>
              </configuration>
            </configuration>
          </plugin>
        </plugins>
      </build>

      <dependencies>
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
        <dependency>
          <groupId>org.jboss.arquillian.container</groupId>
          <artifactId>arquillian-glassfish-remote-6</artifactId>
          <version>${arquillian-glassfish6.version}</version>
          <scope>test</scope>
        </dependency>
      </dependencies>

    </profile>
  </profiles>
</project> 
```

注意，这个 xml 中的`groupId`、`artifactId`和`version`元素的值指的是一个占位符，它可以在生成阶段被截取。

6.将原型描述符(*src/main/resources/META-INF/maven/archetype-metadata . XML*)更改为以下内容。

```
<archetype-descriptor xmlns=”http://maven.apache.org/plugins/maven-archetype-plugin/archetype-descriptor/1.0.0" xmlns:xsi=”http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation=”http://maven.apache.org/plugins/maven-archetype-plugin/archetype-descriptor/1.0.0 http://maven.apache.org/xsd/archetype-descriptor-1.0.0.xsd" name=”${artifactId}”> <fileSets> 
<fileSet filtered=”true” packaged=”true”>
 <directory>src/main/java</directory> 
</fileSet> 
<fileSet> <directory>src/main/resources</directory> 
</fileSet> 
<fileSet filtered=”true” packaged=”true”> 
<directory>src/test/java</directory> 
</fileSet> 
<fileSet> 
<directory>src/test/resources</directory> 
</fileSet> 
</fileSets> 
</archetype-descriptor>
```

在上面的代码中，当`filtered`属性设置为`true`时，它将替换占位符(如`$package`等)。)您已经在模板文件中设置了执行`mvn archetype:generate`生成项目时的参数。当`packaged`属性设置为`true`时，会将生成的文件移动到相应的包中。

为了测试原型资源，在*src/test/resources/projects/it-basic*中准备了一个*prototype . properties*和一个 *goal.txt* 文件。在*集成-测试*阶段，它将使用原型资源生成一个测试项目，并使用*原型.属性*中定义的属性替换占位符，并针对生成的项目执行`goal.txt`中的目标。

我创建了一个 [Github Actions 工作流](https://github.com/hantsy/maven-archetype-jakartaee9/blob/master/.github/workflows/it-with-arq-glassfish-remote.yml)来验证生成的项目。

# 发布到 Maven 中央存储库

> *注意，将您的包发布到 Maven 中央存储库的步骤有点繁琐，请耐心等待。*

一旦你成功地构建了 maven 原型项目，下一步就是将它发布到 Maven 中央存储库并公开，在 Google 结果中有一些有用的指南。我建议你从下面的列表中选择一个或多个，并首先阅读它。

*   [将 JAR 发布到中央 Maven 仓库](http://tutorials.jenkov.com/maven/publish-to-central-maven-repository.html)
*   [如何将您的工件发布到 Maven Central](https://dzone.com/articles/publish-your-artifacts-to-maven-central)
*   [通过 5 个简单的步骤创建你自己的 Maven 原型](https://rieckpil.de/create-your-own-maven-archetype-in-5-simple-steps/)

1.  在 [Sonatype 问题跟踪器](https://issues.sonatype.org)中注册一个账户。
2.  在[社区支持](https://issues.sonatype.org/projects/OSSRH)上提交一个新问题，以提交您发布工件的请求。查看[我为](https://issues.sonatype.org/browse/OSSRH-66424)[hantsy/maven-archetype-jakartaee9](https://github.com/hantsy/maven-archetype-jakartaee9)创建的问题#OSSRH-66424 作为例子。
3.  根据问题的响应，在 Github 上创建一个新的空 repo(名称是问题 id ),用于身份验证需求。例如 [OSSRH-66424](https://github.com/hantsy/OSSRH-66424)
4.  生成 GPG 密钥并发送到密钥服务器。

在 Windows 10 系统下，你必须先安装 GPG 工具。如果您使用的是 [Chocolatey](https://chocolatey.org/) ，只需运行以下命令来安装 gpg4win。

```
choco install gpg4win
```

刷新环境。

```
refreshenv
```

然后生成 GPG 键。

```
# gpg — full-gen-key 
gpg (GnuPG) 2.2.27; Copyright © 2021 g10 Code GmbH This is free software: you are free to change and redistribute it. There is NO WARRANTY, to the extent permitted by law. 
Please select what kind of key you want: 
(1) RSA and RSA (default) 
(2) DSA and Elgamal 
(3) DSA (sign only) 
(4) RSA (sign only) 
(14) Existing key from card Your selection? 1
 RSA keys may be between 1024 and 4096 bits long. 
What keysize do you want? (3072) 
Requested keysize is 3072 bits Please specify how long the key should be valid. 0 = key does not expire <n> = key expires in n days <n>w = key expires in n weeks <n>m = key expires in n months <n>y = key expires in n years Key is valid for? (0) Key does not expire at all Is this correct? (y/N) y 
GnuPG needs to construct a user ID to identify your key. 
Real name: Hantsy Bai 
Email address: hantsy@gmail.com 
Comment: 
You selected this USER-ID: “Hantsy Bai <hantsy@gmail.com>” Change (N)ame, ©omment, (E)mail or (O)kay/(Q)uit? O We need to generate a lot of random bytes. It is a good idea to perform some other action (type on the keyboard, move the mouse, utilize the disks) during the prime generation; this gives the random number generator a better chance to gain enough entropy. gpg: AllowSetForegroundWindow(9152) failed: Access is denied. We need to generate a lot of random bytes. It is a good idea to perform some other action (type on the keyboard, move the mouse, utilize the disks) during the prime generation; this gives the random number generator a better chance to gain enough entropy. gpg: C:/Users/hantsy/AppData/Roaming/gnupg/trustdb.gpg: trustdb created gpg: key E5D9B4272348ADAE marked as ultimately trusted gpg: directory ‘C:/Users/hantsy/AppData/Roaming/gnupg/openpgp-revocs.d’ created gpg: revocation certificate stored as ‘C:/Users/hantsy/AppData/Roaming/gnupg/openpgp-revocs.d\03B4C55DDA964D942C756D79E5D9B4272348ADAE.rev’ public and secret key created and signed. pub rsa3072 2021–03–30 [SC] 03B4C55DDA964D942C756D79E5D9B4272348ADAE uid Hantsy Bai <hantsy@gmail.com> 
sub rsa3072 2021–03–30 [E]
```

将密钥发送到公钥服务器。

```
# gpg — send-keys 03B4C55DDA964D942C756D79E5D9B4272348ADAE gpg: sending key E5D9B4272348ADAE to hkps://hkps.pool.sks-keyservers.net
```

5.按照使用 Apache Maven 将[部署到 OSSRH 的步骤来发布您的工件。](https://central.sonatype.org/pages/apache-maven.html)

> *这里我跳过详细步骤，请仔细阅读 Synatype 的* [*文章*](https://central.sonatype.org/pages/apache-maven.html) *。*

6.然后回到[问题#OSSRH-66424](https://issues.sonatype.org/browse/OSSRH-66424) ，评论并通知版主将您的工件发布到 Maven 中央存储库。一两个小时后，您的工件将出现在 [Maven Central Repository 搜索索引页面](https://search.maven.org/search?q=io.github.hantsy)上。

# 发布到 Github 包

[配置 Apache Maven 用于 Github 包](https://docs.github.com/en/packages/guides/configuring-apache-maven-for-use-with-github-packages)页面提供了[将包](https://docs.github.com/en/packages/guides/configuring-apache-maven-for-use-with-github-packages#publishing-a-package)发布到 GitHub 包的详细步骤。

1.  按照[认证 Github 包](https://docs.github.com/en/packages/guides/configuring-apache-maven-for-use-with-github-packages#authenticating-to-github-packages)的步骤，在您的 *~/.m2/settings.xml* 文件中设置 Github 认证信息。
2.  部署到 Github 包。

*   在我们的例子中，我们已经设置了`distributionManagement`来使用 Maven 中央存储库。请改用以下命令。

```
mvn clean package deploy -DskipTests -DaltDeploymentRepository=github::https://maven.pkg.github.com/hantsy/maven-archetype-jakartaee9
```

*   默认情况下，Github 包不允许任何人未经认证就访问您的存储库。在[咨询了 Github 社区](https://github.community/t/how-to-make-github-packages-to-the-public/171321)之后，有一个解决方案可以很好地解决这个问题，让公众可以访问你的库。
*   生成一个范围为`packages:read`的个人访问令牌。
*   运行以下命令来生成存储库配置。

```
# docker run ghcr.io/jcansdale/gpr encode <your PAT>A NuGet `nuget.config` file: 
<packageSourceCredentials> 
<github> 
<add key=”Username” value=”PublicToken” /> 
<add key=”ClearTextPassword” value=”&#48;28d0c8f723084e499eb87a6fe173b3af295d618" /> 
</github> 
</packageSourceCredentials> A Maven `pom.xml` file:
 <repositories> 
<repository> 
<id>github-public</id> <url>https://public:&#48;28d0c8f723084e499eb87a6fe173b3af295d618@maven.pkg.github.com/<OWNER>/*</url> 
</repository> 
</repositories> An npm `.npmrc` file: 
@OWNER:registry=https://npm.pkg.github.com //npm.pkg.github.com/:_authToken=”\u003028d0c8f723084e499eb87a6fe173b3af295d618"
```

*   如您所见，现在您可以将上述存储库添加到您的 Maven 代理服务器或系统中的全局 Maven 设置( *~/.m2/settings.xml* )中。

检查来自我的 Github 的[源代码。](https://github.com/hantsy/maven-archetype-jakartaee9)