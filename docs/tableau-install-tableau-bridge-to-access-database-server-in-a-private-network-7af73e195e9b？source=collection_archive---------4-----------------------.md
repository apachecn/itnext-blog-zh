# Tableau:安装 Tableau Bridge 以访问私有网络中的数据库服务器

> 原文：<https://itnext.io/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network-7af73e195e9b?source=collection_archive---------4----------------------->

![](img/a24c2c3fc6e0bcbefdc0785eff5382f6.png)

要访问没有公共访问权限的数据库服务器(这是必须的——只能在 AWS VPC 内部访问), Tableau 建议使用其名为 Tableau Bridge 的工具。

这个想法是在网络中运行一个桥接服务，它可以通过自己的私有 IP 访问数据库服务器。此外，网桥将在通过互联网进行传输的过程中执行数据加密。

总的来说，Tableau 做了很多关于它的桥的文档，但是有点不完整，对我来说。

例如，我的第一个问题是——桥服务需要在运行 Tableau Desktop 的同一台机器上运行吗？或者可以托管在另一台 PC 上？文档中没有提到这一点，我不得不询问技术人员。支持，第一次告诉我，他们可以在单独的情况下冲洗。但是后来发现，您仍然需要从带有桌面的 PC 上访问来添加新的数据源。

此外，网桥通常如何在网络层工作也很奇怪。连接是如何路由的？为什么不可能在办公室中有一台带有网桥和桌面的 PC，在数据库服务器网络中的一台 PC 上有另一台只有网桥的 PC，然后在它们之间路由请求？

