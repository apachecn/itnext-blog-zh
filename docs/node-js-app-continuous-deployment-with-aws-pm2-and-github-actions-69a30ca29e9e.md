# 通过 AWS、PM2 和 Github 操作持续部署 Node.js 应用程序。

> 原文：<https://itnext.io/node-js-app-continuous-deployment-with-aws-pm2-and-github-actions-69a30ca29e9e?source=collection_archive---------1----------------------->

所以你有了这个 **node.js** 应用。它不是静态的，它是在 Vue 中，React，Angular，无论什么…它已经准备好并稳定地发布和部署。但是在哪里，怎么做？在本文中，我将介绍如何自动部署您的应用程序并保持其持续交付的一些步骤。

![](img/ef64ea3e314f250a0f5f064d050e6ee6.png)

**下面是一个快速总结**:

1.  在 AWS 上运行 EC2 实例
2.  PM2 设置
3.  部署脚本
4.  GitHub 操作

题外话:如果你的应用没有任何后端相关的功能，我几乎可以肯定它可以用静态模式构建。因此，你可以使用 ie [Netlify](https://netlify.com/) 并跳过这个线程。🙃

**静态应用产生较小的碳足迹**。

好了，让我们回到正题。

**1。在 AWS 上运行 EC2 实例。**

在这种情况下，我们将在 AWS 上使用 EC2。如果您不熟悉安装过程，您需要做的就是:

*   [在 AWS 上创建账户](https://aws.amazon.com/)
*   [启动您的 EC2 实例](https://docs.aws.amazon.com/efs/latest/ug/gs-step-one-create-ec2-resources.html)
*   [创建 EC2 密钥对](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

别担心，这非常简单，我们不需要移动。只要看指导方针就没问题了。然而，也许对你来说，数字海洋水滴将是最好的解决方案；或者任何其他的。用你最喜欢的，我们只需要一个普通的 Ubuntu 实例。但是，请注意下面配置中的路径是特定于 AWS 的。

现在，登录到您新创建的实例。

```
ssh -i key.pem ubuntu@12.345.67.891
```

当然，对于新机器，您需要配置 web 服务器，很可能是 Nginx。关于这个的更多信息，你可以在这里找到或者直接使用这个预定义的。

```
$ sudo vim /etc/nginx/conf.d/nodejs-basic.conf
```

**2。PM2 设置。**

PM2，搞什么鬼？PM2 是一个守护进程管理器，它将帮助你管理和保持你的应用程序 24/7 在线。因此，这将有助于我们的应用程序保持活跃和运行。让我们安装它。

```
npm install pm2 -g
```

然后，在您的主文件夹中创建一个简单的配置文件(`eccosystem.json`)。

**3。部署脚本。**

因此，我们需要一些简单的 bash 脚本(`deploy.sh`)来运行应用程序构建，然后调用 PM2 过程。

你可以使用`sh deploy.sh`命令来测试它。

太好了，我们快到了。服务器配置相对简单，您几乎可以毫不费力地完成。如果您的日常环境是 MacOS 或 Linux，这将非常简单。

现在——只是为了测试——我们可以手动运行整个过程。为此，请遵循以下步骤。

*   配置 web 服务器
*   在实例上克隆/获取您的应用程序
*   构建应用程序
*   安装 PM2 管理器
*   运行`sudo pm2 start npm --name "process" -- start`

或者将其设置为实例设置的简单清单。

**4。GitHub 行动。**

最后，我们可以创建一个 GitHub 动作来运行我们的部署流程。对于我们的例子，我们将创建一个动作，每次合并到`develop`分支时都会调用这个动作。

在你的主应用程序目录中创建`.github`文件夹，里面有`workflows`文件。然后创建一个`deploy.yaml`文件。

如你所见，这里有一些变量。这些是 GitHub 特有的秘密。您可以在您的帐户设置中定义它们。AWS 相关凭证的值可以在控制台中找到。[这里](https://aws.amazon.com/blogs/security/wheres-my-secret-access-key/)你有一些详细的说明。

**记住！始终将敏感数据放在存储库之外。**

仅此而已。将工作流推进到您的 repo 中，并等待您的第一次自动部署。大约需要几分钟(最多)。完成了。

干杯，卢卡斯。