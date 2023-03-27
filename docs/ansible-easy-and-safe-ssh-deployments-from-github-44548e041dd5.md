# Ansible:来自 GitHub 的简单安全的 SSH 部署

> 原文：<https://itnext.io/ansible-easy-and-safe-ssh-deployments-from-github-44548e041dd5?source=collection_archive---------4----------------------->

![](img/0a9c642ec9b5dee43516504b435d22da.png)

Ansible 是一个服务器编排工具，您也可以使用它以可预测和可重复的方式在远程机器上执行工作流。在之前的文章[“使用 Ansible 自动化 Laravel 部署”](https://roelofjanelsinga.com/articles/automating-laravel-deployment-using-ansible)中，我已经列出了如何使用 GitHub 用户名和用户令牌使用 Ansible Vault 部署应用程序。但是，您也可以使用 SSH 来实现这一点，确保您的服务器只对您的应用程序存储库进行拉取访问。这个额外的安全层很容易实现，所以在这篇文章中，我们将看看如何做到这一点。

在这篇博文中，我们将按照以下步骤使用与之前相同的配置，但是使用 SSH 而不是用户令牌或密码:

1.  在服务器上生成 SSH 密钥
2.  将公共 SSH 密钥作为部署密钥提交给 GitHub
3.  使用 SSH 部署您的应用程序

# 先决条件

你可以使用[以前的博客文章](https://roelofjanelsinga.com/articles/automating-laravel-deployment-using-ansible)中的配置来部署你的应用程序，这篇文章中唯一的不同是你不需要 Ansible Vault，所以你可以从那篇文章中提到的配置中删除“vars_files”键。除此之外，您还需要使用 SSH 地址作为“github_repo_url”值:[【电子邮件保护】](https://roelofjanelsinga.com/cdn-cgi/l/email-protection):your-username/your-repository . git

# 在服务器上生成 SSH 密钥

在服务器上生成 SSH 密钥是一个快速的过程，只需要一个命令:

```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
```

让我们来分解一下:

*   “-t”:这是我们定义公钥算法并将其设置为“RSA”的地方
*   "-b ":我们将密钥大小设置为 4096 位(不要再低了)
*   "-f ":我们正在指定我们想要使用的文件名。

指定文件名时，请确保该文件不存在。这将导致现有的密钥被覆盖，这可能会中断您可能拥有的其他 SSH 连接。如果文件已经存在，请选择不同的名称:~/。例如，ssh/your_repository_name。

如果您最终使用了不同于“id_rsa”的文件名，您需要进行额外的更改:

1.  创建和/或打开以下文件:~/。ssh/config
2.  添加下面的代码片段
3.  更新您的存储库的 SSH 地址以使用定制的 SSH 密钥

```
Host github_server 
  Hostname github.com 
  IdentityFile ~/.ssh/your_repository_name 
  IdentitiesOnly yes
```

并将 Ansible 中存储库的 SSH 地址更改为[【email protected】](https://roelofjanelsinga.com/cdn-cgi/l/email-protection)_ server:your-username/your-repository . git。注意，我们不再使用 github.com，而是使用我们的自定义配置:github_server。

# 将公共 SSH 密钥作为部署密钥提交给 GitHub

现在我们已经在服务器上生成了一个私有和公共的 SSH 密钥，我们可以将其作为“部署密钥”添加到我们的 GitHub 存储库中。[部署密钥](https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys)是允许另一台机器访问单个 GitHub 存储库的 SSH 密钥。您作为存储库所有者，甚至可以指定远程计算机是否具有推送权限。默认情况下，部署密钥只具有拉访问权限，这正是我们想要的部署。我们不想要推送特权，也不想让远程机器无限制地访问我们的整个 GitHub 帐户。

要从服务器获取公共 SSH 密钥，请运行以下命令:

```
cat ~/.ssh/id_rsa.pub
```

如果您使用自定义名称作为 SSH 密钥，请改用该名称。这可能是:

```
cat ~/.ssh/your_repository_name.pub
```

注意。SSH 密钥后面的酒吧，这是你的公钥。你可以把这个给别人，只是要确保永远不要把你的私钥给别人(没有。酒吧在尽头)。私人文件的内容必须始终保密。

您现在应该在终端中看到您的公钥，从“ssh-rsa”开始。复制整个密钥，包括 ssh-rsa 和末尾的机器名称。这都是你的公钥的一部分。

现在，转到您在 GitHub 上的存储库，导航到“设置”选项卡->部署密钥->添加部署密钥。

给你的部署密钥一个可识别的标题，像“生产服务器”，并在“密钥”字段中粘贴公共 SSH 密钥。不要选中“允许写访问”复选框，除非您确实需要。现在单击“添加密钥”，您应该会在概览中看到新创建的部署密钥。

# 使用 SSH 部署您的应用程序

现在，您已经将服务器连接到了 GitHub 存储库中，您可以对应用程序进行一些更改，并提交您的更改。当您准备好部署更改时，执行您的 Ansible 行动手册，并使用新的 SSH 设置查看应用程序的部署情况。您可以通过刷新 GitHub 中的“部署密钥”概述来验证您的 SSH 密钥是否用于提取您的更改。您的部署密钥现在应该是绿色的，而不是灰色的，并且应该有一条消息说“上周内最后一次使用”。

要执行您的 Ansible 行动手册，您可以使用以下命令:

```
ansible-playbook your-configuration-file.yml
```

# 结论

使用 SSH 从 GitHub 部署应用程序并不困难，您也不必让您的远程机器访问您的整个 GitHub 帐户。在本文中，我们将通过 GitHub 中的部署密钥来使用 SSH，只允许您的远程机器拉访问单个存储库，以安全轻松地部署您的应用程序。

我写这篇博文是为了分享我最近使用 Ansible 部署应用程序的发现。我可能会错过一些东西，因为我自己也是新手。新的发现总是会在新的博客文章中被提及，不准确的地方会在这篇文章中被修正，以确保我没有传播错误的信息。因此，如果您发现了错误，请让我知道，并帮助我将质量信息传播给其他软件工程师。

发布时间:2020 年 8 月 19 日

*最初发表于*[*【https://roelofjanelsinga.com】*](https://roelofjanelsinga.com/articles/ansible-easy-safe-ssh-deployments-from-github)*。*