不幸的是，Tableau 的文档没有透露这些细节。查看这里:[使用网桥保持数据新鲜](https://help.tableau.com/current/online/en-us/qs_refresh_local_data.htm)，[规划你的网桥部署](https://help.tableau.com/current/online/en-us/to_bridge_scale.htm)。

因此，在本文中，我们将使用 Wirfows 构建一个 EC2，将在那里安装 Tableau Bridge 客户端，Tableau Desktop，并将通过专用网络链接添加一个到新数据库服务器的新连接，以便稍后在 Tableau Online 中使用。

# **内容**

*   [AWS Windows EC2](https://rtfm.co.ua/en/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network/#AWS_Windows_EC2)
*   [来自 Linux 的 Windows RDP](https://rtfm.co.ua/en/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network/#Windows_RDP_from_Linux)
*   [Windows:“您当前的安全设置不允许下载此文件”](https://rtfm.co.ua/en/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network/#Windows_Your_current_security_settings_do_not_allow_this_file_to_be_downloaded)
*   [安装台面桥](https://rtfm.co.ua/en/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network/#Install_Tableau_Bridge)
*   [Tableau 桥模式](https://rtfm.co.ua/en/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network/#Tableau_Bridge_Mode)
*   【Windows 版 MySQL 驱动程序
*   [AWS Aurora RDS MySQL 测试实例](https://rtfm.co.ua/en/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network/#AWS_Aurora_RDS_MySQL_test_instance)
*   [Windows ConEmu 和 MySQL Shell](https://rtfm.co.ua/en/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network/#Windows_ConEmu_and_MySQL_Shell)
*   [Tableau 桌面和一个数据源连接](https://rtfm.co.ua/en/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network/#Tableau_Desktop_and_a_data_source_connection)
*   [“现场带电连接被禁用”错误](https://rtfm.co.ua/en/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network/#The_Live_connections_disabled_on_site_error)
*   [添加新的 Tableau 数据源](https://rtfm.co.ua/en/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network/#Adding_a_new_Tableau_Data_Source)

# AWS Windows EC2

选择 Windows Server 2019:

![](img/11f98aca4c9dc90c4adeeee5905651d7.png)

目前，我们可以使用 *t3.large* ，因为我们有 Windows，加上 Tableau Bridge 是用 Java 编写的，所以它会消耗一些资源:

![](img/a66983383800950ea333f38fde6490b7.png)

选择将托管我们的数据库服务器的 VPC:

![](img/8ba56f98ea7700d1c2cfd9a25724a0a5.png)

为了以后的生产，我们将会有一个专用的 VPC 和 VPC 喷丸。

为实例设置名称:

![](img/b22ca6f98dcbbb971f8902913f7ebc2d.png)

在其安全组办公室允许 RDP:

![](img/94fb1b89fd87a5a18ac3ac135408ae4f.png)

网桥将只创建出站连接，因此无需在此添加任何入站规则。

创建 RSA 密钥:

![](img/d4e2a768cc11955b746b04feef3043e7.png)

## 来自 Linux 的 Windows RDP

使用`[xfreerdp](https://linux.die.net/man/1/xfreerdp)`。

要获取密码，进入 AWS EC2 控制台，点击*安全>获取 Windows 密码*:

![](img/618e0a3cf04b5832704d66bd87a43cc4.png)

上传您的密钥:

![](img/d7e0ddfee4e4430fa0e72c3ab7352019.png)

复制密码:

![](img/96f8ded86fe052089fcc7bcd6779abb2.png)

通过 RDP 连接:

```
$ xfreerdp /u:Administrator /p:’.6o***A*E’ /h:900 /w:1440 /v:18.***.***.139
```

安装 Chrome/Edge/无论什么，只是没有 IE 11:

![](img/b7ead6bf1c8a310df6a12ac953caa81f.png)

哦，我的……Windows 会建议下载它的 Edge，即使谷歌为“ *chrome 下载*”:

![](img/6253ffc5aaaf1fa64f532aefbc024ae4.png)

## Windows:“您当前的安全设置不允许下载此文件”

并将阻止下载尝试:

![](img/8cd1488c12c1f13edc27e8356a06b930.png)

转到 IE 设置，并启用下载:

![](img/de75d7cbaafa4df730870b395224c9ce.png)

# 安装 Tableau 桥

文档— [安装桥架](https://help.tableau.com/current/online/en-us/to_bridge_install.htm)。

不确定版本 Tableau Bridge 是否需要与 Tableau Desktop 相同的版本。

不要觉得这两者有关系，但是我用过相同的版本。

检查当前桌面的版本— *帮助>关于 Tableau* :

![](img/6e1fed8f69126f12d4764c48c7cfd181.png)

从[产品下载和发行说明(链接在新窗口中打开)](https://www.tableau.com/support/releases/bridge?_ga=2.142565170.1252987598.1627460143-931540619.1627460143)页面下载 Tableau Bridge:

![](img/54597dd4bf7e582ba77d72880880a27c.png)

运行安装程序:

![](img/32eb34f65bc7dd91bcc667613dedf3f2.png)

在线登录 Tableau:

![](img/af5d187e054496461b3c0fb589c3899c.png)

我们已经为新的联系做好了准备:

![](img/8cdcf661c985a5094aef0d90e89478ef.png)

## Tableau 桥接模式

在底部左侧，您可以选择桥将如何运行，参见[安装桥关于桥客户端](https://help.tableau.com/current/online/en-us/to_bridge_client.htm)。

简而言之:

*   **应用程序**:只有当用户登录到 Windows 时才会运行 Bridge
*   **服务**:每次系统启动后都会运行桥接

好了，我们已经开始了桥梁服务。到 Tableau Online — *设置>桥*查看:

![](img/01d68556d9ee98abc4aeeffdf176ade2.png)

它的状态是*连接*，这里看起来不错。

## 用于 Windows 的 MySQL 驱动程序

下载 [MySQL 驱动](https://dev.mysql.com/downloads/connector/odbc):

![](img/0afbd5cd852ed1bcfdbe0306acde0487.png)

不登录，点击*就下载*:

![](img/82904422b866bc23a772f17a0b107036.png)

安装它:

![](img/113328f39c558dcb2dfd872232120d38.png)

# AWS Aurora RDS MySQL 测试实例

现在，让我们为测试目的创建一个新的 AWS Aurora RDS 集群:

![](img/d1f80d511d095ea1822dad737a69fc96.png)

在其网络设置中，选择 AWS VPC，在那里我们的 EC2 与 Tableau 桥正在运行。设置*公共访问*禁用，附加一个安全组:

![](img/31eaa33e27e4d3eac55e5aae69abe895.png)

在此安全组中，允许从带 Windows 和网桥的ес2 访问端口 3306 上的 Aurora 集群:

![](img/60fdd4bac9f6540ea60eec80d85048d7.png)

## Windows ConEmu 和 MySQL Shell

可选地，安装 [ConEmu](https://conemu.github.io/) ，因为默认窗口`cmd`很糟糕。

还有，可以用 [MySQL Shell](https://dev.mysql.com/downloads/shell/) 。

运行 Shell:

![](img/ea744d58dedd2b9cbcbb640fd5bc6ce5.png)

使用`\help`命令或特定命令查看帮助，例如`connect` - `\? \connect`。

连接到 Aurora 服务器:

```
MySQL JS > \connect admin@tb-test.bm.local
```

切换到`sql`模式:

![](img/0d198e0156a359a684347a2d22ca7e3b.png)

创建数据库和表:

![](img/a0e08fe40522785ce7db223e007dda29.png)

# Tableau 桌面和数据源连接

接下来，需要在 EC2 上安装 Tableau 桌面，我们的桥就在那里运行。

然后，我们将从这个桌面创建一个新的数据源，将其发布到 Tableau Online，Tableau Online 将使用桥从这个数据库服务器运行提取和刷新。

下载 Tableau 桌面安装程序—[https://www.tableau.com/products/desktop/download](https://www.tableau.com/products/desktop/download)，安装:

![](img/f541a7bcc9a204f19f0ba3cf04f88b89.png)

据我所知，同一个许可密钥可以使用三次，所以我们使用一个现有的密钥。

选择*用产品密钥*激活:

![](img/5809d35cebbd3b2443f525b62b4388cf.png)![](img/396db00317d66d3afa7fa9793171d31c.png)

从旧的 Tableau 桌面获取当前密钥— *帮助>管理产品密钥*，为新的桌面实例输入它:

![](img/e52f6f056234c15c6e73c1da8f987689.png)

## “现场禁用实时连接”错误

此时，我们可以看到桥服务的“ ***现场活动连接禁用*** ”消息:

![](img/68bdd607d9778424c8bf98e7d9963037.png)

Google it，找到[确定实时查询问题的原因](https://help.tableau.com/current/online/en-us/to_bridge_troubleshoot.htm#identify-causes-for-live-query-issues)，转到 Tableau Online，并启用 [Tableau 桥池](https://help.tableau.com/current/online/en-gb/to_enable_bridge_live_connections.htm):

![](img/8496a19201fc398db3d86a83e0c17be2.png)

现在，错误消失了:

![](img/25138cf61200a5db9ca45b9bccf7b08f.png)

## 添加新的 Tableau 数据源

创建新工作簿:

![](img/1d2232de7871f446fbc216fcdf2d97dd.png)

点击*新建数据源*:

![](img/b4b70ce48d477633bd54f9a00da95f00.png)

选择 AWS Aurora:

![](img/b53127979b9702cf3761c7fac2e1e9dc.png)

设置连接详细信息，登录到数据库服务器:

![](img/6ae48bb3ee24f284c7abf1d6c780a733.png)

并且连接就绪:

![](img/c94ea8687777b7c231516674badf9324.png)

推到 Tableau 在线，см。[发布数据源](https://help.tableau.com/current/pro/desktop/en-us/publish_datasources.htm):

![](img/3415c7271d1c09402cbecc6c63544ea2.png)![](img/c94e965abbca24d12f9ea302c916e95f.png)

去 Tableau 在线，检查连接:

![](img/b6d3c669ad44c74322bdc0749afc385d.png)

其网络类型必须设置为*专用网络*:

![](img/b935fc880c40f613a244b4fcff146df8.png)

所以对这个数据源的所有请求都将通过 Tableau 桥路由。

向数据库添加一些数据:

![](img/7eb2a6ee61b6cf63ea4aa3f7048156fc.png)

让我们试着在 Tableau Online 中看到它们—创建一个新工作簿，设置数据源:

![](img/24230c317ed2092dbf9d558798a6b3de.png)

刷新数据，以从数据库中获取最新的更改:

![](img/ee2cf10e67cf9ff9bda9b6c3ffb89f01.png)

将`col1`添加到工作簿，现在您必须看到来自数据库的数据:

![](img/d840f3ce8acd44f6b36a3f2456df4a7b.png)

更新表中的数据:

![](img/58dad8fe9b7145f1b7cc8b1b21cf36d6.png)

刷新数据源:

![](img/3a1d7fc3b619abb53938e8b8ed9c6193.png)

当【T23 进行中】状态变为【T25 送桥】状态时，检查*工作*:

![](img/c186ceea002faee71688bce870d5760f.png)

再次刷新工作簿中的数据:

![](img/8e5e66985005457217d94dec6cfa47b3.png)

你会看到新的数据:

![](img/ed8cc287ac7ee1d5cf7b52a948ed5579.png)

完成了。

*最初发布于* [*RTFM: Linux、DevOps、系统管理*](https://rtfm.co.ua/en/tableau-install-tableau-bridge-to-access-database-server-in-a-private-network/) *。*