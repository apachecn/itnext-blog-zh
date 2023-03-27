# ssh: RSA 密钥和 SSH 代理，用于 SSH 密钥及其密码管理

> 原文：<https://itnext.io/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management-61c68001f54b?source=collection_archive---------3----------------------->

![](img/b36c576a88ddd8dfeff43b74a96dcf73.png)

在对 Nextcloud 客户端进行`keyring`配置的过程中(参见[Linux:next cloud 客户端、qtkeychain 和“org.freedesktop.secrets”名称未被任何人提供。服务文件”错误【T4 邮报】——我决定清理我的 SSH 密钥中的混乱，因为我有很多这样的密钥，有时认证变得很痛苦。](https://rtfm.co.ua/en/linux-the-nextcloud-client-qtkeychain-and-the-the-name-org-freedesktop-secrets-was-not-provided-by-any-service-files-error/)

一般来说，为了使这个更简单，可以使用系统级存储，比如`gnome-keyring`或 KeeyPassXC，但是我们将在下一篇文章中讨论它们。

今天，让我们讨论一下`ssh-agent`，以及如何使用它来管理密码保护的 RSA 密钥，以便在没有这样的后端的情况下进行 SSH 验证。

下面的例子是在一个 Arch Linux 系统上执行的，还做了一些额外的测试。

*   [ssh-agent](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#ssh-agent)
*   [运行代理](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#Running_the_agent)
*   [例题](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#Examples)
*   [密钥生成](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#A_key_generation)
*   [检查 SSH-key 的密码](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#Checking_SSH-keys_password)
*   [ssh-copy-id —将密钥复制到远程主机](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#ssh-copy-id_copy_a_key_to_a_remote_host)
*   [ssh-add](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#ssh-add)
*   [无法打开到您的身份验证代理的连接](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#Could_not_open_a_connection_to_your_authentication_agent)
*   [添加一个键](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#Adding_a_key)
*   [检查按键](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#Checking_keys)
*   [删除键](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#Deleting_keys)
*   [自动向 ssh-agent 添加密钥](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#Automatically_adding_keys_to_ssh-agent)
*   [使用多种类型的终端运行 ssh-agent】](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#Running_ssh-agent_with_multitype_terminals)
*   [~/。bashrc](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#bashrc)
*   [系统 d](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#systemd)
*   [~/。xinitrc](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#xinitrc)

# `ssh-agent`

[ssh-agent](https://rtfm.co.ua/goto/https://www.ssh.com/ssh/agent) 旨在管理用户的 [SSH 密钥](https://rtfm.co.ua/goto/https://www.ssh.com/ssh/identity-key)及其密码，以避免每次您需要使用密钥登录远程主机进行身份验证时都必须输入密钥密码。

## 运行代理

只需执行:

```
$ ssh-agent
SSH_AUTH_SOCK=/tmp/ssh-dMDE5mED77tM/agent.436347; export SSH_AUTH_SOCK;
SSH_AGENT_PID=436348; export SSH_AGENT_PID;
echo Agent pid 436348;
```

对于客户机，比如 ssh-client 或 git，它们需要知道以下变量:

*   `SSH_AGENT_PID` : a 启动了`ssh-agent` PID，例如用`ssh-agent -k`将其杀死
*   `SSH_AUTH_SOCK`:UNIX 套接字文件的路径，该文件将用于从客户端(`ssh`、`git`等)与`ssh-agent`通信

要运行代理而不显示这些变量并应用它们，请执行以下操作:

```
$ eval $(ssh-agent) > /dev/null
```

运行代理的方式有很多种，我们将仔细看看 [*运行 ssh-agent with multi type terminals*](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/#Running_ssh-agent_with_multitype_terminals)部分。

## 例子

下面我们来概述一下`ssh-agent`的基本用法。

**密钥生成**

创建新密钥:

```
$ ssh-keygen -t rsa -b 2048 -f /home/setevoy/.ssh/test-key -C “Testing key” -P pass
Generating public/private rsa key pair.
Your identification has been saved in /home/setevoy/.ssh/test-key.
Your public key has been saved in /home/setevoy/.ssh/test-key.pub.
...
```

此处的选项:

*   `-b`:密钥的长度，以比特为单位(RSA 默认为 3072)
*   `-f`:密钥文件的路径(默认为`~/.ssh/id_rsa`)
*   `-C`:密钥的注释(默认为*用户名@主机名*)
*   `-P`:钥匙的密码

**检查 SSH-key 的密码**

要检查钥匙的密码，您可以使用`ssh-keygen`和`-y`来显示有关该钥匙的信息，这将要求您输入该钥匙的密码:

```
$ ssh-keygen -y -f /home/setevoy/.ssh/test-key
Enter passphrase:
ssh-rsa AAAAB***gud2vedL/V Testing key
```

`**ssh-copy-id**` **-将密钥复制到远程主机**

您可以通过从 *test-key.pub* 文件中获取密钥的公共部分来手动复制密钥:

```
$ cat .ssh/test-key.pub
ssh-rsa AAAAB***gud2vedL/V Testing key
```

并将其添加到目标主机上的~/ `.ssh/authorized_keys`。

另一种方法，也是推荐的方法，是使用一个`ssh-copy-id`实用程序，它可以做同样的事情，但也可以监视文件夹/文件权限——这是基于 SSH RSA 的认证过程中最常见的问题:

```
$ ssh-copy-id -i /home/setevoy/.ssh/test-key setevoy@rtfm.co.ua
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: “/home/setevoy/.ssh/test-key.pub”
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed — if you are prompted now it is to install the new keyssetevoy@rtfm.co.ua’s password:Number of key(s) added: 1
Now try logging into the machine, with: “ssh ‘setevoy@rtfm.co.ua’”
and check to make sure that only the key(s) you wanted were added.
```

现在，您可以使用以下密钥登录:

```
$ ssh setevoy@rtfm.co.ua -i .ssh/test-key
Enter passphrase for key ‘.ssh/test-key’:
Linux rtfm-do-production 4.9.0–8-amd64 #1 SMP Debian 4.9.144–3.1 (2019–02–19) x86_64
…
setevoy@rtfm-do-production:~$
```

# `ssh-add`

好了，现在我们有一个密码保护的 RSA 密钥用于 SSH 验证。

但是在每次 SSH 登录期间，你必须一次又一次地输入密码，当使用大量的连接和密钥时，这将是一个真正的痛苦。

为了避免这个问题——使用`ssh-add`给`ssh-agent`添加一个键。

检查它是否正在运行:

```
$ ps aux | grep ssh-agent
setevoy 1322 0.0 0.0 5796 456 ? Ss Nov30 0:00 ssh-agent -s
setevoy 1324 0.0 0.0 5796 2160 ? Ss Nov30 0:00 ssh-agent -s
…
```

## 无法打开到您的身份验证代理的连接

最常见的问题是当`ssh-add`无法连接到代理时:

```
$ ssh-add
Could not open a connection to your authentication agent.
```

首先—检查其 PID 是否是从`SSH_AGENT_PID`设置的，或者通过检查`$SSH_AUTH_SOCK`变量，因为所有通信都是通过该变量指定的套接字文件进行的:

```
$ test -z $SSH_AGENT_PID; echo $?
0
```

这里是空的，因为`ssh-agent`是在另一个终端实例中启动的(我们很快会谈到如何处理它)。

现在—终止所有已经运行的实例:

```
$ killall ssh-agent
```

并重新运行代理实例:

```
$ eval $(ssh-agent -s)
Agent pid 452333
```

我们使用`-s`选项，因为并不是每个人都会在`bash` shell 和`eval`中执行上述步骤，以应用代理输出的字符串(`export SSH_AUTH_SOCK`)。

再次检查:

```
$ test -z $SSH_AGENT_PID; echo $?
1
```

和`ssh-add`:

```
$ ssh-add -l
The agent has no identities.
```

这里都搞定了。

## 添加密钥

运行:

```
$ ssh-add /home/setevoy/.ssh/test-key
Enter passphrase for /home/setevoy/.ssh/test-key:
Identity added: /home/setevoy/.ssh/test-key (Testing key)
```

## 检查钥匙

使用`-l`选项检查哪些键已经加载到代理实例中:

```
$ ssh-add -l
2048 SHA256:pTyrGtk1hnNHB6b8ilp5jRe1+K4KrLHg50yUGilApLY Testing key (RSA)
```

## 正在删除密钥

使用`-d`删除一个键:

```
$ ssh-add -d .ssh/test-key
Identity removed: .ssh/test-key (Testing key)
```

和`-D`一次删除所有键:

```
$ ssh-add -D
All identities removed.
```

## 自动将钥匙添加到`ssh-agent`

为了使`ssh`(例如`git`)在不需要手动运行`ssh-add`的情况下将用过的钥匙添加到`ssh-agent`中，您可以将`[AddKeysToAgent](https://rtfm.co.ua/goto/https://man.openbsd.org/ssh_config#AddKeysToAgent)` 参数添加到в `~/.ssh/config`中，并指定以下选项之一- *是*、*确认*或*询问*(参见`[SSH_ASKPASS](https://rtfm.co.ua/goto/https://man.openbsd.org/ssh-add.1#DISPLAY_and_SSH_ASKPASS)`):

```
$ head -1 .ssh/config
AddKeysToAgent yes
```

让我们检查一下——目前没有添加任何内容:

```
$ ssh-add -l
The agent has no identities.
```

建立连接，输入密钥密码:

```
$ ssh -i .ssh/test-key setevoy@rtfm.co.ua
Enter passphrase for key ‘.ssh/test-key’:
…
setevoy@rtfm-do-production:~$
```

断开连接，并立即检查代理中的密钥:

```
setevoy@rtfm-do-production:~$ logout
Connection to rtfm.co.ua closed.
$ ssh-add -l
2048 SHA256:pTyrGtk1hnNHB6b8ilp5jRe1+K4KrLHg50yUGilApLY Testing key (RSA)
```

在下一次连接时—`ssh`客户端将使用来自代理的密钥，并且不会再次要求您输入密钥的密码:

```
$ ssh -i .ssh/test-key setevoy@rtfm.co.ua
…
setevoy@rtfm-do-production:~$
```

# 使用多种端子运行`ssh-agent`

另一个大问题是当 bash 会话很少时该怎么办，例如在各种终端的选项卡中，因为它没有设置`$SSH_AUTH_SOCK`变量，ssh 客户端将无法与已经运行的`ssh-agent`实例通信。

也就是说，当您在新终端中运行`ssh-add`时，您将会看到已经提到的“*无法打开到您的身份验证代理*的连接”错误:

```
$ ssh-add -l
Could not open a connection to your authentication agent.
```

有几种方法可以在新的 bash 会话初始化期间初始化变量，例如，您可以将以下内容添加到您的`~/.bashrc`中:

```
if [ -z "$SSH_AUTH_SOCK" ] ; then 
  eval `ssh-agent -s` 
  ssh-add /home/setevoy/.ssh/test-key 
fi
```

但是在这种情况下，每个 bash-sessions 都有自己的 ssh-agent 在运行，这不是问题，但是可能不是您想要的。

另一种方法是将以下代码添加到`~/.bashrc`:

```
ssh-add -l &>/dev/null 
if [ "$?" == 2 ]; then
  test -r ~/.ssh-agent-env && eval "$(<~/.ssh-agent-env)" >/dev/null   ssh-add -l &>/dev/null
  if [ "$?" == 2 ]; then
    (umask 066; ssh-agent > ~/.ssh-agent-env)
    eval "$(<~/.ssh-agent-env)" >/dev/null
    ssh-add /home/setevoy/.ssh/test-key 
  fi
fi
```

此处(参见`ssh-agent` [文档](https://linux.die.net/man/1/ssh-add)中的响应代码):

1.  尝试执行`ssh-add -l`，并将输出重定向到`/dev/null`
2.  检查上一个命令返回的代码:
3.  检查`~/.ssh-agent-env`是否存在并可供读取，读取并将其输出传递给`bash`
4.  重试`ssh-add -l`
5.  如果代码仍然为 2:
6.  创建具有 660 权限的`~/.ssh-agent-env`文件(仅针对所有者读写)
7.  启动`ssh-agent`并将其输出重定向到`.ssh-agent-env`文件中
8.  读取`.ssh-agent-env`内容，并通过管道将其传递给`bash`
9.  运行`ssh-add /home/setevoy/.ssh/test-key`

这是一个不错的解决方案，通过这种方式，我们所有的会议将使用相同的代理，尽管一些指南建议为个人和工作使用不同的代理。

## `systemd`

另一个解决方案是通过添加一个单元文件并作为`systemd`服务运行`ssh-agent`来创建一个专用的`systemd`服务，详情请参见。

如果尚未添加，则创建一个目录:

```
$ mkdir -p .config/systemd/user/
```

并在那里创建一个`~/.config/systemd/user/ssh-agent.service`文件:

```
[Unit]
Description=SSH key agent [Service]
Type=simple
Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
ExecStart=/usr/bin/ssh-agent -D -a $SSH_AUTH_SOCK [Install] WantedBy=default.target
```

接下来，theWiki 讲述了变量的`~/.pam_environment`文件，但在我目前的情况下，我有 Openbox，通常通过`.config/openbox/autostart`文件设置变量:

```
$ head -2 .config/openbox/autostart
# ssh-agent.service
SSH_AUTH_SOCK=”${XDG_RUNTIME_DIR}/ssh-agent.socket”
```

*顺便回忆一下关于 BASH 中设置默认值的事情—*[*bash:переменные—передачазначенийпо-умолчанию$ { var:-default value }、заменазначений—$ { var:+alternate value }исообщений—$ { var:？*](https://rtfm.co.ua/bash-peremennye-peredacha-znachenij-po-umolchaniyu-var-defaultvalue-zamena-znachenij-varalternatevalue-i-soobshhenij-varmessage/)*【Rus】*

现在，停止所有代理运行:

```
$ killall ssh-agent
```

检查`[$XDG_RUNTIME_DIR](https://rtfm.co.ua/goto/https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)`变量值:

```
$ echo $XDG_RUNTIME_DIR
/run/user/1000
```

现在，手动设置`$SSH_AUTH_SOCK`变量:

```
$ SSH_AUTH_SOCK=”${XDG_RUNTIME_DIR}/ssh-agent.socket”
```

并通过`systemctl --user`运行代理:

```
$ systemctl — user start ssh-agent
```

检查一下:

```
$ systemctl — user status ssh-agent
● ssh-agent.service — SSH key agent
Loaded: loaded (/home/setevoy/.config/systemd/user/ssh-agent.service; disabled; vendor preset: enabled)
Active: active (running) since Sun 2019–12–01 09:15:18 EET; 2s ago
Main PID: 497687 (ssh-agent)
CGroup: /user.slice/user-1000.slice/user@1000.service/ssh-agent.service
└─497687 /usr/bin/ssh-agent -D -a /run/user/1000/ssh-agent.socket
Dec 01 09:15:18 setevoy-arch-pc systemd[670]: Started SSH key agent.
Dec 01 09:15:19 setevoy-arch-pc ssh-agent[497687]: SSH_AUTH_SOCK=/run/user/1000/ssh-agent.socket; export SSH_AUTH_SOCK;
Dec 01 09:15:19 setevoy-arch-pc ssh-agent[497687]: echo Agent pid 497687;
```

套接字的变量:

```
$ echo $SSH_AUTH_SOCK
/run/user/1000/ssh-agent.socket
```

并尝试`ssh-add`:

```
$ ssh-add -l
The agent has no identities.
```

“管用！”

您现在可以添加到 autostart:

```
$ systemctl — user enable ssh-agent
Created symlink /home/setevoy/.config/systemd/user/default.target.wants/ssh-agent.service → /home/setevoy/.config/systemd/user/ssh-agent.service.
```

## `~/.xinitrc`

你可以使用的另一种方法是将代理的开始添加到`~/.xinitrc`。

在这种情况下，当您将执行`startx`(例如，在我的情况下，当我没有任何登录管理器，并且通过在控制台中输入`startx`手动启动 X.Org)时，首先，将启动一个代理和一个 Openbox 会话，参见[文档](https://rtfm.co.ua/goto/https://wiki.archlinux.org/index.php/Xinit):

```
$ cat ~/.xinitrc
eval $(ssh-agent) &
exec openbox-session
```

此外，正如在本文开始时已经提到的，键的后端还有其他实现，可以与`ssh-agent`一起使用，或者代替【】——一种从 ssh 客户端到`ssh-agent`实例的“包装器”或“代理”请求，或者完全代替 ssh 代理本身，并且自己存储键和密码，但是我们将在后面的文章中讨论它们。).

*原载于 2019 年 12 月 1 日*[*https://rtfm.co.ua*](https://rtfm.co.ua/en/ssh-rsa-keys-and-ssh-agent-for-ssh-keys-and-their-passwords-management/)*。*