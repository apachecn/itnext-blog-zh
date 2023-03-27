# 如何用一个命令自动部署你的应用？

> 原文：<https://itnext.io/how-to-auto-deploy-your-app-with-one-command-12f9ac00d34a?source=collection_archive---------1----------------------->

## 在本文中，我将描述我为应用程序准备自动部署解决方案的经验。此解决方案可以针对任何其他可以使用 Unix CLI 管理的技术进行修改。

![](img/fce702c6b0b8fcfd4479caff20d5fe28.png)

# 史前史

最近我开始了一个新项目。这应该是一个简单的 NodeJs 服务器应用程序，用于管理移动应用程序。我从头开始设置的。每个项目的一个有用之处是自动部署。它有助于减少开发人员的时间和客户的金钱。此外，任何操作的自动化降低了与人为错误相关的风险。

在我之前的项目中，我们使用了 MeteorUp。这是一个允许在一个命令中部署你的应用程序的工具。这很棒，因为它节省了时间，简化了开发人员的生活。

当我开始一个新项目时，我会把这个想法记在心里，并问自己:“嗯……为什么不呢？让我们在当前项目中也这样做吧！”

![](img/d17c067bfb1fc1675c6b4a055cb8eda6.png)

我做了一些研究，找到了一些解决方案。但是我一个都不喜欢，因为有些非常复杂，有些不可定制，不灵活。我不喜欢为简单的任务准备复杂的解决方案，所以我会试着为这个任务准备一些有效、简单、灵活的解决方案。

该应用程序将被部署到 EC2 VPS 实例。此外，无论您喜欢在哪里部署应用程序，主要要求是:

*   建立 ssh 连接的能力
*   VPS 上类似 Unix 的操作系统
*   本地机器上类似 Unix 的操作系统

# 理论

让我们从一些理论开始，以更好地理解我们做什么，我们最终应该得到什么。

![](img/e26e6e1330f8926c116b0fa61ef091f9.png)

因此，首先，我们假设，我们可以访问 VPS 和这个项目的所有环境。

对于节点项目，这应该是:

*   NodeJs
*   NPM
*   饭桶
*   配置 Nginx 在默认端口和应用端口之间重定向请求
*   开放端口(默认或其他)允许外部 HTTP 请求到我们的应用程序
*   用 Git app 克隆的。

如果所有的背景都准备好了，我们就可以开始设计算法步骤了。要将应用部署到 VPS，需要执行以下步骤:

*   从我们的本地机器连接到 VPS
*   将当前目录更改为 app 目录
*   从远程存储库(可以是 Github、BitBucket 等)获取最新的应用程序代码
*   停止应用程序实例(如果存在)
*   移除已安装的模块(节点应用特定操作)
*   安装新模块
*   运行最新版本的应用程序

所有这些操作都可以通过 Unix 命令行界面(CLI)来完成。同样，在类似 Unix 的操作系统中，我们获得了一些令人敬畏的能力来编写一些场景——shell 脚本。这是可以包含操作系统命令、变量、循环、条件操作符的文件，也可以使用 shell 脚本运行其他 shell 脚本文件，并通过 CLI 执行任何操作系统可以执行的操作。我觉得这一节的理论讲的就够了，下一节可以从一些实践开始。

# 实践

![](img/566c6a55d5c97314f81bf3f04f9e1169.png)

让我们来看一下上述算法的每一点，并回顾一下实施每项操作的 CLI 命令。

## 从我们的本地机器连接到 VPS

首先，我们应该与我们的服务器建立连接。出于这些目的，我使用 ssh 应用程序。它默认安装到 ubuntu 中，但是对于其他 OS 可以要求手动安装。

```
ssh remote_username@remote_host
```

在该命令之后，CLI 会询问您密码，如果您为远程连接输入了正确的密码，所有接下来输入的命令将被重定向到远程服务器，直到您关闭终端或键入“logout”命令。

我的 shell 脚本将存储在存储库中，所以我不喜欢将任何密码提交到脚本中，所以我将使用另一种方法来建立连接—权限密钥。AWS EC2 提供许可密钥文件来连接它们的服务器，而不是传统的密码方式。因此与使用键的连接将如下所示:

```
ssh -i permission_key_file_path remote_username@remote_host
```

如果提供了正确的密钥，连接将在没有任何密码的情况下建立。所以脚本可以存储在存储库中，但是密钥文件应该只有项目所有者。

## 将当前目录更改为 app 目录

这不是一个复杂的动作:

```
cd path_to_project_folder
```

## 从远程存储库中获取最新的应用程序代码

这里也一样。这是基于对 CLI Git 的了解。

