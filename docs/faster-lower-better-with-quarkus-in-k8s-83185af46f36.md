# k8s 的夸库更快，更低，更好

> 原文：<https://itnext.io/faster-lower-better-with-quarkus-in-k8s-83185af46f36?source=collection_archive---------3----------------------->

为什么 Docker 和 Kubernetes 具有非常快的启动速度、低内存占用、带标准框架的本机编译，从而提高可伸缩性、弹性和安全性。

![](img/6847b637654e5f8010c9327b7575bebc.png)

照片由 [Guillaume Jaillet](https://unsplash.com/@i_am_g?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

# TL；DR；

在这篇博客文章中，我向您展示了 Quarkus 如何在 Docker 中使用 Docker 中最少的映像(也是最安全的)在本地运行 Java 应用程序(通过用 GraalVM 将 Java 字节码编译成机器码),并将其部署在 kubernetes 上。

# 夸库斯。

> 一个为 OpenJDK HotSpot 和 GraalVM 定制的 Kubernetes 原生 Java 栈，由同类最佳的 Java 库和标准精心打造。

[Quarkus](https://quarkus.io/) 是由 RedHat 开发的一个框架，它是为容器世界设计的，具有以下特点:

*   启动和停止非常快，包括第一次响应时间
*   低内存占用
*   可以用 GraalVM 编译成本机代码
*   使用同类最佳的 Java 库和标准:Eclipse MicroProfile、CDI、JAX-RS、JPA、Vert.x、Apache Camel、kafka、Vault、..(参见 [https://code.quarkus.io](https://code.quarkus.io) 和[https://quarkus.io/guides/](https://quarkus.io/guides/))
*   支持命令式和反应式代码或两者的混合(参见[https://quarkus.io/vision/continuum](https://quarkus.io/vision/continuum)
*   简单统一的配置(参见[https://quarkus.io/guides/all-config](https://quarkus.io/guides/all-config)

# GraalVM？

> GraalVM 是一个通用虚拟机，用于运行用 JavaScript、Python、Ruby、R、基于 JVM 的语言编写的应用程序，如 Java、Scala、Groovy、Kotlin、Clojure 和基于 LLVM 的语言，如 C 和 C++。
> 
> GraalVM 消除了编程语言之间的隔离，并支持共享运行时的互操作性。它可以独立运行，也可以在 OpenJDK、Node.js 或 Oracle 数据库的上下文中运行。

GraalVm 由 Oracle 开发，能够本地运行基于 JVM 的语言(通过将 Java 字节码编译成机器码),并支持其他语言，如 JavaScript、Python、Ruby、C、C++、R 等。

# 演示

为了演示这种方法，我使用了一个基于[https://github . com/quarkusio/quarkusquickstarts/tree/master/getting-started](https://github.com/quarkusio/quarkus-quickstarts/tree/master/getting-started)的演示应用程序

这是一个最小的 CRUD 服务，通过 REST 公开了几个端点。
在幕后，这个演示使用 RESTEasy 来公开 REST 端点。

您可以在以下 GitHub 资源库中找到源代码:

[](https://github.com/sokube/quarkus-scratch/blob/master/Dockerfile) [## sokube/quarkus-scratch

### 在 GitHub 上创建一个帐户，为 sokube/quarkus-scratch 开发做贡献。

github.com](https://github.com/sokube/quarkus-scratch/blob/master/Dockerfile) 

# 多级码头建造

为了生成具有严格最小值的图像，使用[多级 docker](https://docs.docker.com/develop/develop-images/multistage-build/) 构建，其中基于图像的最后一级为“ [scratch](https://docs.docker.com/develop/develop-images/baseimages/#create-a-simple-parent-image-using-scratch) ”:

[https://github . com/sokube/quar kus-scratch/blob/master/docker file](https://github.com/sokube/quarkus-scratch/blob/master/Dockerfile):

```
### Image for getting maven dependencies and then acting as a cache for the next image
FROM maven:3.6.3-jdk-11 as mavencache
ENV MAVEN_OPTS=-Dmaven.repo.local=/mvnrepo
COPY pom.xml /app/
WORKDIR /app
RUN mvn test-compile dependency:resolve dependency:resolve-plugins

### Image for building the native binary
FROM oracle/graalvm-ce:19.3.1-java11 AS native-image
ENV MAVEN_OPTS=-Dmaven.repo.local=/mvnrepo
COPY --from=mavencache /mvnrepo/ /mvnrepo/
COPY . /app
WORKDIR /app
ENV GRAALVM_HOME=/usr
RUN gu install native-image && \
    ./mvnw package -Pnative -Dmaven.test.skip=true && \
    # Prepare everything for final image
    mkdir -p /dist && \
    cp /app/target/*-runner /dist/application

### Final image based on scratch containing only the binary
FROM scratch
COPY --chown=1000 --from=native-image /dist /work
# it is possible to add timezone, certificat and new user/group
# COPY --from=xxx /usr/share/zoneinfo /usr/share/zoneinfo
# COPY --from=xxx /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
# COPY --from=xxx /etc/passwd /etc/passwd
# COPY --from=xxx /etc/group /etc/group
EXPOSE 8080
USER 1000
WORKDIR /work/
CMD ["./application", "-Djava.io.tmpdir=/work/tmp"]
```

这种多级 docker 构建由 3 个部分组成:

1.  运行 maven dependencies plugin:
    创建一个包含所有 maven jars 的中间映像，作为下一个映像的缓存。
    这样，下次构建 docker 映像时，就不需要再次加载所有依赖项。
2.  创建二进制可执行文件:
    带有 [Graalvm 镜像](https://hub.docker.com/r/oracle/graalvm-ce/tags) +原生镜像目标 [quarkus-maven-plugin](https://quarkus.io/guides/maven-tooling) 它将生成一个 64 位 Linux 可执行文件。
    重要的部分在 [pom.xml](https://github.com/sokube/quarkus-scratch/blob/master/pom.xml#L100-L102) 中:为了创建一个可以在不包含 libc 的 linux 发行版上本地运行的 nativeImage(如 Alpine Linux……),“native-image”命令使用“additionalBuildArg”使用“- static”参数进行编译。
3.  使用基于 scratch 的映像创建最终映像:
    “scratch”是 Docker 的保留名称，用于指示构建过程跳过这一行，转到下一个 Dockerfile 命令。这不是一个你可以拉或运行的图像(尽管它[出现在 DockerHub 的库](https://hub.docker.com/_/scratch))。
    这是所有其他映像的基本祖先，但它是一个没有任何文件夹/文件、外壳、库的空映像……
    要了解 docker 如何解释基于“临时”的映像，请阅读[https://www.mgasch.com/post/scratch/](https://www.mgasch.com/post/scratch/)
    因为前面的步骤带有“- static”参数，所以可执行文件可以在“从临时”构建的 Docker 映像中运行……

# 构建并运行 docker 映像

要执行这种多阶段构建，请使用以下命令行:

```
docker build -t quarkus-app .
```

用 GraalVm 编译一个本地可执行文件需要很长时间，而且占用大量 CPU 资源。因此，最好将此过程委托给您的 CI/CD 管道。好消息是，在没有本地编译的情况下，在开发阶段，Quarkus 热重装非常有效。

要运行生成的图像:

```
docker run -it --rm --name quarkus -p 8888:8080 quarkus-app
```

它将生成以下输出:

```
2020-03-15 18:19:39,643 INFO  [io.quarkus] (main) getting-started 1.0-SNAPSHOT (running on Quarkus 1.2.1.Final) started in **0.028s**. Listening on: http:
//0.0.0.0:8080
2020-03-15 18:19:39,644 INFO  [io.quarkus] (main) Profile prod activated. 
2020-03-15 18:19:39,644 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]
```

请注意 0.028 秒的启动时间

您可以在您的浏览器中测试该应用程序:[http://localhost:8888/hello/greeting/SoKube](http://localhost:8888/hello/greeting/SoKube)

不要认为这是一个简单的 hello World 演示:在引擎盖下，Quarkus 框架使用 CDI 和 Resteasy，就像您使用需要服务于业务请求的真实 java 应用程序一样…

另一个有趣的测试是限制内存和 CPU:

```
docker run -it --rm --name quarkus -p 8888:8080 --cpus="0.05" --memory="4m" --memory-swap="4m" quarkus-app
```

使用 0.05 的 CPU 和 4m 的内存，应用程序在 2.503 秒内启动:

```
2020-03-15 18:35:18,137 INFO  [io.quarkus] (main) getting-started 1.0-SNAPSHOT (running on Quarkus 1.2.1.Final) started in **2.503s**. Listening on: [http://0.0.0.0:8080](http://0.0.0.0:8080)
2020-03-15 18:35:18,137 INFO  [io.quarkus] (main) Profile prod activated. 
2020-03-15 18:35:18,137 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]
```

就分配的资源而言，仍然令人印象深刻！

# Kubernetes Quarkus 部署

为了在 Kubernetes 上部署，我使用了一个本地的 [k3s 发行版](https://rancher.com/docs/k3s/latest/en/)和 [k3d](https://github.com/rancher/k3d) 。要了解更多信息，请参阅我以前的一篇文章“k3d + k3s = k8s 开发和测试的完美匹配”。

因此，首先创建 kubernetes 集群:

```
k3d create --name quarkus-cluster --api-port 6555 --publish 8085:80
export KUBECONFIG="$(k3d get-kubeconfig --name='quarkus-cluster')"
kubectl cluster-info
```

使用[https://github . com/sokube/quar kus-scratch/blob/master/deploy . YAML](https://github.com/sokube/quarkus-scratch/blob/master/deploy.yaml)部署应用程序

克隆这个 Git repo 并将目录更改为“quarkus-scratch”

```
k3d import-images --name quarkus-cluster quarkus-app
kubectl apply -f deploy.yaml
```

第一行导入 k3s 集群中之前创建的名为“quarkus-app”的映像。第二行部署应用程序。

然后，您应该能够使用:[http://localhost:8085/hello/greeting/SoKube](http://localhost:8085/hello/greeting/SoKube)访问该应用程序

# K8s 请求和限制

在 Pod 规范中，我定义了请求并限制资源最多使用 1 毫核 CPU (1000 毫核= 1 个 CPU)和 4Mi 内存

```
 resources:
          limits:
            memory: "4Mi"
            cpu: "1m"
          requests:
            cpu: "1m"
            memory: "4Mi"
```

命令“ *kubectl logs -l app=quarkus* ”显示了启动日志:

```
2020-03-17 10:42:26,239 INFO  [io.quarkus] (main) getting-started 1.0-SNAPSHOT (running on Quarkus 1.2.1.Final) started in **0.170s**. Listening on: [http://0.0.0.0:8080](http://0.0.0.0:8080)
2020-03-17 10:42:26,241 INFO  [io.quarkus] (main) Profile prod activated. 
2020-03-17 10:42:26,241 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]
```

尽管资源有限，仍然非常快！用你的 JEE 或 SpringBoot 应用程序:D 尝试这样的场景

# 扩展您的应用

凭借如此快速的启动和低内存占用，您可以在笔记本电脑上使用 50 个副本轻松扩展您的应用程序:

```
kubectl scale deployment/quarkus --replicas 50NAME      READY   UP-TO-DATE   AVAILABLE   AGE
quarkus   50/50   50           50          1h
```

然后很快地把它缩小:

```
kubectl scale deployment/quarkus --replicas 1NAME      READY   UP-TO-DATE   AVAILABLE   AGE
quarkus   1/1     1            1           1h
```

# 好吧，有趣，但是为什么呢？

部署这样的应用并不是为了“瓦霍效应”(至少不仅仅是！).

*   CPU 和内存是用于对云或内部环境收费的常见资源。这些资源不是无限的，账单可能会不可思议地增加。此外，它还存在像谷歌云平台(GCP)这样的低成本解决方案，在这个平台上，你可以使用“共享核心机器类型”来订购虚拟机。对于运行小型、非资源密集型应用程序来说，这可能是一种经济高效的方法，例如“E2-micro 2vCPU，1GB 内存”每月仅需 6.5 美元。
*   对于 Kubernetes 来说，快速启动和关闭以进行扩展和部署是一个关键方面。
    如果应用程序的启动时间很长，将迫使您将[探测器](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes)配置为高超时(除了从 k8s 1.16 开始，新的启动探测器用于第一次启动，然后使用活动和就绪探测器)。但在所有情况下，它将简化新应用程序版本的推出和回滚。
*   您的笔记本电脑或开发环境上的低内存占用使得本地开发更加容易。将像 k3s 这样的轻量级 Kubernetes 发行版与本地应用程序结合起来，就有可能拥有一个更接近真实生产环境的完整 k8s 平台，但是是本地的！
*   安全性是生产中的一个主要问题，在极简 docker 映像(没有外壳、文件、文件夹或库)中拥有一个专用的 64 位 Linux 可执行文件可以大大减少安全性问题。
    如多阶段构建文件所示，不要忘记保持其他安全最佳实践，比如不要成为 root…
*   无服务器架构和产品可以受益于真实应用程序的更快启动时间。当没有“热”容器可用时，通过“冷启动”按需执行功能。因此，在这些情况下，为了避免高延迟，包括第一次请求响应时间在内的非常快的启动时间非常重要。

# 完美？

依靠 Quarkus 使用原生编译并部署在 kubernetes 上是好的，但还不够！
你的应用需要被设计成一个[云本地应用](https://thenewstack.io/10-key-attributes-of-cloud-native-applications/) (CNA)，我不是在说微服务。你可以写一个 CNA 作为一个“普通”的服务，但是要遵守一些原则，避免，例如，你的应用程序的启动初始化在几分钟内加载一个巨大的缓存在内存中…你的应用程序的设计是非常重要的，没有一个框架，工具，产品可以弥补一个坏的设计！

本地编译的一个缺点是[限制](https://github.com/oracle/graal/blob/master/substratevm/REFLECTION.md)，尤其是在反射和动态类加载的使用方面。这使得将所有应用程序迁移到原生二进制文件变得更加困难(至少目前如此)，但是随着 Graal 的每个版本，兼容性都在提高。这就是为什么 Quarkus [支持有限的扩展列表](https://quarkus.io/guides/),但是它正在增长并且已经包含了很多扩展。

与极简 docker 形象相关的另一个方面是出道！
无法在容器中执行命令！那么如何调试一个不包含 shell，curl，wget…甚至 ls，chmod，chown，mkdir…这样的工具的镜像呢？
我不会详细说明如何实现这一点，但简单来说就是使用像 [busybox](https://hub.docker.com/_/busybox) 这样的图像，并从该图像向极简图像注入外壳和所需的工具…

# 结论

Quarkus 非常适合云时代，在这个时代，容器、Kubernetes、微服务或服务、功能即服务和原生为云设计的应用程序允许达到高水平的生产力和效率。

像 Kubernetes 这样的服务、即时可伸缩性和高密度平台要求应用程序具有较小的内存占用和快速启动。Java 并没有很好地定位，因为它以牺牲 CPU 和 RAM 为代价来支持处理时间。把 Quarkus 和 GraalVM 结合起来，就不再是这样了！

Kubernetes 将长期存在，所以让我们为最大的灵活性、效率和安全性准备好我们的开发、应用和平台。