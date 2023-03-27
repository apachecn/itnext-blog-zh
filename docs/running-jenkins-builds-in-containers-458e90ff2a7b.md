# 在容器中运行 Jenkins 构建

> 原文：<https://itnext.io/running-jenkins-builds-in-containers-458e90ff2a7b?source=collection_archive---------2----------------------->

随着 [Docker](https://opensource.com/resources/what-docker) 和 [Kubernetes](https://opensource.com/resources/what-is-kubernetes) 提供了一个可扩展的、可管理的应用平台，在容器中运行应用程序最近已经成为企业部门广泛接受的实践。基于容器的方法也非常适合在过去几年中势头强劲的[微服务架构](https://martinfowler.com/articles/microservices.html)。

现在，让我们只关注拥有容器应用程序平台的一个最重要的优势:我们可以动态地使用资源有限的隔离容器。让我们看看这将如何改变我们运行 CI/CD 任务的方式。

构建和打包应用程序需要一个可以下载源代码、访问依赖项并安装了构建工具的环境。作为构建的一部分运行单元和组件测试可能会使用本地端口，或者需要在某个地方运行第三方应用程序(数据库、消息代理等)。最后，我们通常有多个预先配置好的构建服务器，每个服务器都运行特定类型的作业。对于测试，我们维护第三方应用程序的专用实例(或尝试嵌入运行它们),并避免并行运行可能会破坏彼此结果的作业。这种 CI/CD 环境所需的预配置可能会很麻烦，而且随着团队在版本和开发平台之间的转换，不同工作所需的服务器数量可能会随着时间的推移而发生显著变化。

一旦我们可以访问容器平台(现场或云中)，为什么不将资源密集型 CI/CD 任务执行转移到动态创建的容器中呢？可以为每个作业的执行独立地启动和配置构建环境。构建过程中的测试可以自由地使用这个孤立的盒子中的可用资源，同时我们也可以在一个只存在于这个作业的生命周期中的侧面容器中调用第三方应用程序。

听起来不错…让我们看看它在现实生活中实际上是如何工作的。

> 这篇博客是基于一个真实世界的解决方案，我们为一个最近在 [Red Hat OpenShift](https://www.openshift.com/) v3.7 集群上运行的项目整理的。OpenShift 是 Kubernetes 的企业版，因此这些实践也适用于 k8s 集群。尝试下载[红帽 CDK](https://developers.redhat.com/products/cdk/overview/) 并运行[“詹金斯-短暂”或“詹金斯-持久”模板](https://github.com/openshift/origin/tree/master/examples/jenkins)，在 OpenShift 上为你创建一个预配置的詹金斯主模板。

## 解决方案概述

执行 CI/CD 任务(构建、测试等)的解决方案。)基于 [Jenkins 分布式构建](https://wiki.jenkins.io/display/JENKINS/Distributed+builds)，这意味着:

*   需要一个 Jenkins 主服务器，它可以在集群内部运行，但也可以与外部主服务器一起工作
*   Jenkins 特性/插件照常可用，因此可以使用现有的项目
*   Jenkins GUI 可用于配置、运行和浏览作业输出
*   如果你更喜欢代码，可以使用 [Jenkins Pipeline](https://jenkins.io/doc/book/pipeline/)

从技术角度来看，运行作业的动态容器是 Jenkins 代理节点。当构建开始时，首先启动一个新节点，并通过 JNLP(端口 5000)向 Jenkins 主节点“报到”。构建将排队等候，直到代理节点出现并获得构建。构建输出被发送回主服务器，就像普通的 Jenkins 代理服务器一样，但是一旦构建完成，代理容器就会关闭。

![](img/41f20a36a5f3fb57029267f43c902516.png)

不同类型的构建(例如，java、nodejs、python 等。)需要不同的代理节点。这并不新鲜——以前也可以使用标签来限制哪些代理节点应该运行构建。要为每个作业启动的 Jenkins 代理容器定义配置，您需要设置以下内容:

*   要启动的 docker 映像
*   资源限制
*   环境变量
*   已装载的卷

这里的核心组件是 Jenkins Kubernetes 插件。这个插件与 k8s 集群交互(通过使用 ServiceAccount)并启动/停止代理节点。在插件的配置下，可以将多个代理类型定义为“Kubernetes Pod 模板”(在项目中通过标签*引用它们)。*

这些[代理映像](https://access.redhat.com/containers/#/search/jenkins%2520slave)是开箱即用的(也在 [centos7](https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=openshift+jenkins+slave+&starCount=0) 上):

*   [jenkins-slave-base-rhel7](https://github.com/openshift/jenkins/tree/master/slave-base) :启动连接到 Jenkins master 的代理的基础映像。根据容器内存设置 Java 堆。
*   Jenkins-slave-Maven-rhel 7:Maven 和 Gradle 构建的图像(扩展基础)
*   [Jenkins-slave-nodejs-rhel 7](https://github.com/openshift/jenkins/tree/master/slave-nodejs):带有 NodeJS4 工具的图像(扩展基础)

> 请注意，这里描述的解决方案与 OpenShift 的[源到映像(S2I)](https://docs.openshift.com/container-platform/3.7/architecture/core_concepts/builds_and_image_streams.html#source-build) 构建无关，但也可以用于某些 CI/CD 任务。

## 学习材料

有几个关于 Jenkins 在 OpenShift 上构建的博客和很好的文档。这些链接是很好的开始:

*   [OpenShift Jenkins](https://docs.openshift.com/container-platform/3.7/using_images/other_images/jenkins.html) 图像文档和[来源](https://github.com/openshift/jenkins)
*   [带 OpenShift 的 CI/CD](https://blog.openshift.com/cicd-with-openshift/)网络广播
*   [外部詹金斯整合](http://v1.uncontained.io/playbooks/continuous_delivery/external-jenkins-integration.html)行动手册

快速浏览上面的这些资源，了解整体解决方案。在这里，我只想展示我们在应用这些实践时遇到的不同问题。

## 构建我的应用程序

作为一个[例子](https://github.com/bszeti/camel-springboot/tree/master/camel-rest-complex)，让我们假设一个 java 项目有以下构建步骤:

*   Source:从 Git 存储库中提取项目源代码。
*   使用 maven 构建:依赖项来自内部存储库(让我们使用 Apache Nexus)镜像外部 Maven 存储库。
*   部署工件:构建的 jar 被上传到存储库。

在 CI/CD 流程中，我们需要与 Git 和 Nexus 进行交互，因此 Jenkins 作业需要以某种方式访问这些系统。这需要可以在不同位置管理的配置和存储的凭据:

*   在 Jenkins 本身中:您可以向 Jenkins 添加 git 插件可以使用的凭证，并以某种方式向项目添加文件。没有什么新鲜的，只是因为我们使用容器。
*   在 OpenShift 中:使用作为文件或环境变量添加到 Jenkins 代理容器中的 ConfigMap 和 Secret 对象。
*   在完全定制的 Docker 映像中:预先配置了运行某类作业所需的一切。扩展其中一个代理映像。

这是一个偏好的问题，你最终的解决方案可能是一个组合。当配置主要在 OpenShift 中管理时，下面我们看一下第二个选项。我们通过设置环境变量和挂载文件，经由 Kubernetes 插件配置来定制 *maven* 代理容器。

> 由于一个[错误](https://issues.jenkins-ci.org/browse/JENKINS-47112)，通过 UI 添加环境变量在 Kubernetes plugin v1.0 版中不起作用。作为一种解决方法，更新插件或直接编辑 config.xml 并重新启动 Jenkins。

## 从 Git 提取源代码

拉一个公共 git 是微不足道的。对于私有 git repo，需要身份验证，并且客户端需要信任服务器以获得安全连接。Git 拉取通常可以通过两种协议完成:

*   HTTPS:使用用户名/密码进行认证。服务器的 SSL 证书必须受到作业的信任，这只有在它由自定义 CA 签名时才比较棘手。
    `git clone [https://git.mycompany.com:443/myapplication.git](https://git.mycompany.com:443/myproject.git)`
*   SSH:认证是用私钥进行的。当在 *known_hosts* 文件中找到其公钥的指纹时，服务器是可信的。
    `git clone ssh://git@git.mycompany.com:22/myapplication.git`

手动使用用户名/密码通过 HTTP 下载源代码是可以的，对于自动构建来说 SSH 更好。

## 使用 ssh 运行 Git

对于 ssh 下载，显然我们需要确保代理容器和 git 的 ssh 端口之间的 SSH 连接正常工作。首先，我们需要一个私钥-公钥对。为了生成一个运行:
`ssh keygen -t rsa -b 2048 -f my-git-ssh -N ''` 它在 *my-git-ssh* (空密码短语)中生成一个私钥，在 *my-git-ssh.pub* 中生成一个匹配的公钥。在 git 服务器上给用户添加公钥(最好是一个服务帐户)，web UIs 通常支持上传。要使 ssh 连接工作，代理容器中需要两个文件:

*   位于 *~/的私钥。ssh/id_rsa*
*   服务器的公钥在 *~/中。ssh/known_hosts*
    要做到这一点，请在您的机器上尝试`ssh git.mycompany.com` ，接受指纹，这将在您的 known_hosts 文件中创建一个新行。用那个。

将私钥作为 *id_rsa* 存储，将服务器的公钥作为 *known_hosts* 存储在 OpenShift secret(或配置映射)中。

```
apiVersion: v1
kind: Secret
metadata:
  name: mygit-ssh
stringData:
  id_rsa: |-
    -----BEGIN RSA PRIVATE KEY-----
    ...
    -----END RSA PRIVATE KEY-----
  known_hosts: |-
    git.mycompany.com ecdsa-sha2-nistp256 AAA...
```

然后在 Kubernetes 插件中为挂载点 */home/jenkins/的 *maven* pod 配置一个卷。ssh/。*密匙中的每一项都将是挂载目录下一个与密匙名称相匹配的文件。你可以使用 UI(管理 Jenkins/Configure/Cloud/Kubernetes)，或者编辑 Jenkins config*/var/lib/Jenkins/config . XML*:

```
<org.csanchez.jenkins.plugins.kubernetes.PodTemplate>
<name>maven</name>
...
  <volumes>
    <org.csanchez.jenkins.plugins.kubernetes.volumes.SecretVolume>
      <mountPath>/home/jenkins/.ssh</mountPath>
      <secretName>mygit-ssh</secretName>
    </org.csanchez.jenkins.plugins.kubernetes.volumes.SecretVolume>
  </volumes>
```

现在，通过 ssh 拉 git 源代码应该可以在这个代理上运行的作业中工作。

> 也可以在~/中自定义 ssh 连接。ssh/config。例如，如果我们不想麻烦 knows_hosts，或者私钥安装在不同的位置:

```
*Host git.mycompany.com
   StrictHostKeyChecking no
   IdentityFile /home/jenkins/.config/git-secret/ssh-privatekey*
```

## 使用 http 的 Git

如果首选 HTTP 下载，将用户名/密码添加到某处的 [git-credential-store](https://git-scm.com/docs/git-credential-store/1.8.2) 文件中:

*   例如/ *home/jenkins/。来自 OpenShift secret 的 config/git-secret/credentials*，每行一个站点:

```
[https://username:password@git.mycompany.com](https://bszeti:d5pgj-yFhurFPZTkHHby@gitlab.consulting.redhat.com)
[https://user:pass@github.com](https://username:password@git.mycompany.com)
```

*   在 */home/jenkins/预期的 [git-config](https://git-scm.com/docs/git-config/1.8.2) 中启用它。config/git/config*

```
[credential]
  helper = store --file=/home/jenkins/.config/git-secret/credentials
```

如果 git 服务有一个定制 CA 签署的证书，最快的方法就是为代理设置`GIT_SSL_NO_VERIFY=true` env var。合适的解决方案需要两样东西:

*   将自定义 CA 的公共证书从配置映射添加到代理容器的路径(例如*/usr/CA/mytrustedca . PEM*)
*   在一个 env var `GIT_SSL_CAINFO=/usr/ca/myTrustedCA.pem`
    或上面提到的 git-config 文件中告诉 git 这个证书的路径:

```
[http "https://git.mycompany.com"]
    sslCAInfo = /usr/ca/myTrustedCA.pem
```

> 在 OpenShift v3.7 版(以及更早版本)中，配置映射和秘密挂载点[不能重叠](https://bugzilla.redhat.com/show_bug.cgi?id=1430322)，所以你不能同时映射到 */home/jenkins* 和 */home/jenkins/dir* 。这就是为什么我们没有使用上面众所周知的文件位置。OpenShift v3.9 版有望修复。

## 专家

要让 Maven 构建工作，通常要做两件事:

*   应建立一个公司 Maven 存储库(如 Apache Nexus ),作为外部回购的代理。用这个当镜子。
*   这个内部存储库可能有一个 https 端点，该端点带有由自定义 CA 签名的证书。

如果构建在容器中运行，拥有一个内部 maven 存储库实际上是必不可少的，因为它们从一个空的本地存储库(缓存)开始，所以 Maven 每次都下载所有的 jar。从本地网络上的内部代理回购下载显然比直接从互联网上下载要快。

[*maven* Jenkins 代理](https://github.com/openshift/jenkins/tree/master/slave-maven)镜像支持一个环境变量，该变量可用于设置该代理的 url。在 Kubernetes 插件容器中设置模板:
`MAVEN_MIRROR_URL=https://nexus.mycompany.com/repository/maven-public`

构建工件(jar)也应该归档在一个存储库中，这个存储库可能是也可能不是上面作为依赖关系镜像的那个存储库。Maven *deploy* 需要[*distribution management*](https://maven.apache.org/pom.html#Distribution_Management)下 pom.xml 中的 repo url(所以这与代理映像无关):

```
<project ...>
<distributionManagement>
 <snapshotRepository>
  <id>mynexus</id>
  <url>[https://nexus.mycompany.com/repository/maven-snapshots/](http://nexus3-bszeti-ci-cd.apps.na1.openshift.opentlc.com/repository/maven-snapshots/)</url>
 </snapshotRepository>
 <repository>
  <id>mynexus</id>
  <url>[https://nexus.mycompany.com](http://nexus3-bszeti-ci-cd.apps.na1.openshift.opentlc.com/repository/maven-snapshots/)/[repository/maven-releases/](http://nexus3-bszeti-ci-cd.apps.na1.openshift.opentlc.com/repository/maven-releases/)</url>
 </repository>
</distributionManagement>
```

上传工件可能需要身份验证。在这种情况下，用户名/密码必须设置在 settings.xml 中与 pom.xml 中的服务器 id 相匹配的服务器 id 下。我们需要从 OpenShift secret 在 *maven* Jenkins 代理容器上挂载一个包含 url、用户名和密码的完整 settings.xml。我们还可以使用如下环境变量:

*   将环境变量从一个 secret 添加到容器
    中`MAVEN_SERVER_USERNAME=admin
    MAVEN_SERVER_PASSWORD=admin123`
*   将 settings.xml 从配置映射挂载到*/home/Jenkins/. m2/settings . XML*

```
<settings ...>
 <mirrors>
  <mirror>
   <mirrorOf>external:*</mirrorOf>
   <url>${env.MAVEN_MIRROR_URL}</url>
   <id>mirror</id>
  </mirror>
 </mirrors>
 <servers>
  <server>
   <id>mynexus</id>
   <username>${env.MAVEN_SERVER_USERNAME}</username>
   <password>${env.MAVEN_SERVER_PASSWORD}</password>
  </server>
 </servers>
</settings>
```

> 禁用交互模式(使用批处理模式)来跳过下载日志，方法是对 maven 命令使用'-B '，或者在 settings.xml 中添加`*<interactiveMode>false</interactiveMode*` >

如果 Maven repository https 端点使用由定制 CA 签署的证书，我们需要使用 [keytool](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html) 创建一个包含可信 CA 证书的 Java 密钥库。这个密钥库应该作为 Openshift 中的配置映射上传。使用 *oc* 命令从文件创建配置图:

`oc create configmap maven-settings --from-file=settings.xml=settings.xml --from-file=myTruststore.jks=myTruststore.jks`

在 Jenkins 代理上的某个地方挂载配置映射。在这个例子中，我们使用了 */home/jenkins/.m2，*，但是仅仅因为我们在同一个配置映射中有 settings.xml，所以密钥库可以放在任何路径下。

然后，通过在容器的 MAVEN_OPTS 环境变量中设置 java 参数，使 Maven java 进程将该文件用作信任库:

```
MAVEN_OPTS=
-Djavax.net.ssl.trustStore=/home/jenkins/.m2/myTruststore.jks
-Djavax.net.ssl.trustStorePassword=changeit
```

## 内存使用

这可能是最重要的部分，因为如果你没有正确设置最大内存，你会在一切看起来正常工作后遇到间歇性的构建失败。

如果我们不在 java 命令行中设置堆，在容器中运行 java 会导致高内存使用错误。JVM [看到的是主机](https://developers.redhat.com/blog/2017/03/14/java-inside-docker/)的总内存，而不是容器的内存限制，并相应地设置[默认最大堆](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size)。这通常远远超过容器的内存限制，当 java 进程为堆分配更多内存时，OpenShift 会直接杀死容器。

尽管 jenkins-slave-base 映像有一个内置的[脚本来将 max heap](https://github.com/openshift/jenkins/blob/master/slave-base/contrib/bin/run-jnlp-client) 设置为容器内存的一半(可以通过 env var `CONTAINER_HEAP_PERCENT=0.50`修改)，但它只适用于 jenkins 代理 java 进程。在 Maven 构建的情况下，我们有重要的额外 java 进程在运行:

*   mvn 命令本身就是一个 java 工具
*   默认情况下, [maven-surefire-plugin](http://maven.apache.org/surefire/maven-surefire-plugin/examples/fork-options-and-parallel-execution.html) 在分叉的 JVM 中执行单元测试

一天结束时，我们将有三个 java 进程同时在容器中运行，估计它们的内存使用情况以避免意外死亡是很重要的。每个进程都有不同的方式来设置 JVM 选项:

*   詹金斯代理堆如上所述计算，但我们绝对不应该让代理有这么大的堆。另外两个 JVM 需要内存。设置 *JAVA_OPTS* 适用于 Jenkins 代理。
*   Jekins 作业调用 *mvn* 工具。设置 *MAVEN_OPTS* 来定制这个 java 进程。
*   surefire 为单元测试生成的 JVM 可以通过[*argLine*](http://maven.apache.org/surefire/maven-surefire-plugin/test-mojo.html#argLine)maven 属性定制。它可以在 pom.xml 中设置，在 settings.xml 的配置文件中设置，或者简单地通过在 *MAVEN_OPTS* 的 *mvn* 命令中添加 *-DargLine=…* 来设置

下面是一个如何为 *maven* 代理容器设置这些环境变量的例子:
`JAVA_OPTS=-Xms64m -Xmx64m
MAVEN_OPTS=-Xms128m -Xmx128m -DargLine=${env.SUREFIRE_OPTS}
SUREFIRE_OPTS=-Xms256m -Xmx256m` 这些数字在我们为 SpringBoot 应用程序构建和运行 1024Mi 代理容器内存限制单元测试时发挥了作用。这些都是相对较低的数字和较大的堆大小，复杂的 maven 项目和单元测试可能需要更高的限制。

> Java8 进程的实际内存使用量大约是 heap size+MetaSpace+OffHeapMemory，这可能远远大于最大堆大小设置。按照上面的设置，在我们的例子中，三个 java 进程总共占用了 900 多兆内存。容器内进程见 rss 内存:
> `ps -e -o pid,user,rss,comm,args`

Jenkins 代理映像安装了 JDK 64 位和 32 位。对于 mvn 和 surefire，默认使用 64 位 JVM。为了降低内存使用，只要-Xmx 小于 1.5 GB，强制使用 32 位 JVM 是有意义的:
`JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161–0.b14.el7_4.i386`

> 还可以在 java_TOOL_OPTIONS env var 中设置 JAVA 参数，任何启动的 JVM 都会选择它。JAVA_OPTS 和 MAVEN_OPTS 中的参数覆盖了 JAVA_TOOL_OPTIONS 中的参数，因此我们可以在不使用 *argLine* :
> `JAVA_OPTS=-Xms64m -Xmx64m
> MAVEN_OPTS=-Xms128m -Xmx128m
> JAVA_TOOL_OPTIONS=-Xms256m -Xmx256m` 的情况下为我们的 JAVA 进程实现与上面相同的堆配置，这仍然有点令人困惑，因为所有 JVM 都记录了“挑选 JAVA_TOOL_OPTIONS:”。

## 詹金斯管道公司

按照上面的设置，我们现在应该已经为运行一个成功的构建做好了准备。我们可以提取代码，下载依赖项，运行单元测试，并将工件上传到我们的仓库。让我们创建一个 Jenkins pipeline 项目来实现这一点:

对于一个真实的项目，当然你的 CI/CD 管道应该做的不仅仅是 maven 构建:部署到开发环境，运行集成测试，提升到更高的环境，等等。上面链接的文章展示了如何做的例子。

## 多个容器

一个 pod 可以运行多个容器，每个容器都有自己的资源限制。它们共享同一个网络接口，所以您可以在 *localhost* 上访问已启动的服务，但是您需要考虑端口冲突。环境变量是单独设置的，但是对于在一个 Kubernetes Pod 模板中配置的所有容器，装载的卷是相同的。

当单元测试需要一个外部服务，而一个嵌入式解决方案不起作用时(例如，数据库、消息代理等)，使用多个容器是很有用的。).在这种情况下，第二个容器也以 Jenkins 代理开始和停止。

请参见 Jenkins config.xml 片段，其中我们在 Maven 构建的侧面启动了一个 httpbin 服务:

```
<org.csanchez.jenkins.plugins.kubernetes.PodTemplate>
  <name>maven</name>
  <volumes>
    ...
  </volumes>
  <containers>
    <org.csanchez.jenkins.plugins.kubernetes.ContainerTemplate>
      <name>jnlp</name>
      <image>registry.access.redhat.com/openshift3/jenkins-slave-maven-rhel7:v3.7</image>
      <resourceLimitCpu>500m</resourceLimitCpu>
      <resourceLimitMemory>1024Mi</resourceLimitMemory>
      <envVars>
      ...
      </envVars>        
      ...
    </org.csanchez.jenkins.plugins.kubernetes.ContainerTemplate>
    <org.csanchez.jenkins.plugins.kubernetes.ContainerTemplate>
      <name>httpbin</name>
      <image>citizenstig/httpbin</image>
      <resourceLimitCpu></resourceLimitCpu>
      <resourceLimitMemory>256Mi</resourceLimitMemory>
      <envVars/>
      ...
    </org.csanchez.jenkins.plugins.kubernetes.ContainerTemplate>
  </containers>
  <envVars/>
</org.csanchez.jenkins.plugins.kubernetes.PodTemplate>
```

## 摘要

作为总结，请参见 Jenkins config.xml 中创建的 OpenShift 资源和 Kubernetes 插件配置，以及上面描述的所有配置。

从文件
`oc create configmap maven-settings --from-file=settings.xml=settings.xml --from-file=myTruststore.jks=myTruststore.jks`创建了一个额外的配置图

Kubernetes 插件配置:

快乐建造！

> 感谢[乌什纳什](http://ushnash.blogspot.com/)的贡献。