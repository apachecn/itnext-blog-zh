# 使用 Maven 构建 Docker

> 原文：<https://itnext.io/using-maven-to-build-docker-73c83e626025?source=collection_archive---------0----------------------->

![](img/13dae0cba9ffd2f41d69524f5405e14c.png)

图片由来自 [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=1080589) 的 [Michal Jarmoluk](https://pixabay.com/users/jarmoluk-143740/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=1080589) 拍摄

Docker 正迅速成为服务的标准交付和部署工具。但是对于 Java web 应用程序呢？我们怎么才能加入码头工人派对？

# 传统码头建筑

传统的 docker 构建不需要太多。安装 docker 和 docker 文件就可以满足你的所有需求。有许多预构建的 docker 容器可以用作您的基础:

*   [香草爪哇](https://hub.docker.com/_/java)
*   雄猫

设置您的 docker 文件:

```
**FROM** openjdk:8-jdk-alpine
**VOLUME /**tmp
**EXPOSE** 8080
**ARG *JAR_FILE* COPY** ${***JAR_FILE***} app.jar
**ENTRYPOINT** [**"java"**,**"-Djava.security.egd=file:/dev/./urandom"**,**"-jar"**,**"/app.jar"**]
```

然后运行您的构建:

```
docker build \
-t io.github.efenglu.example/mavenDocker:0.0.1-SNAPSHOT \
--build-arg JAR_FIlE=target/dependencies/service.jar \
--no-cache .
```

当然，这会让你跑起来，但我们在这里错过了很多。

*   我们如何将它与我们的持续集成过程集成呢？
*   web.war 正走在一条神奇的道路上(共享隐性知识)
*   我们的应用程序版本是作为命令行参数提供的

# Maven Docker 构建

您可能已经在使用 maven 构建您的 jar 和 war 了。Maven 有版本信息，并且已经集成到我们的持续集成过程中。

但是我们如何让它构建 docker 呢？

Maven 构建 java…对吧？

是的，maven 构建 java。但其核心 Maven 只是一个构建框架。你可以写一个 maven 插件来做任何事情。因此，让我们让 maven 来构建我们的容器。有几种不同的方法。各有利弊。

# Maven Exec

maven exec 插件提供了一种简单的方法来执行作为 maven 构建一部分的任何程序。让我们像从命令行那样使用它来构建 docker。

首先，我们需要得到我们需要部署到容器中的工件。我们可以使用依赖插件将所有工件提取到一个已知的位置。

```
<**plugin**>
    <**groupId**>org.apache.maven.plugins</**groupId**>
    <**artifactId**>maven-dependency-plugin</**artifactId**>
    <**executions**>
        <**execution**>
            <**goals**>
                <**goal**>copy</**goal**>
            </**goals**>
            <**phase**>prepare-package</**phase**>
            <**configuration**>
                <**artifactItems**>
                    <**artifactItem**>
                        <**groupId**>${project.groupId}</**groupId**>
                        <**artifactId**>example-service</**artifactId**>
                        <**version**>${project.version}</**version**>
                        <**overWrite**>true</**overWrite**>
                        <**outputDirectory**>target/dependency</**outputDirectory**>
                        <**destFileName**>service.jar</**destFileName**>
                    </**artifactItem**>
                </**artifactItems**>
            </**configuration**>
        </**execution**>
    </**executions**>
</**plugin**>
```

这里，我们将工件 jar 复制到我们的目标/依赖文件夹中，命名为 service.jar

我们现在可以将它传递给我们的 docker 构建，以及其他信息，如版本:

```
<**plugin**>
    <**groupId**>org.codehaus.mojo</**groupId**>
    <**artifactId**>exec-maven-plugin</**artifactId**>
    <**version**>1.6.0</**version**>
    <**executions**>
        <**execution**>
            <**id**>docker-package</**id**>
            <**phase**>package</**phase**>
            <**goals**>
                <**goal**>exec</**goal**>
            </**goals**>
            <**configuration**>
                <**executable**>docker</**executable**>
                <**workingDirectory**>${project.basedir}</**workingDirectory**>
                <**arguments**>
                    <**argument**>build</**argument**>
                    <**argument**>.</**argument**>
                    <**argument**>-t</**argument**>
                    <**argument**>${project.groupId}/${project.artifactId}:${project.version}</**argument**>
                    <**argument**>--build-arg</**argument**>
                    <**argument**>JAR_FILE=target/dependency/service.jar</**argument**>
                </**arguments**>
            </**configuration**>
        </**execution**>
    </**executions**>
</**plugin**>
```

在这里，您可以看到我们正在当前项目目录中调用 docker 命令。我们传递参数，使命令看起来像这样:

```
docker build . -t ${project.groupId}/${project.artifactId}:${project.version} --build-arg JAR_FILE=target/dependency/service.jar
```

组 id、工件 id 和版本将被 maven 值替换。

## 优势:

*   简单的
*   完整的 Dockerfile 文件支持

## 不足之处

*   docker 文件系统分层效率不高
*   需要访问 docker 代理来构建
*   maven 和 docker 之间的原始集成级别

## 进一步阅读

*   [https://www.mojohaus.org/exec-maven-plugin/](https://www.mojohaus.org/exec-maven-plugin/)

# Spotify Maven Docker 插件

Spotify Maven 插件抽象了一些 docker 概念，并提供了一个更传统的类似 Maven 的界面来配置 docker 版本。然而，它仍然利用 docker 代理来执行实际的构建。

首先，通过在我们的构建参数中设置从哪里获取 jar 来配置插件，并设置一个归档清单以包含大量的构建元数据。

```
<**plugin**>
    <**groupId**>com.spotify</**groupId**>
    <**artifactId**>dockerfile-maven-plugin</**artifactId**>
    <**version**>1.4.9</**version**>
    <**configuration**>
        <**buildArgs**>
            <**DEPENDENCY**>target/dependency</**DEPENDENCY**>
        </**buildArgs**>
        <**archive**>
            <**manifest**>
                <**addDefaultImplementationEntries**>true</**addDefaultImplementationEntries**>
                <**addDefaultSpecificationEntries**>true</**addDefaultSpecificationEntries**>
            </**manifest**>
            <**manifestEntries**>
                <**Specification-Title**>${project.name}</**Specification-Title**>
                <**Specification-Version**>${project.version}</**Specification-Version**>
                <**Implementation-Title**>${project.groupId}.${project.artifactId}</**Implementation-Title**>
                <**Implementation-Build-Time**>${maven.build.timestamp}</**Implementation-Build-Time**>
                <**X-Docker-Repo**>${docker.org}/${docker.image}:${project.version}</**X-Docker-Repo**>
            </**manifestEntries**>
        </**archive**>
    </**configuration**>
```

然后，我想在适当的 maven 阶段添加一个执行:

在打包阶段，我们构建容器:

```
<**execution**>
    <**id**>build</**id**>
    <**phase**>package</**phase**>
    <**goals**>
        <**goal**>build</**goal**>
    </**goals**>
    <**configuration**>
        <**repository**>${docker.org}/${docker.image}</**repository**>
        <**tag**>${project.version}</**tag**>
        <**classifier**>docker-local</**classifier**>
    </**configuration**>
</**execution**>
```

在安装阶段，我们用一个本地标签和一个更适合部署到 DTR 存储库的标签来标记容器:

```
<**execution**>
    <**id**>tag</**id**>
    <**phase**>install</**phase**>
    <**goals**>
        <**goal**>tag</**goal**>
    </**goals**>
    <**configuration**>
        <**repository**>${docker.org}/${docker.image}</**repository**>
        <**tag**>latest</**tag**>
        <**classifier**>docker-local-latest</**classifier**>
    </**configuration**>
</**execution**>
<**execution**>
    <**id**>tag-repo-deploy</**id**>
    <**phase**>install</**phase**>
    <**goals**>
        <**goal**>tag</**goal**>
    </**goals**>
    <**configuration**>
        <**repository**>${docker.registry}/${docker.org}/${docker.image}</**repository**>
        <**tag**>${docker.deploy.tag}</**tag**>
        <**classifier**>docker-info-tag</**classifier**>
    </**configuration**>
</**execution**>
```

最后，在部署阶段，我们将容器推送到 DTR:

```
<**execution**>
    <**id**>push-version</**id**>
    <**phase**>deploy</**phase**>
    <**goals**>
        <**goal**>push</**goal**>
    </**goals**>
    <**configuration**>
        <**repository**>${docker.registry}/${docker.org}/${docker.image}</**repository**>
        <**tag**>${docker.deploy.tag}</**tag**>
        <**classifier**>docker-info</**classifier**>
    </**configuration**>
</**execution**>
```

## 优势:

*   简单的
*   对 docker 概念更直接的支持
*   支持 Dockerfile

## 缺点:

*   Docker 文件系统层
*   需要码头代理

## 延伸阅读:

*   [Spotify Docker Maven 插件](https://github.com/spotify/docker-maven-plugin)

# 谷歌三角帆

JIB 插件比 Spotify 更进一步。JIB 直接从 Java 与 docker 文件系统层进行交互。这消除了对任何外部工具的需要，如运行 docker 代理。但这也意味着您只能将文件复制到基本容器中。您不能执行任何运行操作。

JIB 插件没有太多需要配置的。

```
<**configuration**>
    <**allowInsecureRegistries**>true</**allowInsecureRegistries**>
    <**to**>
        <**image**>${project.groupId}/${project.artifactId}:${project.version}</**image**>
    </**to**>
    <**container**>
        <**ports**>
            <**port**>${jib.app.port}</**port**>
        </**ports**>
        <**jvmFlags**>
            <**jvmFlag**>-Xms512m</**jvmFlag**>
            <**jvmFlag**>-Xmx1024m</**jvmFlag**>
        </**jvmFlags**>
    </**container**>
</**configuration**>
```

因为 JIB 不需要 docker 容器，所以默认情况下它不会提供容器的本地 docker 版本。要创建本地容器构建，您可以添加以下执行:

```
<**execution**>
    <**id**>docker daemon install</**id**>
    <**phase**>install</**phase**>
    <**goals**>
        <**goal**>dockerBuild</**goal**>
    </**goals**>
    <**configuration**>
        <**skip**>${jib.skip.install}</**skip**>
    </**configuration**>
</**execution**>
```

这将使用本地 docker 代理构建容器。这对调试很有用。但是，您应该在 CI/CD 框架的构建过程中禁用它，以避免依赖 docker 代理。因此，跳过我们定义的属性。

最后，您还需要配置 JIB 以部署到您的 DTR 实例:

```
<**execution**>
    <**id**>DTR Deploy</**id**>
    <**phase**>deploy</**phase**>
    <**goals**>
        <**goal**>build</**goal**>
    </**goals**>
    <**configuration**>
        <**skip**>${jib.skip.deploy}</**skip**>
        <**to**>
            <**auth**>
                <**username**>${jib.dtr.auth.username}</**username**>
                <**password**>${jib.dtr.auth.password}</**password**>
            </**auth**>
        </**to**>
    </**configuration**>
</**execution**>
```

## 优势:

*   小而快
*   不需要代理或任何其他外部可执行文件(在 ci/cd 中)
*   更小的文件系统分层直接与 maven 依赖项集成

## 缺点:

*   只有简单的容器，没有 Dockerfile 支持

## 延伸阅读:

*   [谷歌 JIB 插件](https://github.com/GoogleContainerTools/jib)

# 结论

一般来说，对于基于 Java 的 web 服务，你主要是将一个 jar 复制到一个容器 JIB 中，这绝对是正确的做法。使用 docker 代理在持续集成环境中构建容器可能非常复杂，尤其是当您的构建环境已经在容器中时。您将很快遇到这样的问题，要么必须将/var/docker.sock 卷装载到您的构建容器中(这是一个巨大的安全风险)，要么以某种方式让 docker-in-docker 工作。那还不容易！有许多文件系统问题、主机系统兼容性问题、版本和性能问题。一般情况下，不要使用 docker-in-docker。这是为了调试 docker，而不是在您的 CI/CD 框架中使用。

如果你想试试 DIND 码头，你可以在这里找到更多的细节:[https://github.com/jpetazzo/dind](https://github.com/jpetazzo/dind)

JIB 没有这些问题。它不需要任何外部程序或服务来运行。这是一个巨大的优势。

但是如果除了文件复制之外，您还需要以任何方式定制容器，那么您真的需要使用 Maven exec 或 SonaType。但实际上，我可能会建议您也看看您的云为 docker 构建提供了哪些本机支持，也许不要使用 maven 来编排这些构建。

总的来说，我建议你使用类似 [Kaniko](https://github.com/GoogleContainerTools/kaniko) 的东西来构建容器，然后限制自己只为你的应用程序安装 jar。

完整的例子请查看我的 Github Repo:

[](https://github.com/efenglu/docker-maven-build) [## 埃丰路/码头-马文-建造

### 使用 maven 构建 docker 容器的一组示例配置

github.com](https://github.com/efenglu/docker-maven-build)