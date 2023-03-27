# 使用 Docker Compose 进行树莓 Pi 集群仿真

> 原文：<https://itnext.io/raspberry-pi-cluster-emulation-with-docker-compose-2d5583cd645?source=collection_archive---------0----------------------->

## 码头深水潜水

## 不含 SD 卡

# TL；速度三角形定位法(dead reckoning)

本指南讨论了使用 QEMU、Docker、Docker Compose 和 Ansible 模拟一个简单、可伸缩、完全二进制兼容的 Raspberry Pi 集群(在云中或其他地方)所需的一切。有关更新的代码和其他细节，请参见 GitHub 上的 [pidoc](https://github.com/mrhavens/pidoc) 库。

![](img/49272232555662fee15b63d880897d01.png)

作者图片

# 介绍

Raspberry Pi 不再只是一个供学生学习计算的低成本平台，它现在是一个合法的研发平台，用于物联网、网络、分布式系统和软件开发。它甚至在生产环境中被管理性地使用。

在 2012 年第一个 Raspberry Pi 发布后不久，一些公司开始将它们构建成低成本集群，通常是为了研究和测试目的。 [DataStax](https://www.datastax.com/) 的实习生建立了一个多数据中心，32 个节点 [Cassenda](https://cassandra.apache.org/) 容错演示，配有一个大红色按钮来模拟整个数据中心的故障。David Guill 构建了一个 [40 节点的 Raspberry Pi 集群](https://web.archive.org/web/20180816070042/http://likemagicappears.com/projects/raspberry-pi-cluster/)，这是他的 MSCE 论文的一部分。 [Balena](https://www.balena.io/) ，构建了“ [The Beast](https://www.balena.io/blog/what-would-you-do-with-a-120-raspberry-pi-cluster/) ”，一个 120 节点的 Raspberry Pi 集群，用于其在线平台的规模测试。在光谱的极端，甲骨文构建了一个 [1060 节点的 Raspberry Pi 集群](https://www.servethehome.com/oracle-shows-1060-raspberry-pi-supercomputer-at-oow/)，他们在甲骨文全球大会 2019 上推出了该集群。

随着 Raspberry Pi 被应用于从 wi-fi 扩展器到安全摄像头，甚至更大的集群，其创新仍在继续。虽然这些集群的主要价值在于它们的规模和低成本，但它们的普及使它们成为越来越常见的开发平台。由于 Raspberry Pi 使用 ARM 处理器，这可能会给我们这些专门在云中工作的人带来开发问题。虽然存在商业解决方案，但我们将使用托管在 Google Compute Engine 上的完全开源的堆栈来构建我们自己的仿真集群。

# 用例

除了从经验中学习之外，将一个仿真的 Raspberry Pi 编写成 Dockerizing 使我们能够做三件事。第一，它变成了软件，否则它将是一个没有人需要记住随身携带的纯硬件设备(我总是丢失外围电缆)。第二，它使 Docker 能够为 Pi 做 Docker 在其他方面做得最好的事情:它使软件可移植、易于管理、易于复制。第三，它不占用物理空间。如果我们可以用 Docker 构建一个树莓 Pi，我们就可以构建许多。如果我们能建造很多，我们可以把它们都连接起来。虽然我们可能会遇到一些限制，但这个构建将模拟一个 Raspberry Pi 1s 集群，该集群在逻辑上等同于一个简单的多节点物理集群。

# 仿真硬件架构

虽然技术上不完全相同，但是我们将使用的仿真软件 QEMU 提供了一个与 Raspberry Pi 1 大致兼容的`ARM-Versatile`架构。为了让它能够与 Raspbian 一起正常工作，对内核进行一些修改是必要的，但对于我们的目的来说，这是可用的更稳定的开源解决方案之一。

```
pi@raspberrypi:~$ cat /proc/cpuinfo 
processor       : 0
model name      : ARMv6-compatible processor rev 7 (v6l)
BogoMIPS        : 577.53
Features        : half thumb fastmult vfp edsp java tls
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x0
CPU part        : 0xb76
CPU revision    : 7

Hardware        : ARM-Versatile (Device Tree Support)
Revision        : 0000
Serial          : 0000000000000000
```

与实际的 Raspberry Pi 1 相比，它们几乎完全相同:

```
pi@raspberrypi:~ $ cat /proc/cpuinfo
processor	: 0
model name	: ARMv6-compatible processor rev 7 (v6l)
BogoMIPS	: 697.95
Features	: half thumb fastmult vfp edsp java tls 
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xb76
CPU revision	: 7

Hardware : BCM2835
Revision : 000d
Serial  : 000000003d9a54c5
```

# 背景

## QEMU 是什么？

QEMU 是一个处理器仿真器。它支持许多不同的处理器，但我们唯一感兴趣的是能够毫无困难地运行 Raspberry Pi 映像的东西。在这种情况下，我们将使用 QEMU 4.2.0，它支持 ARM11 指令集，与 Raspberry Pi 1 和 Zero 上的 Broadcom BCM2835 (ARM1176JZFS)芯片兼容。我们将在 QEMU 上使用 ARM1176 支持，这将允许我们或多或少地模拟一个 Raspberry Pi 1。我说或多或少是因为我们仍然需要使用一个定制的 Raspbian 内核，以便在仿真硬件上引导。QEMU 对 Pi 的支持仍在开发中，所以我们让它在这里工作的方法只是一个聪明的黑客，从 CPU 利用率的角度来看，它决不会是最优的或高效的。

## QEMU 特性

QEMU 支持 Docker 中的许多相同特性，但是，它可以在没有主机内核驱动程序的情况下运行完整的软件仿真。这意味着它可以在 Docker 或任何其他虚拟机中运行，而无需主机虚拟化支持。QEMU 特性列表非常广泛，学习曲线也非常陡峭。但是，我们在此版本中将重点关注的主要功能是主机端口转发，以便数据可以传递到主机。

## Dockerized QEMU

Docker 的优势之一是它不处理成熟的虚拟化，而是依赖于主机系统的架构。由于我们的主机系统将运行英特尔处理器，我们不能指望 Docker 自己处理 ARM 操作。因此，我们将把 QEMU 放在 Docker 容器中。由于 Docker 被设计为以接近本机的性能运行软件，因此运营效率的挑战将来自 QEMU 本身。另一方面，QEMU 完全用软件支持机器架构的仿真。这样做的好处是，它可以在任何虚拟化系统或容器中运行，与其系统架构无关。如果有耐心，我们甚至可以在另一个 dockered Raspberry Pi 容器中运行一个 dockered Raspberry Pi 容器。QEMU 的缺点是，与其他类型的虚拟化相比，它的性能相对较差。但是，我们可以通过利用 QEMU 的 ARM 仿真，同时依赖 Docker 来处理其他所有事情，从而从两个世界中获益。

## 拉斯比安

基于 Debian， [Raspbian](https://www.raspbian.org/) 是一个流行的、受到良好支持的 Raspberry Pi 操作系统，也是该平台最常推荐的操作系统之一。[社区](https://www.raspberrypi.org/forums/viewforum.php?f=66)非常活跃，管理良好。

## 物理树莓 Pi 速度比较

以下测试旨在作为比较我们的虚拟化系统的基准。因为我们将模拟单核，所以这些测试只是单核、单线程的，不管架构中包含多少物理内核。

## 树莓 Pi 1 2011 年 1 2 月

```
Test execution summary:
    total time:                          330.5514s
    total number of events:              10000
    total time taken by event execution: 330.5002
    per-request statistics:
         min:                                 32.92ms
         avg:                                 33.05ms
         max:                                 40.94ms
         approx.  95 percentile:              33.24ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   330.5002/0.00
```

## 树莓 Pi 1 A+2014v 1.1

```
Test execution summary:
    total time:                          328.7505s
    total number of events:              10000
    total time taken by event execution: 328.6931
    per-request statistics:
         min:                                 32.71ms
         avg:                                 32.87ms
         max:                                 78.93ms
         approx.  95 percentile:              33.03ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   328.6931/0.00
```

## 树莓派零度 W v1.1 2017

```
Test execution summary:
    total time:                          228.2025s
    total number of events:              10000
    total time taken by event execution: 228.1688
    per-request statistics:
         min:                                 22.76ms
         avg:                                 22.82ms
         max:                                 35.29ms
         approx.  95 percentile:              22.94ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   228.1688/0.00
```

## 树莓 Pi 2 型号 B v1.1

```
Test execution summary:
    total time:                          224.9052s
    total number of events:              10000
    total time taken by event execution: 224.8738
    per-request statistics:
         min:                                 22.20ms
         avg:                                 22.49ms
         max:                                 32.85ms
         approx.  95 percentile:              22.81ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   224.8738/0.00
```

## 树莓 Pi 3 型号 B v1.2 2015

```
Test execution summary:
    total time:                          139.6140s
    total number of events:              10000
    total time taken by event execution: 139.6087
    per-request statistics:
         min:                                 13.94ms
         avg:                                 13.96ms
         max:                                 34.06ms
         approx.  95 percentile:              13.96ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   139.6087/0.00
```

## 树莓 Pi 4 B 2018

```
Test execution summary:
    total time:                          92.6405s
    total number of events:              10000
    total time taken by event execution: 92.6338
    per-request statistics:
         min:                                  9.22ms
         avg:                                  9.26ms
         max:                                 23.50ms
         approx.  95 percentile:               9.27ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   92.6338/0.00
```

# 项目要求

## 单一主机规格

历史上，QEMU 是单线程的，在单个 CPU 上模拟系统架构的所有内核。虽然情况不再如此，但我们仍将模拟单核的 Raspberry Pi。稍后我们将进行一些基准测试，比较每个节点上不同的 CPU 限制对性能的影响。但是现在，我们将在每个单核节点上使用一个 CPU。由于 QEMU 固有的低效率，它有可能使用大量 CPU 资源，因此我们最初的三节点集群将从每个节点至少一个 CPU 的基线开始，留下一个 CPU 专用于主机以避免性能问题。为此任务选择的虚拟机规格如下。

```
Cloud Provider: Google Cloud Platform
Instance Type: n1-standard-4
CPUs: 4
Memory: 15GB
Disk: 100GB
Operating System: Ubuntu 18.04 LTS
```

## 码头工人

安装在主机上，我们还使用默认版本的 Docker，它可以在 Ubuntu 18.04 LTS 的默认 apt 存储库中找到。

```
# docker -v
Docker version 18.09.7, build 2d0083d
```

## Docker 撰写

```
# docker-compose -v
docker-compose version 1.25.0, build 0a186604
```

## Docker Hub Ubuntu 图像

```
18.04, bionic-20200112, bionic, latest
```

## QEMU

安装在 Docker 容器中，我们将使用以下版本的 QEMU for ARM:

```
# qemu-system-arm --version
QEMU emulator version 4.2.0
Copyright (c) 2003-2019 Fabrice Bellard and the QEMU Project developers
```

## 为 Raspbian 定制的 QEMU 内核

从 Docker 内部的 QEMU 加载，我们将使用 [Dhruv Vyas](https://github.com/dhruvvyas90) 的[编译内核](https://github.com/dhruvvyas90/qemu-rpi-kernel)用于 Raspbian，它已经被修改为可用于 QEMU。

## 拉斯比简装图像

同样从 QEMU 启动，我们将使用 2019 年 9 月 30 日的未修改版本的 [Raspbian Lite](https://downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2019-09-30/) 。

## 期望(Tcl/Tk)

Docker 容器上安装了以下版本的 Expect:

```
# expect -v
expect version 5.45.4
```

## `ssh` / `sshd`

需要在每个 Raspbian 节点上启用`sshd`，并且应该在主机上启用`ssh`。

## Ansible

Ansible 的以下版本及其其他依赖项也在使用中:

```
# ansible --version
ansible 2.5.1
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.17 (default, Nov  7 2019, 10:07:09) [GCC 7.4.0]
```

# 构建 Docker 图像

## QEMU 构建容器

我们将从源代码编译 QEMU 4.2.0。它将需要所有的支持构建工具，所以为了让我们的应用程序容器尽可能小，我们将使用 Docker Hub 的 Ubuntu 18.04 的最小版本为 QEMU 构建创建一个单独的构建容器。

## QEMU 应用程序容器

一旦 QEMU 从源代码编译完成，我们将把它转移到 app 容器中。同样从 Docker Hub，我们将使用相同的 Ubuntu 18.04 最小版本来托管 QEMU 二进制文件。

# Docker 配置

## `Dockerfile`

我们将使用下面的 Dockerfile 文件，它也可以在 Github 上本指南附带的 [pidoc](https://github.com/mrhavens/pidoc) 库中找到。下面的每个代码片段构成了 docker 文件的一部分。感谢[卢克·蔡尔德](https://github.com/lukechilds)为 [dockerpi](https://github.com/lukechilds/dockerpi) 所做的工作。

`qemu-system-arm`的构建阶段:

```
FROM ubuntu AS qemu-system-arm-builder
ARG QEMU_VERSION=4.2.0
ENV QEMU_TARBALL="qemu-${QEMU_VERSION}.tar.xz"
WORKDIR /qemu
```

```
RUN apt-get update && \
    apt-get -y install \
                       wget \
                       gpg \
                       pkg-config \
                       python \
                       build-essential \
                       libglib2.0-dev \
                       libpixman-1-dev \
                       libfdt-dev \
                       zlib1g-dev \
                       flex \
                       bison
```

下载源。

```
RUN wget "https://download.qemu.org/${QEMU_TARBALL}"

RUN # Verify signatures...
RUN wget "https://download.qemu.org/${QEMU_TARBALL}.sig"
RUN gpg --keyserver keyserver.ubuntu.com --recv-keys CEACC9E15534EBABB82D3FA03353C9CEF108B584
RUN gpg --verify "${QEMU_TARBALL}.sig" "${QEMU_TARBALL}"
```

提取 tarball。

```
RUN tar xvf "${QEMU_TARBALL}"
```

构建源代码。

```
RUN "qemu-${QEMU_VERSION}/configure" --static --target-list=arm-softmmu
RUN make -j$(nproc)
RUN strip "arm-softmmu/qemu-system-arm"
```

构建中间 pidoc 虚拟机应用程序映像。

```
FROM ubuntu as pidoc-vm
ARG RPI_KERNEL_URL="https://github.com/dhruvvyas90/qemu-rpi-kernel/archive/afe411f2c9b04730bcc6b2168cdc9adca224227c.zip"
ARG RPI_KERNEL_CHECKSUM="295a22f1cd49ab51b9e7192103ee7c917624b063cc5ca2e11434164638aad5f4"
```

将二进制文件从构建容器传输到应用程序容器。

```
COPY --from=qemu-system-arm-builder /qemu/arm-softmmu/qemu-system-arm /usr/local/bin/qemu-system-arm
```

下载修改后的内核并安装。

```
ADD $RPI_KERNEL_URL /tmp/qemu-rpi-kernel.zip
```

```
RUN apt-get update && \
    apt-get -y install \
                        unzip \
                        expect
RUN cd /tmp && \
    echo "$RPI_KERNEL_CHECKSUM  qemu-rpi-kernel.zip" | sha256sum -c && \
    unzip qemu-rpi-kernel.zip && \
    mkdir -p /root/qemu-rpi-kernel && \
    cp qemu-rpi-kernel-*/kernel-qemu-4.19.50-buster /root/qemu-rpi-kernel/ && \
    cp qemu-rpi-kernel-*/versatile-pb.dtb /root/qemu-rpi-kernel/ && \
    rm -rf /tmp/*VOLUME /sdcard
```

然后，我们从主机的主目录中复制入口点脚本。

```
ADD ./entrypoint.sh /entrypoint.sh
ENTRYPOINT ["./entrypoint.sh"]
```

使用加载的 Raspbian Lite 文件系统构建最终的 app pidoc 映像。

```
FROM pidoc-vm as pidoc
ARG FILESYSTEM_IMAGE_URL="http://downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2019-09-30/2019-09-26-raspbian-buster-lite.zip"
ARG FILESYSTEM_IMAGE_CHECKSUM="a50237c2f718bd8d806b96df5b9d2174ce8b789eda1f03434ed2213bbca6c6ff"

ADD $FILESYSTEM_IMAGE_URL /filesystem.zip
ADD pi_ssh_enable.exp /pi_ssh_enable.exp

RUN echo "$FILESYSTEM_IMAGE_CHECKSUM  /filesystem.zip" | sha256sum -c
```

## `entrypoint.sh`文件

首先，脚本确定文件系统是否已经下载，如果没有，它下载并解压缩文件系统。

```
#!/bin/sh

raspi_fs_init() {
  image_path="/sdcard/filesystem.img"
  zip_path="/filesystem.zip"

  if [ ! -e $image_path ]; then
    echo "No filesystem detected at ${image_path}!"
    if [ -e $zip_path ]; then
        echo "Extracting fresh filesystem..."
        unzip $zip_path
        mv *.img $image_path
        rm $zip_path
    else
      exit 1
    fi
  fi
}
```

然后，该脚本检查一个空的`raspi-init`文件，该文件作为一个标记来确定之前是否启动了 Expect 来在 Raspbian 上启用 ssh。

```
if [ ! -e /raspi-init ]; then
  touch /raspi-init
  raspi_fs_init
  echo "Initiating Expect..."
  /usr/bin/expect /pi_ssh_enable.exp `hostname -I`
  echo "Expect Ended..."
```

如果之前已经启用了 Expect，那么我们只需要启动 QEMU，而不需要 Expect。注意，我们将 Raspbian 上的端口`22`转发到 Docker 容器内的端口`2222`。

```
else
  /usr/local/bin/qemu-system-arm \
        --machine versatilepb \
        --cpu arm1176 \
        --m 256M \
        --hda /sdcard/filesystem.img \
        --net nic \
        --net user,hostfwd=tcp:`hostname -I`:2222-:22 \
        --dtb /root/qemu-rpi-kernel/versatile-pb.dtb \
        --kernel /root/qemu-rpi-kernel/kernel-qemu-4.19.50-buster \
        --append "root=/dev/sda2 panic=1" \
        --no-reboot \
        --display none \
        --serial mon:stdio
fi
```

## 在 Raspbian 上启用 SSHD(除了 Tcl/Tk 方法)

QEMU 没有一种直接的方法在引导时运行配置脚本。因为 Raspbian 默认情况下没有启用 SSH，所以我们必须自己打开它。我们的选择是手动完成，或者使用某种可以与`stdio`交互的脚本工具。另一种选择是在安装之前定制 Raspbian 映像。然而，这必须在主机上完成，因为 Docker 限制了新文件系统的挂载。在任何情况下，为了使这个构建具有最大的可移植性和主机独立性，最简单的方法就是使用一个 [Expect](https://en.wikipedia.org/wiki/Expect) 脚本，并在构建时将它复制到我们的 Docker 映像中。

## `pi_ssh_enable.exp`文件

由于未修改的 Raspbian 映像默认没有可访问的端口，我们将使用 Expect 与 QEMU 中的`stdio`进行交互，使用默认用户名和密码登录，并启用`sshd`监听器。

```
#!/usr/bin/expect -f
set ipaddr [lindex $argv 0]
set timeout -1
spawn /usr/local/bin/qemu-system-arm \
  --machine versatilepb \
  --cpu arm1176 \
  --m 256M \
  --hda /sdcard/filesystem.img \
  --net nic \
  --net user,hostfwd=tcp:$ipaddr:2222-:22 \
  --dtb /root/qemu-rpi-kernel/versatile-pb.dtb \
  --kernel /root/qemu-rpi-kernel/kernel-qemu-4.19.50-buster \
  --append "root=/dev/sda2 panic=1" \
  --no-reboot \
  --display none \
  --serial mon:stdio
expect "raspberrypi login:"
send -- "pi\r"
expect "Password:"
send -- "raspberry\r"
expect "pi@raspberrypi:"
send -- "sudo systemctl enable ssh\r"
expect "pi@raspberrypi:"
send -- "sudo systemctl start ssh\r"
expect "pi@raspberrypi:"
expect eof
```

## 构建图像

在带有`Dockerfile`的文件夹中，我们将构建我们的两个容器。第一个将是我们的构建容器，其中包含编译 QEMU 的所有依赖项，另一个将是我们运行 QEMU 的应用程序容器。

```
docker build -t pidoc .
```

## 网络转发和故障排除

构建完成后，将其分离出来并遵循日志。

```
docker run -itd --name testnode pidoc
docker logs testnode -f
```

Raspbian 将自动下载和解压缩，QEMU 应该开始从映像启动。

一旦 Raspbian 完全启动，Expect 应该会自动启用`sshd`。登录 docker 容器，测试 SSH 是否可以从容器内部的端口`2222`上到达。

```
# docker exec -it testnode bash
root@d4abc2f655e6:/# hostname -I
172.17.0.3
root@d4abc2f655e6:/# cat < /dev/tcp/172.17.0.3/2222
SSH-2.0-OpenSSH_7.9p1 Raspbian-10
```

取消和杀死容器并移除体积。

```
root@d4abc2f655e6:/# exit
exit
# docker kill testnode
testnode
# docker container rm testnode
testnode
```

# 测试 Docker 容器

## 开始/测试容器

我们需要启动容器进行测试。这主要是为了获得关于 QEMU 性能的一些直觉，以便我们能够更好地做出关于我们的集群的设计决策。系统应该会干净地出现一些良性警告，这些警告与更一般化的仿真硬件和预期的物理 raspberry Pi 硬件之间的差异有关。我发现有必要确保 QEMU 和 Docker 映像之间的端口转发工作正常，这样我就可以进一步验证 Docker 映像和主机之间的端口转发工作正常。我们的第一个目标是双转发 SSH，这样就可以从主机直接访问 QEMU。

```
docker run -itd -p 127.0.0.1:2222:2222 --name testnode pidoc
docker logs testnode -f
```

一旦系统再次联机，通过使用`ssh`登录到 Raspbian 来测试主机端口`2222`上的`sshd`:

```
# ssh pi@localhost -p 2222
The authenticity of host '[localhost]:2222 ([127.0.0.1]:2222)' can't be established.
ECDSA key fingerprint is SHA256:N0oRF23lpDOFjlgYAbml+4v2xnYdyrTmBgaNUjpxnFM.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[localhost]:2222' (ECDSA) to the list of known hosts.
pi@localhost's password:
Linux raspberrypi 4.19.50+ #1 Tue Nov 26 01:49:16 CET 2019 armv6l
```

```
The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/\*/copyright.Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Jan 21 12:24:59 2020SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.pi@raspberrypi:~ $
```

## 测试部分 CPU 利用率

为了运行这个集群，我们使用运行 Ubuntu 18.04 LTS 版的 GCP n1-standard-4 实例(4x15)。但是我们现在注意到一旦拉斯边开始做任何事情，QEMU 是多么的低效。多个 Raspberry Pi 实例在空闲时可能会堆叠得很好，但是如果我们希望保持系统的可行性，我们将需要限制每个实例上的 CPU 利用率，否则一旦有多个节点处于负载之下，系统就会变得不可用。幸运的是，Docker 可以为我们处理这些。我们在这个实例上有 15GB 的 ram，所以让我们看看如果我们稍微大胆一点，将 6 个 Raspberry Pi 容器放到我们的 VM 上会发生什么。我们将有一个完整的核心留给主机来管理其他任务，而不会有太大的失败风险。我们可以在以后用 Docker Compose 来扩展它。

我们将运行 50%和 100%的两个测试容器进行基准测试。

```
docker run -itd --cpus="0.50" -p 127.0.0.1:2250:2222 --name pidoc_50_test pidoc
docker run -itd --cpus="1.00" -p 127.0.0.1:2200:2222 --name pidoc_00_test pidoc
```

此时，从技术上讲，我们已经有了一个集群。除了手工操作，我们没有其他方法来管理它们。

## 表演

当一个完整的内核分配以接近物理 Raspberry Pi 的速度执行时，一个运行 50%的实例的运行速度只有这个速度的一半。在某些情况下，这可能是可以管理的，但不是最理想的。根据手头的任务，群集的整体效率可能会提高。但是现在，我们将继续我们最初的 3 个节点的全核心分配，然后用 6 个节点进行测试。

## 单线程基准测试

可以通过使用以下简单的基准测试来完成测试。

**CPU 主要测试**

```
sysbench --test=cpu --cpu-max-prime=9999 run
```

**CPU 整数测试**

```
time $(i=0; while ((i<9999999)); do ((i++)); done)
```

**硬盘读取测试**

```
dd bs=16K count=102400 iflag=direct if=test_data of=/dev/null
```

**硬盘写入测试**

```
dd bs=16k count=102400 oflag=direct if=/dev/zero of=test_data
```

## 结果(单线程)

对于本指南，我们将只关注使用`sysbench`的 CPU 主要测试

**主持人**

```
General statistics:
    total time:                          10.0009s
    total number of events:              9417

Latency (ms):
         min:                                  1.04
         avg:                                  1.06
         max:                                  1.63
         95th percentile:                      1.10
         sum:                               9992.36

Threads fairness:
    events (avg/stddev):           9417.0000/0.00
    execution time (avg/stddev):   9.9924/0.00
```

**虚拟树莓 Pi —限制:100%**

```
Test execution summary:
    total time:                          397.8781s
    total number of events:              10000
    total time taken by event execution: 397.4056
    per-request statistics:
         min:                                 38.61ms
         avg:                                 39.74ms
         max:                                 57.15ms
         approx.  95 percentile:              40.92ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   397.4056/0.00
```

**虚拟树莓派—限制:50%**

```
Test execution summary:
    total time:                          823.8272s
    total number of events:              10000
    total time taken by event execution: 822.9329
    per-request statistics:
         min:                                 38.68ms
         avg:                                 82.29ms
         max:                                184.02ms
         approx.  95 percentile:              94.65ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   822.9329/0.00
```

# 组成集群

## 创建`docker-compose.yml`文件

我们将使用 Docker Compose 来创建集群。最初，我们将把它放在三个节点上，以便于管理。一旦我们有了概念验证群集，我们就可以向外扩展。处理这个问题最直接的方法是为每个容器将单独的端口映射到本地主机。我们可以指定在`docker-compose.yml`文件中使用的端口范围，如下所示。

```
version: '3'

services:
  node:
    image: pidoc
    ports:
      - "2201-2203:2222"
```

## 调出集群

使用`docker-compose`调出三个节点，使用`--scale`选项。

```
docker-compose up --scale node=3
```

## 可变配置

既然我们已经为集群准备好了所有的基础设施，我们需要管理它。我们可以使用 Docker 双重连接到 QEMU 监视器，但是`ssh`更加健壮。既然我们用的是`ssh`，就可以用 Ansible。这里提供了几个基本操作:`update`、`upgrade`、`reboot`、`shutdown`。这些可以根据需要扩展，以开发一个更健壮的系统。

## `hosts`文件

请注意我们之前在`docker-compose.yml`文件中指定的端口，并相应地编辑您的`hosts`清单。

```
[all:vars]
ansible_user=pi
ansible_ssh_pass=raspberry
ansible_ssh_extra_args='-o StrictHostKeyChecking=no'

[pidoc-cluster]
node_1.localhost:2201
node_2.localhost:2202
node_3.localhost:2203
```

想要更全面的了解 Ansible，请阅读[如何在 Ubuntu](https://appfleet.com/blog/how-to-install-and-configure-ansible-on-ubuntu-part-1/) 上安装和配置 Ansible。

## `update.yml`文件

```
---
- name: Apt update Pi...
  hosts: pidoc-cluster
  tasks:
    - name: Update apt cache...
      become: yes
      apt:
        update_cache=yes
```

用法:`ansible-playbook playbooks/update.yml -i hosts`

## `upgrade.yml`文件

```
---
- name: Upgrade Pi...
  hosts: pidoc-cluster
  gather_facts: no
  tasks:
    - name: Update and upgrade apt packages...
      become: true
      apt:
        upgrade: yes
        update_cache: yes
        cache_valid_time: 86400
```

用法:`ansible-playbook playbooks/upgrade.yml -i hosts`

## `reboot.yml`文件

```
---
- name: Reboot Pi...
  hosts: pidoc-cluster
  gather_facts: no
  tasks:
    - name: Reboot Pi...
      shell: shutdown -r now
      async: 0
      poll: 0
      ignore_errors: true
      become: true

- name: Wait for reboot...
      local_action: wait_for host=state=started delay=10
      become: false
```

用法:`ansible-playbook playbooks/reboot.yml -i hosts`

## `shutdown.yml`文件

```
---
- name: Shutdown Pi...
  hosts: pidoc-cluster
  gather_facts: no
  tasks:
    - name: 'Shutdown Pi'
      shell: shutdown -h now
      async: 0
      poll: 0
      ignore_errors: true
      become: true

- name: "Wait for shutdown..."
      local_action: wait_for host= state=stopped
      become: false
```

用法:`ansible-playbook playbooks/shutdown.yml -i hosts`

# 按比例放大

Docker Compose 使得在同一台主机上扩展 Raspberry Pi 容器变得微不足道。通过使用 Ansible 进行集群管理，通过将端口绑定从 localhost 更改为可路由的 IP 地址，还可以非常容易地横向扩展到其他主机。这是我们的例子，有 6 个节点，而不是 3 个。

## `docker-compose.yml`文件

```
version: '3'

services:
  node:
    image: pidoc
    ports:
      - "2201-2212:2222"
    deploy:
      resources:
        limits:
          cpus: "0.5"
```

# 调出集群

我们应该停止使用以前集群中的容器，并在扩展修改后的集群之前删除所有卷。要使用`docker-compose`调出所有 6 个节点，再次使用`--scale`选项。

```
docker-compose up --scale node=5
```

# 未来的工作

QEMU 的 Raspberry Pi 仿真仍在开发中。虽然这个项目的配置相对稳定，但还有很大的改进空间。尝试迁移到 Raspberry Pi 3 仿真将是一个雄心勃勃的下一步。Docker Compose 虽然是为单主机构建设计的，但已经足够容易手动或通过 Ansible 复制到其他主机。但是，它可以很容易地用 [Swarm](https://appfleet.com/blog/docker-swarm-and-shared-storage-volumes/) 或 k8s 扩展，使我们能够构建任何规模的模拟 Raspberry Pi 集群。此外，通过一个或多个端口重定向，可以实施其他控制系统，包括各种节点端点，这取决于目的和应用。

[![](img/c28bc9538d899d922b6e6c14ff24a48a.png)](https://www.buymeacoffee.com/markrhavens)

如果你喜欢这篇文章，像你这样的粉丝再给我一杯咖啡，肯定会鼓励我再写一篇这样的文章。顺便来看看，打个招呼，让我知道你对你可能想读的话题的想法。你可能是我需要写一些更棒的东西的灵感火花！

# 更新:2022 年 11 月 24 日

*在 2020 年疫情爆发之前，我提出并被委托写下上述的穿越。由* [*Appfleet*](https://appfleet.com/) *发布，并于 2020 年 1 月 21 日作为 Appfleet 博客* *发布* [*。*](https://appfleet.com/blog/raspberry-pi-cluster-emulation-with-docker-compose/) [*这个故事的一个版本*](https://havdevops.com/Raspberry-Pi-Cluster-Emulation-with-Docker-Compose/) *也可以在*[*my havdoveps 博客*](https://havdevops.com/) *上找到。我将上面更新的故事发布到 Medium，只做了微小的编辑和格式更改。如果有人发现错误需要我的注意，请不吝赐教。*

作为 Medium 的作者， [*我邀请你成为会员*](https://mark-havens.medium.com/membership) *并获得我的故事集，以及成千上万其他天才作家写的故事。你的会员费直接支持我和你阅读的其他作家，并让你完全接触媒体上的每一个故事。*

[*马克·兰道尔·哈文斯*](https://markhavens.us/) *是一位连续创业者和创造者，他从 19 岁起就开始创业。他是北德克萨斯州最知名的两个创客社区* [*达拉斯创客社区*](https://dallasmakercommunity.org/) *和* [*达拉斯创客空间*](https://dallasmakerspace.org/) *的创始人。他拥有科罗拉多技术大学的管理学硕士学位，并获得了德克萨斯大学 Rio Grand Valley 分校的计算机科学学士学位。为了表彰他在创客社区的工作，他获得了位于阿灵顿的德克萨斯大学的博士奖学金。当马克不从事制造工作或探索生活、文化和技术时，他住在达拉斯的中心郊区——沃斯堡大都会区，探索新的、创新的方式来养活他的母亲和两个孩子。*