# 处理 Docker 中运行的 Java 应用程序的死机问题

> 原文：<https://itnext.io/docker-and-java-handling-oomkilled-7dba9759df72?source=collection_archive---------0----------------------->

在 Docker 容器中运行 Java 应用程序时，经常会遇到致命错误。这个错误的主要原因是 JVM 错误地检测了 docker 容器中的内存。我们可以通过为 Java 应用程序设置正确的堆大小来解决这个问题。

首先，我们来看看为什么 JVM 会为我们的 Docker 容器检测到不准确的内存？

**当我们让 JVM 为它在容器中的堆大小设置默认值时，JVM 将分配运行容器的机器的物理内存的 1/4。**

这背后的公式是，

`MaxHeap = MaxRAM * 1 / MaxRAMFraction`

其中 MaxRAMFraction 默认为 4。因此，JVM 分配物理机 RAM 大小的 1/4 作为 JVM 进程的最大堆大小。因为 JVM 不知道 Linux 容器。它认为自己正在主机上运行，并且可以访问机器的最大 RAM 大小。

假设我们在一台拥有 4000MB 内存的机器上运行一个 docker 容器。然后我们给 docker 容器分配 250MB 内存。默认情况下，JVM 分配机器 1/4 的内存(在本例中是 1000MB)作为最大堆。当 JVM 试图使用 docker 容器内存之外的更多内存时，对容器中可用内存的无效检测会导致容器被杀死。所以 JVM 试图超越 200MB 的那一刻，就会被 docker 杀死。

通过为 JVM 进程设置最大堆大小，而不是让 JVM 选择默认值，我们可以很容易地解决这个问题。因此，我们可以相应地为我们的 Java 进程定义准确的 **-Xms** 和 **-Xmx** 值。例如，如果我们分配 250MB 给 docker 容器，我们可以在 docker 文件中设置如下 JAVA_OPTS，

```
JAVA_OPTS='-Xms128m -Xmx128m'
```

这将有助于 JVM 不超过 docker 容器内存，因为最大堆大小(128MB)小于容器内存(250MB)。在这种情况下，我们应该将 MaxHeap 值设置为一个仔细猜测的值。否则，JVM 进程将无法运行。

为 docker 容器中运行的 JVM 设置 Xms 和 Xmx 值是解决 OOMKilled 问题的最佳解决方案吗？绝对没有。

什么？没有吗？那么这里的问题是什么呢？

这意味着您需要控制内存两次，一次在 JVM 中，一次在 Docker 中。这不是一个理想的解决方案。应该有更好的方法来做这件事。否则，每次你需要改变的时候，你都要做两次。

Java 10+引入了它最伟大的特性之一**选择容器可用 RAM 大小的堆大小百分比。**

对 JAVA_OPTS 使用以下标志将根据容器内存调整 JVM 堆大小:

```
**To JVM to aware to read the size of the container memory instead of reading the RAM size of the host machine.** -XX:-UseContainerSupport**To set the** **percentage of real memory used for initial heap size**
-XX:InitialRAMPercentage=40 **To set the** **Maximum percentage of real memory used for maximum heap size**
-XX:MaxRAMPercentage=60 
```

假设上述场景的 docker 内存限制是 1GB，那么 JVM 根据上述标志将初始堆大小设置为 400MB，将最大堆大小设置为 600MB。

如果你使用的是 java 8 或 9，你就没有机会使用上面的标志了。但是不用担心。有一个解决方法。对于 Java 8u131 和 Java 9，您有以下选择:

```
**To JVM to aware to read the size of the container memory instead of reading the RAM size of the host machine.** -XX:+UnlockExperimentalVMOptions 
-XX:+UseCGroupMemoryLimitForHeap **To set the** **percentage of real memory used for initial heap size**
-XX:InitialRAMFraction=4**To set the** **Maximum percentage of real memory used for maximum heap size**
-XX:MaxRAMFraction=2
```

在这种情况下，如果 docker 内存限制是 1GB，那么 JVM 将初始堆大小设置为 250 MB(1GB 的 1/4 ),将最大堆大小设置为 500 MB(1GB 的 1/2)。java 10+的唯一缺点是我们不能使用一定百分比的内存，相反，我们必须使用一部分内存，而且这个部分必须是整数。

我建议将初始堆大小和最大堆大小设置为相同的值，因为这样可以防止堆扩展导致的暂停。这会导致在等待 JAVA 处理内存分配更改时防止性能问题。

当使用上述标志时，您不需要为 JVM 和 Docker 调整两次内存。这些标志将根据 docker 容器内存限制调整 JVM 的堆。干杯！！