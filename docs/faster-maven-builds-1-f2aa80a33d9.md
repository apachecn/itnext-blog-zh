# 更快的 Maven 构建

> 原文：<https://itnext.io/faster-maven-builds-1-f2aa80a33d9?source=collection_archive---------3----------------------->

![](img/0fc5c993e7592a09c961aecc15983c82.png)

构建需要一些属性，其中最主要的是可再现性。我认为速度是优先级中较低的。然而，这也是你的发布周期的最大限制因素之一:如果你的构建需要 *T* ，你不能比每个 *T* 发布得更快。因此，在达到一定的成熟度级别后，您可能希望加快构建速度，以支持更频繁的发布。

在这篇文章中，我想详细介绍一些你可以用来提高 Maven 构建速度的技术。下面这篇文章将重点介绍如何在 Docker 内部做同样的事情。

# 基线

因为我想提出技术并评估它们的影响，所以我们需要一个样本库。我选择了 [Hazelcast 代码示例](https://github.com/hazelcast/hazelcast-code-samples)，因为它提供了足够大的多模块代码库，包含许多子模块；确切的提交是 [448febd](https://github.com/hazelcast/hazelcast-code-samples/commit/448febd9977b9927a3f00bbf61ba50b2c0d94bb4) 。

规则如下:

*   我运行该命令五次，以避免临时问题
*   我在每次运行之间执行`mvn clean`,从一个空的`target`存储库开始
*   所有依赖项和插件都已下载
*   我在控制台日志中报告了 Maven 显示的时间:

```
[INFO] -------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] -------------------------------------------------------
[INFO] Total time:  22.456 s (Wall Clock)
[INFO] Finished at: 2021-09-24T23:20:41+02:00
[INFO] -------------------------------------------------------
```

让我们从我们的基线开始，`mvn test`。结果是:

*   02:00 分
*   01 时 57 分
*   01 时 58 分
*   01 时 56 分
*   01 时 58 分

# 使用所有 CPU

默认情况下，Maven 使用单线程。在多核时代，这只是浪费。通过设置一个绝对数字或相对于可用内核数量的数字，可以使用多线程运行并行构建。更多信息，请查看[相关文档](https://cwiki.apache.org/confluence/display/MAVEN/Parallel+builds+in+Maven+3)。

你拥有的相互不依赖的子模块越多，*也就是*，Maven 可以并行构建它们，你用这种技术就能实现得越好。它非常适合我们的代码库。

我们将使用尽可能多的可用内核线程。相关命令是`mvn test -T 1C`。

当该命令启动时，您应该会在控制台中看到以下消息:

```
Using the MultiThreadedBuilder implementation with a thread count of X
```

*   51.487 秒(挂钟)
*   40.322 秒(挂钟)
*   52.468 秒(挂钟)
*   41.862 秒(挂钟)
*   41.699 秒(挂钟)

数字要好得多，但方差更大。

# 并行测试执行

并行化是一种优秀的技术。我们可以在测试执行方面做同样的事情。默认情况下，Maven Surefire 插件按顺序运行测试，但是也可以将其配置为并行运行测试。请参考[文档](https://maven.apache.org/surefire/maven-surefire-plugin/examples/fork-options-and-parallel-execution.html#Parallel_Test_Execution)了解整套选项。

如果每个模块中有大量的单元，这种方法是非常好的。请注意，您的测试**需要**相互独立。

我们将手动设置线程数量:

```
mvn test -Dparallel=all -DperCoreThreadCount=false -DthreadCount=16 #1 #2
```

1.  配置 Surefire 并行运行类和方法
2.  手动将线程数改为 16

让我们运行它:

*   02 分 04 秒
*   02 分 03 秒
*   01:46 分钟
*   01 分 52 秒
*   01:53 分钟

似乎线程同步的成本抵消了运行并行测试的潜在收益。

# 脱机的

Maven 将在每次运行时检查一个`SNAPSHOT`依赖项是否有新的“版本”。这意味着额外的网络往返。我们可以用`--offline`选项来防止这种检查。

虽然您应该避免`SNAPSHOT`依赖，但有时这是不可避免的，尤其是在开发期间。

该命令为`mvn test -o`，`-o`为`--offline`的快捷方式。

*   01:46 分钟
*   01:46 分钟
*   01:47 分钟
*   01:55 分钟
*   01:44 分钟

代码库有相当多的`SNAPSHOT`依赖项；因此离线大大加快了构建速度。

# JVM 参数

Maven 本身是一个基于 Java 的应用程序。这意味着每次运行都会启动一个新的 JVM。JVM 首先解释*字节码* **和**，然后分析工作负载，并相应地将*字节码*编译成*本机代码*:这意味着性能达到峰值，但只是在一段(长)时间之后。它非常适合长时间运行的流程，但不太适合命令行应用程序。

在构建的环境中，我们可能不会达到最高的性能点，因为它们是相对短暂的，但是我们仍然在为分析成本买单。我们可以通过配置适当的 JVM 参数来配置 Maven 放弃它。有几种配置 JVM 的方法。最直接的方法是在项目文件夹的`.mvn`子文件夹中创建一个专用的`jvm.config`配置文件。

```
-XX:-TieredCompilation -XX:TieredStopAtLevel=1
```

现在让我们简单地运行`mvn test`:

*   01:44 分钟
*   01:44 分钟
*   01:53 分钟
*   01:53 分钟
*   01:55 分钟

# Maven 守护进程

Maven 守护进程是 Maven 生态系统的新成员。它从 [Gradle 守护进程](https://docs.gradle.org/current/userguide/gradle_daemon.html)中汲取灵感:

> *Gradle 运行在 Java 虚拟机(JVM)上，并使用几个支持库，这些库需要大量的初始化时间。因此，有时启动会显得有点慢。这个问题的解决方案是 Gradle Daemon:一个长寿的后台进程，它执行您的构建要比其他情况快得多。我们通过避免昂贵的引导过程和通过在内存中保存项目数据来利用缓存来实现这一点。*

Gradle 团队很早就认识到命令行工具并不是 JVM 的最佳用法。为了解决这个问题，需要保持一个 JVM 后台进程(守护进程)一直运行。它充当服务器，而 CLI 本身则扮演客户端的角色。

作为一个额外的好处，这样一个长时间运行的进程只加载类一次(如果它们在两次运行之间没有改变)。

一旦你安装了软件，你可以用`mvnd`命令运行守护进程，而不是标准的`mvn`命令。下面是`mvnd test`的结果:

*   33.124 秒(挂钟)
*   33.114 秒(挂钟)
*   34.440 秒(挂钟)
*   32.025 秒(挂钟)
*   29.364 秒(挂钟)

注意守护进程默认使用多线程，带有`number of cores - 1`。

# 混合和搭配

我们已经看到了几种加速构建的方法。如果我们联合使用它们会怎么样？

让我们首先尝试一下目前为止我们在同一次跑步中看到的每一项技术:

```
mvnd test -Dparallel=all -DperCoreThreadCount=false -DthreadCount=16 -o #1 #2 #3 #4
```

1.  使用 Maven 守护进程
2.  并行运行测试
3.  不更新`SNAPSHOT`依赖关系
4.  通过`jvm.config`文件如上配置 JVM 参数——不需要设置任何选项

该命令返回以下结果:

*   27.061 秒(挂钟)
*   24.457 秒(挂钟)
*   24.853 秒(挂钟)
*   25.772 秒(挂钟)

想想看，Maven 守护进程*是*一个长期运行的进程。出于这个原因，让 JVM 分析并编译从*字节码*到*本机代码*是合情合理的。因此，我们可以删除`jvm.config`文件并重新运行上面的命令。结果是:

*   23.840 秒(挂钟)
*   26.589 秒(挂钟)
*   22.283 秒(挂钟)
*   23.788 秒(挂钟)
*   22.456 秒(挂钟)

现在，我们可以显示整合后的结果:

[https://gist . github . com/nfrankel/e 60 AE 1a 08009370 db 98 b 87 C5 e 552 da 16](https://gist.github.com/nfrankel/e60ae1a08009370db98b87c5e552da16)

# 结论

在这篇文章中，我们看到了几种加速 Maven 构建的方法。总结如下:

*   **Maven 守护进程**:坚实、安全的起点
*   **并行化构建**:当构建包含多个相互独立的模块时
*   **并行测试**:当项目包含多个测试时
*   **离线**:当项目包含`SNAPSHOT`依赖项*和*时，您不需要更新它们
*   **JVM 参数**:当你想做得更多的时候

我建议每个用户开始使用 Maven 守护进程，并在必要时根据你的项目继续优化。

在下一篇文章中，我们将重点关注如何加速容器内的 Maven 构建。

**更进一步:**

*   [Maven 3 中的并行构建](https://cwiki.apache.org/confluence/display/MAVEN/Parallel+builds+in+Maven+3)
*   [万全并行测试执行](https://maven.apache.org/surefire/maven-surefire-plugin/examples/fork-options-and-parallel-execution.html#Parallel_Test_Execution)
*   [mvnd—Maven 守护进程](https://github.com/mvndaemon/mvnd)

*原载于* [*一个 Java 怪胎*](https://blog.frankel.ch/faster-maven-builds/1/)*2021 年 10 月 3 日*