```
git pull
git checkout branch_name
git pull origin branch_name
```

## 停止应用程序实例

这个要看你怎么管理你的 app 了。我更喜欢使用 [PM2](http://pm2.keymetrics.io/) 来管理我的应用程序。对于 pm2，这看起来像

```
pm2 stop app_id_or_name
```

## 移除已安装的模块并安装新模块

这是简单的删除文件夹和所有嵌套的文件夹/文件

```
rm -rf node_modules
npm install
```

## 运行最新版本的应用程序

用于此的同一个 pm2 工具。

```
pm2 start app_id_or_name
pm2 save
```

# 工作解决方案的构建

上面描述的所有命令片段看起来都很好，但是它们仍然不能解决我们的任务。让我们将上一节中描述的所有内容构建到一个可用的解决方案中。
照做就是了！

![](img/13db3f229e73b547eedf9ed881930af6.png)

首先，我们需要创建 shell 脚本文件。为此，只需用*创建一个文件。sh 分辨率。在我的例子中是“deploy.sh”

然后将您的第一条命令写入该文件

```
echo hello world!
```

并保存该文件。它将打印到控制台“hello world！”。
要运行您的简单脚本，您应该更改文件权限并允许它可执行。您可以使用操作系统的用户界面或使用简单的方式通过终端来完成:

```
chmod +x deploy.sh
```

现在，您的 shell 脚本可以使用下一个命令运行了:

```
sh deploy.sh
```

所以下一个自动部署脚本将被逐行描述。

脚本变量的初始化:

```
SSH_KEY_PATH="key.pem"
SERVER="remote_username@remote_host"
DEST_FOLDER="path_to_project_folder"
```

分支名称变量的初始化。可以将其指定为脚本参数，或者如果不指定该参数，则将“master”用作默认分支。

```
[[ $1 = '' ]] && BRANCH="master" || BRANCH=$1
```

使用之前声明的变量再构造一个变量

```
PARAMS="BRANCH=\"$BRANCH\" DEST_FOLDER=\"$DEST_FOLDER\""
```

变量“PARAMS”将用于通过 shh 连接传递参数

要使用访问密钥文件，应使用命令将其访问权限更改为 400。

```
chmod 400 key.pem
```

与服务器的连接

```
ssh -i $SSH_KEY_PATH $SERVER $PARAMS 'bash -i'  <<-'ENDSSH'
    #Server actions will be here
ENDSSH
```

为了连接到能够在其上执行命令的服务器，我使用

```
<<-'ENDSSH'
    #Server actions will be here
ENDSSH
```

！重要提示:使用正确的环境变量' bash -i '参数，通过 ssh 允许 exec 命令。没有它，npm 和节点路径将不可用。

以及通过 ssh 在远程服务器上执行的终端命令，以加载并启动最新版本的应用程序。

```
cd $DEST_FOLDER

git stash
# to stash package-lock.json file changes

git pull
git checkout $BRANCH
git pull origin $BRANCH

rm -rf node_modules/

npm install

pm2 stop app_name
pm2 start app_name
pm2 save
pm2 list

exit
```

这组命令可以针对任何其他技术进行更改，例如更改为另一组操作或针对其他技术执行相同的操作。

# 最终剧本

```
[[ $1 = '' ]] && BRANCH="master" || BRANCH=$1

SSH_KEY_PATH="key.pem"
SERVER="remote_username@remote_host"
DEST_FOLDER="path_to_project_folder"
PARAMS="BRANCH=\"$BRANCH\" DEST_FOLDER=\"$DEST_FOLDER\""

echo ===================================================
echo Autodeploy server
echo selected barcn $BRANCH
chmod 400 $SSH_KEY_PATH
echo ===================================================
echo Connecting to remote server...
ssh -i $SSH_KEY_PATH $SERVER $PARAMS 'bash -i'  <<-'ENDSSH'
    #Connected

    cd $DEST_FOLDER

    git stash
    # to stash package-lock.json file changes

    git pull
    git checkout $BRANCH
    git pull origin $BRANCH

    rm -rf node_modules/

    npm install

    pm2 stop app_name
    pm2 start app_name
    pm2 save
    pm2 list

    exit
ENDSSH
```

# 结论

结果，我得到了一个脚本，当我需要部署应用程序时，它可以让我节省时间。我意识到存在一些其他的解决方案，也许在其他情况下会更好。但是对我来说，如果你想为一个小项目快速灵活地设置部署过程，这是一个很好的解决方案。

此外，这种远程服务器管理方法可以用于其他一些任务和管理其他一些技术

如果这篇文章对你有用，你可以鼓掌(即使是 50 次))))
如果你有什么问题，欢迎在评论中问我。