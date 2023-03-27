# 优化应用程序监控流程。或者如何用一个命令从服务器获取应用程序日志。

> 原文：<https://itnext.io/optimizing-app-monitoring-process-or-how-to-fetch-app-logs-from-the-server-with-one-command-7fdf5d9a936?source=collection_archive---------5----------------------->

## 这篇短文描述了我在优化开发过程中的经验。我将描述如何用一个命令将服务器日志提取到我的本地机器中。

在我以前的文章中，我描述了一种自动化大多数远程服务器操作的方法。在本文中，我将描述 shell 脚本编程的一些额外的有用功能。如果你没有看到我之前的文章，请先看看这个[链接](https://medium.com/@bagrijroman/how-to-auto-deploy-your-app-with-one-command-12f9ac00d34a)。

# 初始任务

在当前的工作中，需要不时地监控服务器日志。正如你可能已经猜到的，这个问题可以用两种方法解决——普通的更简单的方法，和更有趣的自动化方法。

一种更简单的方法是通过 ssh 或 ftp 连接到服务器，并查看服务器日志进行分析。

其次，更有趣的自动化方法是准备 shell 脚本，它将在一个命令中完成大部分工作。这个想法是获取本地机器的日志并进行分析。这种日志处理方式将减少连接服务器和手动获取日志的时间。

![](img/dfc9798884bf89a5ffa47543f6d3dcb7.png)

# 我如何解决任务

我准备下一个算法来解决我的任务

*   为服务器应用程序日志创建(如果之前未创建)本地文件夹
*   通过网络将服务器日志文件夹复制到本地目标文件夹
*   使用操作时间戳重命名复制的文件夹

经过这三个简单的步骤后，服务器日志副本将出现在我的本地计算机上，我可以用方便的方式分析它们，用任何 UI 编辑器而不是 CLI 编辑器。在我看来，在 cli 界面中读取数千行日志并不是最好的主意。

# 从理论到实践

首先，我声明将在我的脚本中使用的变量。我更喜欢将值移动到变量中，因为这使我的脚本更加清晰和灵活。此外，在类似的情况下，只需更改变量声明块中的一些值，就可以重用它。

```
DESTINATION_FOLDER='logs_dump'
DUMP_FOLDER_NAME=`date +"%Y-%m-%d_%T"`
SSH_KEY_PATH="./key.pem"
SERVER="<user>@<server_address>"
SERVER_PROJECT_FOLDER="<project_folder_absolute_path>"
SERVER_LOGS_FOLDER="<logs_folder_path>"
```

DESTINATION_FOLDER —本地计算机上将日志文件夹复制到的文件夹。

转储文件夹名称—基于操作时间戳的复制日志文件夹的新名称。

SSH_KEY_PATH —权限密钥文件的路径。

服务器—连接远程服务器的 ssh 地址。

SERVER_PROJECT_FOLDER —远程服务器上项目文件夹的路径。

SERVER_LOGS_FOLDER —基于项目文件夹路径的日志文件夹路径。在我的例子中，日志将存储在项目目录中的一个单独的文件夹中。

## 第二个动作

为日志创建(如果不存在)本地文件夹，并将目录更改为该文件夹。

```
mkdir -p "$DESTINATION_FOLDER"
cd "$DESTINATION_FOLDER"
```

## 第三个动作

将日志文件夹从远程服务器复制到上一步创建的文件夹中

```
scp -r -i $SSH_KEY_PATH $SERVER:$SERVER_PROJECT_FOLDER/$SERVER_LOGS_FOLDER ./
```

出于这些目的，我使用 scp 命令。它允许在两个目标点之间拷贝文件，如果其中一个是远程主机。例如，将文件或文件夹从本地机器复制到远程服务器或相反方向。

选项"-r "允许对整个目录进行递归复制。

选项“-i”选择标识(私钥)来自的文件。

## 第四项行动

使用操作时间戳值重命名复制的日志文件夹。

```
mv $SERVER_LOGS_FOLDER $DUMP_FOLDER_NAME
```

经过上述步骤，我的项目中的一个常规任务变得自动化。

![](img/72c518056d0d53d519f9077738decd52.png)

整个脚本代码:

```
#==================VARIABLES_SECTION=======================
DESTINATION_FOLDER='logs_dump'
DUMP_FOLDER_NAME=`date +"%Y-%m-%d_%T"`
SSH_KEY_PATH="./key.pem"
SERVER="<user>@<server_address>"
SERVER_PROJECT_FOLDER="<project_folder_absolute_path>"
SERVER_LOGS_FOLDER="<logs_folder_path>"
#==========================================================

echo ======================================================
echo Create sever logs dump...
echo ======================================================

mkdir -p "$DESTINATION_FOLDER"
cd "$DESTINATION_FOLDER"

echo Fetching server log files...

scp -r -i $SSH_KEY_PATH $SERVER:$SERVER_PROJECT_FOLDER/$SERVER_LOGS_FOLDER ./
mv $SERVER_LOGS_FOLDER $DUMP_FOLDER_NAME

echo fetched...
echo ======================================================

echo Dump folder $DESTINATION_FOLDER/$DUMP_FOLDER_NAME
echo ======================================================
```

# 结论

在这篇文章中，我以通过网络在本地和远程机器之间复制一些文件的方式描述了我的任务自动化的经验。这种能力有助于我发现服务器日志，但是如果需要的话，这种方法也可以用于任何其他需要通过网络复制文件的任务。

如果这篇文章对你有用，你可以在这里留下你的掌声(即使是 50 次),或者问我一个问题。