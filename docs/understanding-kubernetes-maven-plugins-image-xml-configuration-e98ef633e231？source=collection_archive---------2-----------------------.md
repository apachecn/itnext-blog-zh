# 了解 Kubernetes Maven 插件的图像 XML 配置

> 原文：<https://itnext.io/understanding-kubernetes-maven-plugins-image-xml-configuration-e98ef633e231?source=collection_archive---------2----------------------->

![](img/47d070768d1a77f5505555855fcc3df7.png)

[日蚀 JKube](https://www.eclipse.org/jkube)

在我以前的博客中，您可能已经看到了 Eclipse JKube 如何通过提供自以为是的默认值(也称为[零配置模式](https://www.eclipse.org/jkube/docs/kubernetes-maven-plugin#zero-config))来简化映像构建过程。

但有时，您可能希望在应用程序的 docker 映像中进行配置。这些包括像激活一个额外的端口，添加额外的文件到你的图像，改变入口点等。在这篇博客中，我们将看看如何利用 Eclipse JKube 的 image XML 配置来根据您的需要创建任何类型的 Docker 图像。Eclipse JKube 也支持 Dockerfiles，我们将在另一篇博客中看到。

假设您想要构建一个简单的 docker 映像，其 docker 文件如下所示:

```
**FROM** adoptopenjdk/openjdk11:alpine-jre 
**WORKDIR** /usr/home/app 
**COPY** target/your-project.jar your-project.jar 
**EXPOSE** 8080 
**CMD** ["java", "-jar", "your-project.jar"]
```

这是一个非常简单的 docker 文件，它将在目标中找到的任何 jar 文件复制到容器中，然后执行它。如果我们想通过 Eclipse JKube Plugins 的 XML 配置做同样的事情，看起来应该是这样的:

```
**<plugin>
  <groupId>**org.eclipse.jkube**</groupId>
  <artifactId>**kubernetes-maven-plugin**</artifactId>
  <version>**${jkube.version}**</version>
  <configuration>
     <images>
        <image>
          <!--(1)-->
          <name>**user/imagename:latest**</name>
           <build>
              <!--(2)-->
              <from>**adoptopenjdk/openjdk11:alpine-jre**</from>**
              **<!--(3)-->
              <assembly>
                 <mode>**dir**</mode>
                 <targetDir>**/usr/home/app**</targetDir>
                 <inline>**
                    **<id>**copy-jar**</id>
                    <baseDirectory>**/home/home/app**</baseDirectory>
                    <!--(4)-->**
                    **<files>**
                       **<file>**
                          **<source>** target/${project.artifactId}-${project.version}.jar
                          **</source>**
                          **<outputDirectory>**.**</outputDirectory>**
                       **</file>**
                    **</files>**
                 **</inline>**
              **</assembly>
              <!--(5)-->
**              **<workdir>**/usr/home/app**</workdir>
              <!--(6)-->**
              **<cmd>** java -jar ${project.artifactId}-${project.version}.jar
              **</cmd>
              <!--(7)-->
              <ports>
                 <port>**8080**</port>**
              **</ports>
           </build>
         </image>
     </images>
  </configuration>
</plugin>**
```

以下是解释的所有组件:

1.  要构建的图像的名称
2.  此应用程序映像所基于的基本映像，`FROM`等效
3.  用于将文件/目录复制到您的映像的程序集配置
4.  要复制到映像中的文件列表
5.  工作目录为镜像，`WORKDIR`等效
6.  容器启动时要运行的命令，`CMD`等效
7.  端口暴露在您的图像中，`EXPOSE`相当于

你可以像往常一样使用`mvn k8s:build`目标来建立你的形象。您还可以检查图像，以检查您的 jar 是否被正确复制。

```
**$** mvn k8s:build...**$** docker run -it cf0c7 /bin/sh 
/usr/home/app # ls 
random-generator-0.0.1.jar
```

## 将多个文件复制到映像中:

Eclipse JKube 还允许您将多个文件复制到映像中。假设您的项目目录中有两个文件，您想将其复制到映像中:

```
**eclipse-jkube-demo-project : $** ls -l 

-rw-rw-r--. 1 rohaan rohaan    19 May 16 19:47 first.txt 
-rwxrwxr-x. 1 rohaan rohaan  4259 May 10 20:30 **pom.xml** 
-rw-rw-r--. 1 rohaan rohaan    20 May 16 19:47 second.txt 
drwxrwxr-x. 1 rohaan rohaan    16 Nov 16 19:06 **src** 
drwxrwxr-x. 1 rohaan rohaan   394 May 10 20:32 **target**
```

如果我想将`first.txt`和`second.txt`与我的 jar 文件一起复制到我的映像中。我需要添加额外的`<file>`元素，引用需要复制到图像的文件的路径，如下所示:

```
**<assembly>
    <mode>**dir**</mode>
    <targetDir>**/usr/home/app**</targetDir>
    <inline>
        <id>**copy-jar<**/id>
        <baseDirectory>**/usr/home/app**</baseDirectory>**
        **<files>**
            **<file>
                <source>**first.txt**</source>
                <outputDirectory>**.**</outputDirectory>
            </file>
            <file>
                <source>**second.txt**</source>
                <outputDirectory>**.**</outputDirectory>
            </file>
        </files>
    </inline>
</assembly>**
```

## 将目录复制到映像:

对于复制目录，您需要使用`<fileSet>` XML 元素。假设您想要将`src/main/resources`和`target/classes`目录复制到您的目标映像。下面是使用 XML 配置的方法:

```
**<assembly>
    <mode>**dir**</mode>
    <targetDir>**/usr/home/app**</targetDir>
    <inline>
        <id>**copy-jar**</id>
        <baseDirectory>**/usr/home/app**</baseDirectory>
        <fileSets>
            <fileSet>
                <directory>**target/classes**</directory>
                <outputDirectory>**classes**</outputDirectory>
            </fileSet>
            <fileSet>
                <directory>**src/main/resources**</directory>
                <outputDirectory>**resources**</outputDirectory>
            </fileSet>
        </fileSets>
    </inline>
</assembly>**
```

然后您可以像往常一样执行`mvn k8s:build`,检查目录是否被复制到您的容器映像中:

```
**$** mvn k8s:build...**$** docker run -it 1706a /bin/sh 
/usr/home/app # ls 
**classes**                     random-generator-0.0.1.jar  **resources** 
/usr/home/app # cd resources/ 
/usr/home/app/resources # ls 
application.properties 
/usr/home/app/resources # cat application.properties  
spring.devtools.remote.secret=mysecret
```

## 切换映像构建策略:

Eclipse JKube 不仅使用 [Docker](https://www.docker.com/) 构建容器映像，还提供了构建容器映像的替代方法。在撰写本文时，它还支持 JIB，使用它您可以构建您的容器映像，而无需任何类型的守护程序。还有 [S2I](https://github.com/openshift/source-to-image) 构建策略，但它只在 openshift-maven-plugin 的情况下可用。

默认情况下，Docker 是默认的构建策略。但是您可以通过`jkube.build.strategy`属性或者通过`<buildStrategy>` XML 配置元素来定制它，如下所示:

```
**<plugin>
    <groupId>**org.eclipse.jkube**</groupId>
    <artifactId>**kubernetes-maven-plugin**</artifactId>
    <version>**${jkube.version}**</version>
    <configuration>** <!-- JIB Build Strategy: No daemon required--> **<buildStrategy>**jib**</buildStrategy>
        <images>
        ...
        </images>
    </configuration>
</plugin>**
```

## 指定容器注册表凭据:

Eclipse JKube 还允许您将图像推送到容器注册中心。您可以通过`docker login` (Eclipse JKube 读作`~/.docker/config.json`)或您的`~/.m2/settings.xml`来提供容器注册凭证。但是您也可以使用 XML 配置使用`<authConfig>`部分来指定这些。假设您已经为注册表用户名和密码定义了环境变量:

```
**$** export DOCKERHUB_USERNAME=myusername
**$** export DOCKERHUB_PASSWORD=secret
```

您可以像这样在`<authConfig>`中指定这些:

```
**<plugin>
    <groupId>**org.eclipse.jkube**</groupId>
    <artifactId>**kubernetes-maven-plugin**</artifactId>
    <version>**${jkube.version}**</version>
    <configuration>
        <images> 
          ...
        </images> 
        <authConfig>
            <username>**${env.DOCKERHUB_USERNAME}**</username>
            <password>**${env.DOCKERHUB_PASSWORD}**</password>
        </authConfig>
    </configuration>
</plugin>**
```

## 结论:

我希望这篇博客简要介绍了如何在为 java 应用程序构建容器映像的同时利用 Eclipse JKube XML 配置。你可以在 [Eclipse JKube 构建配置文档](https://www.eclipse.org/jkube/docs/kubernetes-maven-plugin#build-goal-configuration)中读到更多关于它的细节。