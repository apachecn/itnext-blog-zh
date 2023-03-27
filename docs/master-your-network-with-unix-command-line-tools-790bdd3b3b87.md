# 使用 Unix 命令行工具掌控您的网络

> 原文：<https://itnext.io/master-your-network-with-unix-command-line-tools-790bdd3b3b87?source=collection_archive---------3----------------------->

## 基于 UNIX/OSX 命令行网络工具，用于分析、配置和控制您的网络。

![](img/d1c8b24c75fa678409ea02a352813e84.png)

# 基于 Unix 的系统中的网络

**Linux** 和其他**基于 Unix 的**操作系统是许多**网络**开发&系统管理被执行的地方。熟悉你的网络学习&使用一些最流行的**网络工具用于基于 **Unix 的**系统。**

使用他们提供的信息来**改进**、**诊断**或**了解**您的系统网络。

> 无论你是使用你的**家庭网络**还是使用**系统**来配置和设置**网络**，你都应该在这些工具中找到价值。

其中一些是内置的工具，只需一个命令就可以运行。其他的需要**下载**和**安装，**在 Linux 上经常和`apt-get`在一起，`brew`在 MacOS (OSX)上，通过`git`或者`pip`。

# 转速试验🚀

[Speedtest-cli](https://github.com/sivel/speedtest-cli) 是使用 speedtest.net 测试互联网带宽的命令行界面。它将在**位**或**字节**中确定**下载** & **上传**的速度。

> **慢网飞？**

`speedtest-cli`可能持有关键(**虽然不是解决方案，**那可能是你已经打开的 28 个 chrome 标签)。

## 使用 Pip (Python 包管理器)轻松安装

```
**$** pip install speedtest-cli
```

## 通过 Git 下载+安装

```
**$** git clone https://github.com/sivel/speedtest-cli.git
**$** cd speedtest-cli
**$** python setup.py install
```

## 通过自制软件下载+安装

```
**$** brew install speedtest-cli
```

## 以字节为单位获取速度

```
**$** speedtest-cli --bytes
```

# 带 ip 的网络接口(类似于过时的 ifconfig)

[ip](https://linux.die.net/man/8/ip) 允许配置网络接口，显示 Unix/Linux/OSX 系统上运行的所有当前接口的信息。

**可以:**

*   设置一个 ip 地址，**网络掩码**或**向网络接口广播**地址
*   为网络接口创建一个**别名**
*   设置**硬件**地址
*   启用或禁用网络**接口**。

## 查看与所有接口相关的信息

```
**$** sudo ip addr show
```

## 隔离特定网络接口以显示信息

```
**$** sudo ip eth0
```

## 显示路由表

```
**$** sudo ip route show
```

## 删除 IP 地址

```
**$** sudo ip addr del 192.168.50.5/24 dev eth0
```

## 打开接口(如果接口关闭)

```
**$** ifup eth0
```

## 关闭接口(如果接口打开)

```
**$** ifdown eth0
```

# 追踪路线

显示路由(路径)并测量数据包的传输延迟。它提供了连接到远程系统的路径列表(它连接到哪些路由器)。本例中的远程系统是`google.com`。

```
**$** traceroute google.com
```

# Iftop

[Iftop](http://www.ex-parrot.com/pdw/iftop/) 是一款流行的 linux 工具，可以让你**监控**带宽使用情况。它是对系统网络进程的**实时**分析。

> 如果您想知道哪些进程占用了网络并想诊断任何问题，这是很有用的。

## 家酿啤酒(MacOS)也有售

```
**$** brew install iftop
```

## 安装依赖项

```
[Debian/Ubuntu]**$** sudo apt install libpcap0.8 libpcap0.8-dev libncurses5 libncurses5-dev[CentOS/RHEL]**$** yum  -y install libpcap libpcap-devel ncurses ncurses-devel
```

## 下载并安装 iftop

```
[Debian/Ubuntu]**$** sudo apt install iftop[CentOS/RHEL]
Note: you need to enable the [EPEL repository](https://www.tecmint.com/how-to-enable-epel-repository-for-rhel-centos-6-5/) first**$** yum install epel-release
**$** yum install  iftop
```

## 使用

```
**$** sudo iftop
```

## 使用 ifconfig 监控接口

首先，您想找到要监视的接口标识符…

```
**$** sudo ifconfig
```

然后用`-i`将该接口传递给 iftop 命令

```
**$** sudo iftop -i en0
```

## 打开端口显示

```
**$** sudo iftop -P eth0
```

# 悬浮物

用于**调查插座**的实用程序。它显示带有**路由表**、**接口**统计、**伪装**连接和**组播**成员的网络连接信息。它给你所有的 **tcp** 、 **udp** 和 **unix** 套接字连接的有组织的信息。

## 显示所有插座

```
**$** ss -a
```

## 显示所有 TCP 套接字

```
**$** ss **-t -a**
```

## 显示所有 UDP 套接字

```
**$** ss **-u -a**
```

显示所有已建立的 SSH 连接

```
**$** ss **-o state established '( dport = :ssh or sport = :ssh )'**
```

> ss 的强大之处在于能够使用 TCP 状态进行过滤，TCP 状态是 TCP 连接在其生命周期中的各个阶段。

```
**$** ss -4 state <FILTER>
```

因此，您可以用过滤器的实际名称来替换`<FILTER>`,比如`ss -4 state listening`,以查看机器上所有侦听的 ipv4 套接字。

# Vnstat

是一个**网络流量监视器**，它记录特定接口的网络流量。Vnstat 本身实际上并不“T28”嗅探流量，它使用**内核**提供的关于网络的信息。因此，它被认为是一个低 CPU 使用率的**轻量级进程/应用程序**。

```
**$** vnstat --help
```

## 显示 5 分钟的网络流量

```
**$** vnstat -5
```

## 选择一个接口

```
**$** vnstat -i eth0 -5
```

## 启动

```
**$** vnstat -l
```

# Nettop

[Nettop](http://www.manpagez.com/man/1/nettop/) 随 MacOS (OSX)发货，通过一列**套接字或路由**显示更新的网络**结构**。它向与系统进程/应用程序的**套接字**连接提供信息。您还可以查看正在连接的**远程地址**。

## 它有 **3** 种模式

*   传输控制协议（Transmission Control Protocol）
*   用户数据报协议(User Datagram Protocol)
*   途径

使用不太密集的`-c`选项，更新不太频繁。您也可以在程序运行时用`p`切换“人类可读数字”。

```
**$** nettop **-m tcp**
```

> 模式由`-m`标志指定

# Nmap

Nmap 功能强大。它是网络安全专业人员在**道德黑客** & **渗透测试**和网络管理中使用的网络**发现**和安全**审计**工具。

你可以用**防火墙**，包过滤**监控**主机和正常运行时间，**端口**扫描，**检测**操作系统&服务版本，监控主机。它可以快速扫描大型网络或单个主机。

> Nmap 可以在所有主要的计算机操作系统上运行，官方的二进制软件包可以用于 Linux、Windows 和 Mac OS X。

```
**$** nmap
```

如果你喜欢图形用户界面，那么你可以下载 **Zenmap，**它是 **nmap** ，但是有一个(不太漂亮的)图形用户界面。这对你学习**技术**和**命令**很有用，你最终可以记住并使用命令行。

## 发现子网中的 IP

```
**$** nmap -sP 192.168.0.0/24
```

## 通过 Syn 和 UDP 端口识别 TCP(Root 权限)

`-Pn`无 ping(在防火墙阻止 icmp 回复时有用)

`-sS` TCP 同步扫描(隐形)

`-sU` UDP 扫描

```
***$*** *nmap -sS -sU -Pn 192.168.0.164*
```

> 这将检查前 2000 个 TCP 和 UDP 端口。

## T4 —更具侵略性的扫描

这是使用 T4 预设，这是更明显，但也快得多。

```
**$** nmap -T4 -A 192.168.0.0/24
```

# Tcpdump 数据包嗅探器

[TCP 包分析器](https://www.tcpdump.org/manpages/tcpdump.1.html)，它根据布尔表达式为**接口**打印出**包**的内容。

它打印出:

*   **捕获了**数据包
*   由**过滤器**接收的数据包**(有时不可预测，取决于操作系统)**
*   数据包**被丢弃**，通常是由于 *tcpdump* 正在运行的操作系统中的数据包捕获机制**缺少缓冲区大小**。

> 需要 root 权限

```
**$** sudo tcpdump
```

## 打印既不是来自也不是发往本地主机的流量

```
**$ tcpdump ip and not net** *localnet*
```

## 打印不是通过以太网广播或组播发送的 IP 广播或组播数据包

```
**$** tcpdump 'ether[0] & 1 = 0 and ip[16] >= 224'
```

# 网络结构

使用`networksetup`、用于**系统偏好**的**配置**工具来配置您的网络

> 需要管理员权限。

## 显示位置列表

```
**$** sudo networksetup -listlocations
```

## 获取当前位置

```
**$** sudo networksetup -getcurrentlocation
```

## 开关位置

```
**$** sudo networksetup -switchlocation Work
```

## 设置手动静态 IP

```
**$** sudo networksetup -setmanual Wi-Fi 10.0.0.2 255.255.255.0 10.0.0.1
```

# 机场连接信息(MacOS)

```
**$** /System/Library/PrivateFrameworks/Apple80211.framework/Versions/A/Resources/airport -I
```

# 安全审计

## 琳妮丝

被描述为基于 unix 系统的[经实战检验的**安全**工具](https://github.com/CISOfy/lynis)，它将执行**广泛的**系统安全**健康**扫描。如果您的目标是系统**硬化**和**符合性**测试，那么这是理想的选择。

**典型用例**

*   安全审计
*   合规性测试(例如 PCI、HIPAA、SOx)
*   渗透测试
*   漏洞检测
*   系统硬化

> Lynis 是模块化和机会主义的，这意味着它将运行特定于主机系统和可用软件的测试。Lynis 将根据其发现进行具体测试。

```
**$** git clone [https://github.com/CISOfy/lynis](https://github.com/CISOfy/lynis)**$** cd lynis; ./lynis audit system
```

> 当然，还有许多其他流行/有效的网络工具！但这是我经历过的一些例子。

## 如果你知道任何对你的工作重要的高质量的网络工具，请告诉我！