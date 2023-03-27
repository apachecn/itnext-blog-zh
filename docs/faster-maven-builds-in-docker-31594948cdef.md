# Docker 中更快的 Maven 构建

> 原文：<https://itnext.io/faster-maven-builds-in-docker-31594948cdef?source=collection_archive---------3----------------------->

![](img/0fc5c993e7592a09c961aecc15983c82.png)

上周，我描述了[不同的技术](https://blog.frankel.ch/faster-maven-builds/1/)来加固你的 Maven 构建。今天，我想扩大范围，对 Docker 中的 Maven builds *做同样的事情。*

在每次运行之间，我们通过添加一个空行来更改源代码；在每个部分之间，我们删除所有构建的图像，包括作为多阶段构建结果的中间图像。这个想法是为了避免重用以前构建的映像。

# 基线

为了计算一个有用的基线，我们需要一个示例项目。我为此创建了一个 [one](https://github.com/nfrankel/fast-maven-builds) :这是一个相对较小的 Kotlin 项目。

下面是相关的`Dockerfile`:

```
FROM openjdk:11-slim-buster as build                         #1COPY .mvn .mvn                                               #2
COPY mvnw .                                                  #2
COPY pom.xml .                                               #2
COPY src src                                                 #2RUN ./mvnw -B package                                        #3FROM openjdk:11-jre-slim-buster                              #4COPY --from=build target/fast-maven-builds-1.0.jar .         #5EXPOSE 8080ENTRYPOINT ["java", "-jar", "fast-maven-builds-1.0.jar"]     #6
```

1.  从包装步骤的 JDK 图像开始
2.  添加所需资源
3.  创建罐子
4.  从 JRE 开始映像创建步骤
5.  复制上一步中的罐子
6.  设置入口点

让我们执行构建:

```
time DOCKER_BUILDKIT=0 docker build -t fast-maven:1.0 . #1
```

1.  暂时忘记环境变量，我将在下一节解释

以下是五次运行的结果:

```
* 0.36s user 0.53s system 0% cpu 1:53.06 total
* 0.36s user 0.56s system 0% cpu 1:52.50 total
* 0.35s user 0.55s system 0% cpu 1:56.92 total
* 0.36s user 0.56s system 0% cpu 2:04.55 total
* 0.38s user 0.61s system 0% cpu 2:04.68 total
```

# 赢得胜利的构建套件

最后一个命令行使用了`DOCKER_BUILDKIT`环境变量。这是告诉 Docker 使用传统引擎的方式。如果你有一段时间没有更新 Docker，这是你正在使用的引擎。如今， [BuildKit](https://github.com/moby/buildkit) 已经取代了它，成为新的默认设置。

BuildKit 带来了几项性能改进:

*   自动垃圾收集
*   并发依赖解析
*   高效的指令缓存
*   构建缓存导入/导出
*   等等。

让我们在新的引擎上重新执行前面的命令:

```
time docker build -t fast-maven:1.1 .
```

以下是第一次运行时控制台日志的摘录:

```
...
 => => transferring context: 4.35kB
 => [build 2/6] COPY .mvn .mvn
 => [build 3/6] COPY mvnw .
 => [build 4/6] COPY pom.xml .
 => [build 5/6] COPY src src
 => [build 6/6] RUN ./mvnw -B package
...0.68s user 1.04s system 1% cpu 2:06.33 total
```

相同命令的以下执行结果略有不同:

```
...
 => => transferring context: 1.82kB
 => CACHED [build 2/6] COPY .mvn .mvn
 => CACHED [build 3/6] COPY mvnw .
 => CACHED [build 4/6] COPY pom.xml .
 => [build 5/6] COPY src src
 => [build 6/6] RUN ./mvnw -B package
...
```

请记住，我们会在运行之间更改源代码。我们不改变的文件，即`.mvn`、`mvnw`和`pom.xml`，由 build kit*缓存*。但是这些资源很小，所以缓存不会显著改善构建时间。

```
* 0.69s user 1.01s system 1% cpu 2:05.08 total
* 0.65s user 0.95s system 1% cpu 1:58.51 total
* 0.68s user 0.99s system 1% cpu 1:59.31 total
* 0.64s user 0.95s system 1% cpu 1:59.82 total
```

快速浏览日志可以发现，构建中最大的瓶颈是所有依赖项(包括插件)的下载。每次我们修改源代码时都会发生这种情况。这就是 BuildKit 没有提升性能的原因。

# 层，层，层

我们应该把精力集中在从属关系上。为此，我们可以利用*层*并将构建分为两步:

*   第一步，我们下载依赖项
*   在第二个例子中，我们进行适当的包装

每一步都创建一个层，第二步依赖于第一步。

有了分层，如果我们改变第二层的源代码，第一层不受影响，可以重用。我们不需要再次下载依赖项。新的`Dockerfile`看起来像:

```
FROM openjdk:11-slim-buster as buildCOPY .mvn .mvn
COPY mvnw .
COPY pom.xml .RUN ./mvnw -B dependency:go-offline                          #1COPY src srcRUN ./mvnw -B package                                        #2FROM openjdk:11-jre-slim-busterCOPY --from=build target/fast-maven-builds-1.2.jar .EXPOSE 8080ENTRYPOINT ["java", "-jar", "fast-maven-builds-1.2.jar"]
```

1.  目标`go-offline`下载所有的依赖项和插件
2.  此时，所有依赖项都是可用的

注意`go-offline`并不下载所有东西。如果您尝试使用`-o`选项(用于离线)，该命令将不会成功运行。是[一个众所周知的老 bug](https://issues.apache.org/jira/browse/MDEP-82) 。在所有情况下，都是“足够好”。

让我们运行构建:

```
time docker build -t fast-maven:1.2 .
```

第一次运行花费的时间比基线长得多:

```
0.84s user 1.21s system 1% cpu 2:35.47 total
```

但是，后续的构建要快得多。更改源代码只会影响第二层，不会触发(大多数)依赖项的下载:

```
* 0.23s user 0.36s system 5% cpu 9.913 total
* 0.21s user 0.33s system 5% cpu 9.923 total
* 0.22s user 0.38s system 6% cpu 9.990 total
* 0.21s user 0.34s system 5% cpu 9.814 total
* 0.22s user 0.37s system 5% cpu 10.454 total
```

# 构建中的卷装载

构建分层极大地改善了构建时间。我们可以改变源代码，并保持低。不过，还有一个问题。更改单个依赖关系会使该层无效，因此我们需要再次下载所有依赖关系。

幸运的是，BuildKit 在构建期间引入了**卷挂载(而不仅仅是在运行期间)。有几种类型的挂载可供选择，但我们感兴趣的是[缓存挂载](https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/syntax.md#run---mounttypecache)。这是一个*实验性的*特性，所以你需要明确地选择加入:**

```
# syntax=docker/dockerfile:experimental                      #1
FROM openjdk:11-slim-buster as buildCOPY .mvn .mvn
COPY mvnw .
COPY pom.xml .
COPY src srcRUN --mount=type=cache,target=/root/.m2,rw ./mvnw -B package #2FROM openjdk:11-jre-slim-busterCOPY --from=build target/fast-maven-builds-1.3.jar .EXPOSE 8080ENTRYPOINT ["java", "-jar", "fast-maven-builds-1.3.jar"]
```

1.  选择加入实验功能
2.  使用缓存构建

是时候运行构建了:

```
time docker build -t fast-maven:1.3 .
```

构建时间高于常规构建，但仍低于层构建:

```
0.71s user 1.01s system 1% cpu 1:50.50 total
```

以下构件与层相当:

```
* 0.22s user 0.33s system 5% cpu 9.677 total
* 0.30s user 0.36s system 6% cpu 10.603 total
* 0.24s user 0.37s system 5% cpu 10.461 total
* 0.24s user 0.39s system 6% cpu 10.178 total
* 0.24s user 0.35s system 5% cpu 10.283 total
```

然而，与层相反，我们只需要下载更新的依赖关系。这里，我们把科特林的版本从`1.5.30`改成`1.5.31`:

```
<properties>
    <kotlin.version>1.5.31</kotlin.version>
</properties>
```

就构建时间而言，这是一个巨大的改进:

```
* 0.41s user 0.57s system 2% cpu 44.710 total
```

# 考虑到 Maven 守护进程

在之前关于[常规 Maven 构建](https://blog.frankel.ch/faster-maven-builds/1/)的帖子中，我提到了 Maven 守护进程。让我们相应地改变我们的构建:

```
FROM openjdk:11-slim-buster as buildADD https://github.com/mvndaemon/mvnd/releases/download/0.6.0/mvnd-0.6.0-linux-amd64.zip . #1RUN apt-get update \                                         #2
 && apt-get install unzip \                                  #3
 && mkdir /opt/mvnd \                                        #4
 && unzip mvnd-0.6.0-linux-amd64.zip \                       #5
 && mv mvnd-0.6.0-linux-amd64/* /opt/mvnd                    #6COPY .mvn .mvn
COPY mvnw .
COPY pom.xml .
COPY src srcRUN /opt/mvnd/bin/mvnd -B package                            #7FROM openjdk:11-jre-slim-busterCOPY --from=build target/fast-maven-builds-1.4.jar .EXPOSE 8080ENTRYPOINT ["java", "-jar", "fast-maven-builds-1.4.jar"]
```

1.  下载最新版本的 Maven 守护程序
2.  刷新包索引
3.  安装`unzip`
4.  创建专用文件夹
5.  提取我们在步骤 1 中下载的归档文件
6.  将提取的归档文件的内容移动到以前创建的文件夹中
7.  使用`mvnd`代替 Maven 包装器

让我们现在运行构建:

```
docker build -t fast-maven:1.4 .
```

该日志输出以下内容:

```
* 0.70s user 1.01s system 1% cpu 1:51.96 total
* 0.72s user 0.98s system 1% cpu 1:47.93 total
* 0.66s user 0.93s system 1% cpu 1:46.07 total
* 0.76s user 1.04s system 1% cpu 1:50.35 total
* 0.80s user 1.18s system 1% cpu 2:01.45 total
```

与基线相比，没有明显的改善。

我试图创建一个专用的`mvnd`图像，并将其用作父图像:

```
# docker build -t mvnd:0.6.0 .
FROM openjdk:11-slim-buster as buildADD https://github.com/mvndaemon/mvnd/releases/download/0.6.0/mvnd-0.6.0-linux-amd64.zip .RUN --mount=type=cache,target=/var/cache/apt,rw apt-get update \
 && apt-get install unzip \
 && mkdir /opt/mvnd \
 && unzip mvnd-0.6.0-linux-amd64.zip \
 && mv mvnd-0.6.0-linux-amd64/* /opt/mvnd# docker build -t fast-maven:1.5 .
FROM mvnd:0.6.0 as build# ...
```

这种方法以任何显著的方式改变了输出。

`mvnd`只有在守护进程多次运行时才是好的。我找不到和 Docker 一起做的方法。如果你对如何实现它有任何想法，请告诉我；额外加分，如果你能给我指出一个实现。

# 结论

在 Docker 中加速 Maven 构建的性能与常规构建有很大不同。在 Docker 中，限制因素是依赖项的下载速度。如果您停留在旧版本上，您需要使用层来缓存依赖项。

对于 BuildKit，我建议使用新的缓存挂载功能，以避免在层失效时下载所有依赖项。

这篇文章的完整源代码可以在 [Github](https://github.com/ajavageek/fast-maven-builds) 上以 Maven 格式找到。

**更进一步:**

*   [更快的 Maven 构建](https://blog.frankel.ch/faster-maven-builds/1/)
*   [介绍 BuildKit](https://blog.mobyproject.org/introducing-buildkit-17e056cc5317)
*   [Docker 层解释](https://mydeveloperplanet.com/2019/03/13/docker-layers-explained/)
*   [建造坐骑](https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/syntax.md#build-mounts-run---mount)

*原载于* [*一个 Java 极客*](https://blog.frankel.ch/faster-maven-builds/2/)*2021 年 10 月 10 日*