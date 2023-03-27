# 如何在 Ubuntu(AWS-EC2)上设置 SFTP 服务器

> 原文：<https://itnext.io/how-to-set-up-sftp-server-on-ubuntu-aws-ec2-970c4353f894?source=collection_archive---------5----------------------->

我最近请求为一个客户设置 SFTP，以便他们可以管理环境内外的文件传输。在做好一切准备的同时，似乎许多向导在某种形式上遗漏了关键的一步。所以我决定自己写一份指南，详细介绍我的整个过程。

**■关于 SFTP** 的介绍
SFTP 代表 SSH 文件传输协议。顾名思义，这是一种使用加密 SSH 连接在机器之间传输文件的安全方式。尽管名为 FTP(文件传输协议),但它是一种完全不同的协议，尽管现代 FTP 客户端广泛支持它。

![](img/729a84df0832083bc801b559f2fbd8e2.png)

在某些情况下，您可能希望只允许某些用户传输文件，而不允许 SSH 访问。在本教程中，我们将设置 SSH 守护进程来限制 SFTP 对一个目录的访问，每个用户都不允许 SSH 访问。

对于 FTP 设置，请通过此链接。【https://itnext.io/create-ftp-on-aws-ec2-f6e57d4d7f25】T5


所以，让我们开始 SFTP 设置。
**步骤 1:安装 OpenSSH-server & SSH**
如果您还没有这样做，在服务器中安装 OpenSSH，您可以使用以下命令:

```
$ sudo apt install openssh-server
```

您还需要在将要访问 SFTP 服务器的系统上安装 SSH。

```
$ sudo apt install ssh
```

**步骤 2:创建 SFTP 用户帐户**
首先，我们需要创建一个新用户，该用户只被授予对服务器的文件传输访问权限。

```
$ sudo adduser sftp_user
```

系统会提示您为该帐户创建一个密码，然后输入一些用户信息。用户信息是可选的，因此您可以按 ENTER 键将这些字段留空。

```
Enter new UNIX password: 
Retype new UNIX password: 
.....
passwd: password updated successfully
```

您现在已经创建了一个新用户，我们将被授予访问受限目录的权限。
下一步，我们将创建文件传输目录并设置必要的权限。

**步骤 3:为文件传输创建目录**
为了限制 SFTP 对一个目录的访问，首先，我们必须确保该目录符合 SSH 服务器的权限要求，这是非常特殊的。

具体来说，目录本身以及文件系统树中它上面的所有目录必须归 root 所有，其他任何人都不可写。因此，不可能简单地限制对用户主目录的访问，因为主目录归用户所有，而不是归根目录所有。

这里，我们将创建并使用 **/var/sftp/myfolder/data/** 作为目标上传目录。 **/var/sftp/myfolder** 将归 root 所有，其他用户不可写。
子目录 **/var/sftp/myfolder/data/** 将归 **sftp_user** (我们之前创建的)所有，以便用户能够向其中上传文件。

首先，创建目录。

```
$ sudo mkdir -p /var/sftp/myfolder/data/
```

将 **/var/sftp/myfolder** 的所有者设置为 **root** 。

```
$ sudo chown root:root /var/sftp/myfolder
```

授予 root 用户对同一目录的写权限，仅授予其他用户读和执行权限。

```
$ sudo chmod 755 /var/sftp/myfolder
```

将上传目录的所有权更改为 sftp_user。

```
$ sudo chown sftp_user:sftp_user /var/sftp/myfolder/data/
```

这里我们已经完成了目录限制。
因此，我们的 sftp_user 将只使用下面路径中的 **/data/** 。sftp_user 从不更改目录。
**/var/sftp/my folder/data/**

**步骤 4:sshd_config 设置**
在此步骤中，我们将修改 SSH 服务器配置，以禁止 sftp_user 的终端访问，但允许文件传输访问。

使用下面的命令打开 SSH 服务器配置文件。

```
$ sudo nano /etc/ssh/sshd_config
```

或者可以通过↓来做。

```
$ sudo vi /etc/ssh/sshd_config
```

滚动到文件的最底部，并追加以下配置片段:

**/etc/ssh/sshd_config**

```
Port <your_port_number>
Match User sftp_user
ForceCommand internal-sftp
PasswordAuthentication yes
ChrootDirectory /var/sftp/myfolder
PermitTunnel no
AllowAgentForwarding no
AllowTcpForwarding no
X11Forwarding no
```

然后保存并关闭文件。[按:wq + enter]

下面是这些指令的作用:
**●匹配用户**告诉 SSH 服务器只对指定的用户应用以下命令。这里，我们指定 sftp_user。
**●force command internal-sftp**强制 SSH 服务器在登录时运行 sftp 服务器，不允许 shell 访问。
**●password authentic ation**yes 允许对该用户进行密码认证。
**●ch root directory/var/sftp/myfolder**确保不允许用户访问/var/sftp/my folder 目录之外的任何内容。
**●AllowAgentForwarding no**、AllowTcpForwarding no、X11Forwarding no 对该用户禁用端口转发、隧道和 X11 转发。

在**匹配用户【用户名】**中，您也可以通过使用以下命令来使用该组。
**匹配组【sftp_group】**
注意:你需要创建一个名为，sftp _ Group 的新组。

**步骤 5:重启服务**
要应用配置更改，重启服务。

```
$ sudo systemctl restart sshd
```

或者

```
$ sudo /etc/init.d/ssh restart
```

现在，您已经将 SSH 服务器配置为只允许 sftp_user 访问文件传输。

**第六步:在 AWS-EC2 安全组**
中打开你的 sftp 端口如果你使用的是 AWS-EC2 实例，那么你需要在这里打开端口。
**登录您的 AWS 账户。**
↓
**转到服务然后点击 EC2 菜单- >运行实例。**
↓
**转到你的实例。**
↓
**打开安全组。**
↓
**在入库规则中，编辑入库规则**
↓
**请做如下设置**
**1。类型** =自定义 TCP
**2。协议** = TCP
**3。端口范围** = your_port(与 sshd_config 文件中设置的相同)
**4。如果你不想在任何地方设置，你需要在这里将 IP 列入白名单。
**5。描述—可选的** =你可以在这里提到一些有用的信息。**

如果你想看视频，请访问这个链接。
https://itnext.io/create-ftp-on-aws-ec2-f6e57d4d7f25

最后一步是测试配置，以确保它按预期工作。

**第七步:验证配置**
您可以在您的终端和第三方软件(如 WinSCP)中进行验证。

**结论**虽然本教程只使用了一个目录和一个用户，但是您也可以将这个示例扩展到多个用户和多个目录。SSH 服务器允许更复杂的配置方案，包括同时限制组或多个用户的访问，甚至限制对某些 IP 地址的访问。

我希望这篇文章能帮助你在 Ubuntu 上设置 SFTP 服务器。如果你遇到任何错误，请与我分享。

如果本指南对您和您的团队有所帮助，请与他人分享！

**感谢&最诚挚的问候
Alok Rawat**

*最初发表于 2020 年 5 月 4 日 https://qiita.com*[](https://qiita.com/alokrawat050/items/709d3c777407ab658aa9)**。**