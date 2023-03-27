# 使用 Python 的云本机 RPA

> 原文：<https://itnext.io/cloud-native-rpas-with-python-a22cdb6690d0?source=collection_archive---------5----------------------->

![](img/fd381c9677b99661a99b5d39b93b6101.png)

**使用可破坏的基础设施在 Python 中构建云原生 RPA**

在本文中，我们将:

*   设置虚拟桌面来开发和运行 RPA。我们的 RPA 将在 docker 容器内运行的 linux 虚拟桌面上运行。这允许我们尽可能地将代码从基础架构中分离出来，并确保“云原生”RPA。
*   构建一个从 SEC 官网下载财务数据的 RPA。由于 SEC 数据库名为 EDGAR，我们将称此 RPA 为“EDGAR_investigator”。
*   确保我们的 RPA 与其持久数据分离，并且有可能在任何基础架构上运行

要求:

*   对 Python、RPA 和业务流程有基本了解
*   安装在服务器上的 Docker Desktop 或 Docker and Compose(运行和开发我们的云原生 RPA)

关于 RPA 是什么以及开发它们时的一些最佳实践的概述，请查看我以前的文章[文章](/the-modern-enterprise-business-in-code-c6a5e0f4ed7e)。

本教程的所有内容都可以在以下存储库中找到:

【https://github.com/dkatz23238/pybotlib-tutorial 

**介绍 Pybotlib**

Pybotlib 为几个 python 库提供了一个高级包装器，并为用 Python 开发 RPA 提供了一些最佳实践。让我们回顾一下每个 pybotlib RPA 都必须经历的一些基础知识。完成后，我们将开始使用 python 代码和预构建的 docker 映像构建我们的云本机 RPA。

在 pybotlib 中，您可以创建一个“虚拟代理”对象来维护执行业务流程所需的整体状态和行为，而不是让解耦的任务按顺序执行。它代表了您的 RPA 及其功能。要创建 VirtualAgent 对象，必须指定以下参数:

*   bot _ name:RPA 的唯一标识符
*   downloads_directory:存储通过互联网下载的文件的位置
*   firefoxProfile:主机上的目录，firefox 可以在其中找到 profiles.ini 文件，并在加载 firefox webdriver 时实例化一个配置文件。通常位于/home/$USER/。mozilla/firefox

所有操作都是通过单个对象实例 pybotlib.VirtualAgent 执行的。一旦我们创建了 VirtualAgent 对象，我们将继续为它创建一个日志文件来记录消息，最后打开一个 web 浏览器。要开始使用 RPA 浏览 web，您必须调用 get_geckodriver 脚本，该脚本下载 firefox-geckodriver 的最新版本并将其放在当前工作目录中。创建 RPA 日志文件不需要任何依赖关系。

关于 pybotlib 的所有细节可以通过下面链接中的文档获得:

 [## 欢迎使用 pybotlib - pybotlib 0.0.1 文档

### 编辑描述

pybotlib.readthedocs.io](https://pybotlib.readthedocs.io/en/latest/) 

在下一节中，我们将启动一个虚拟桌面来运行以下代码。

以下代码显示了大多数 RPA 脚本应该从使用 pybotlib 开始。它导入所需的包，下载最新的 geckodriver，实例化 VirtualAgent，初始化 web 驱动程序，最后访问某个网站(在本例中是 google.com)。

```
import os
from pybotlib import VirtualAgent
from pybotlib.utils import get_geckodriver

# Downloads the geckodriver
get_geckodriver()
# Any file downloaded will be saved to the current working dir
# under a folder called bot_downloads
bot_download_dir = os.path.join(os.getcwd(),"bot_downloads")

mybot=VirtualAgent(
    bot_name="my_first_robot",
    downloads_directory=bot_download_dir,
    # In order to load specific profiles from host machine
    firefoxProfile="/home/$USER/.mozilla/firefox" )
# Start a web browser that can be accessed via mybot.driver or other mybot methods
mybot.create_log_file()
mybot.initialize_driver()
mybot.get("http://www.google.com")
mybot.log("I am accessing google")
```

一旦我们实例化了 RPA，我们将继续规划和协调它需要执行的一系列活动。在我们脚本的最后，我们将做一些清理工作:

```
mybot.driver.quit()
mybot.log_bot_completion() 
```

在 RPA 脚本运行期间的任何时候，都可以通过 pybotlib.driver 访问控制 firefox 的 selenium webdriver，或者您可以调用 webdriver 之上的其他方法。一些便利的方法可以让浏览网页变得更容易，其中之一就是“virtual agent . find _ by _ tag _ and _ attr()”。

该方法将返回所有带有特定标签和属性的 HTML 元素，这些标签和属性对应于您决定的一些评估字符串。VirtualAgent 对象必须已经实例化，并且 webdriver 已经初始化。
如果我们想使用 className = "hugebutton "搜索网页上的所有按钮，您可以使用以下代码:

```
mybutton = mybot.find_by_tag_and_attr(“button”, “className”, “hugebutton”, 1)
```

您还可以直接与 webdriver 交互，并使用所提供的任何 Selenium 方法。

```
# Example
mybot.driver.refresh()
```

**设置我们的环境**

现在我们知道了基础知识，让我们开始创建我们的 RPA 代码。最佳做法是让我们的整个 RPA 通过单个入口点运行，例如名为 run_RPA.py 或类似的 python 程序。为了实现这一点，我们将组织我们的代码。

分离输入/输出数据并使 RPA 执行尽可能无状态也是最佳做法。我们开发的任何 RPA 都应该能够旋转，“检查”是否有未完成的工作要做，并相应地以最少的人工交互来执行这项工作。

使用云文件存储是分离输入和输出数据的最佳方法。在本例中，我们将使用 Google Sheets 电子表格选项卡作为业务流程的输入，我们将启动一个 minio 对象存储容器，监听端口 9000 以存储我们的输出数据文件。我们的输入文件将是一个电子表格，其中包含需要查找的公司，我们的输出将只是一个文件夹，其中包含来自上述公司的 SEC 报告。我们的 RPA 应该能够加速旋转，检查是否有待处理的行，处理它们，然后保存输出。让我们创建我们的 RPA 将在其上运行的可变基础架构。一个包含两个图像的简单 docker 文件就足够了，一个容器将运行完整的 Ubuntu 桌面环境，另一个将运行我们的文件存储(在这种情况下，我们将使用 minio)。我们可以为输出使用任何类型的对象存储，只要我们将 RPA 设计为“可销毁”,即任何需要持久存储的数据都存储在桌面环境之外的某个地方。对于您的组织来说，拥有一个包含所有持久数据的中央虚拟桌面可能更实用，但是您横向扩展 RPA 操作的规模越大，拥有一个不可变的基础架构就越方便。我们也可以使用 minio 作为我们的数据输入源，但是使用 google sheet 会比手动编辑电子表格并上传更加用户友好。

现在让我们回到编码上来。

首先让我们创建一个“docker-compose.yml”。

该文件的内容应该如下:

```
version: "3"

services:
  virtual-desktop:
    image: dorowu/ubuntu-desktop-lxde-vnc:bionic-lxqt
    ports:
      - "5910:5900"
    environment:
      - USER=robot
      - PASSWORD=robot
      - MINIO_ACCESS_KEY=V42FCGRVMK24JJ8DHUYG
      - MINIO_SECRET_KEY=bKhWxVF3kQoLY9kFmt91l+tDrEoZjqnWXzY9Eza
  minio:
    hostname: minio
    image: minio/minio
    container_name: minio
    ports:
      - "9000:9000"
    environment:
      - MINIO_ACCESS_KEY=123456
      - MINIO_SECRET_KEY=password
    command: server /data
```

现在我们要做的就是启动我们的容器，并开始开发我们的 RPA。

```
docker-compose up -d
```

我们现在可以使用任何 VNC 客户端连接到我们的虚拟桌面。您可以在 windows 或 Mac 上使用 RealVNC。Ubuntu 有一个内置的远程桌面客户端，可以创造奇迹。桌面应该可以通过 [http://127.0.0.1:5910](http://127.0.0.1:5910`) 访问。如果您想通过浏览器访问桌面，您可以映射 noVNC http 端口并通过浏览器访问它。请注意，默认情况下，安全性是关闭的。在虚拟桌面中创建的用户称为 robot，其密码为 robot。这可以在 docker-compose.yml 文件中更改。

需要安装 python 3.7 并将其设置为默认版本的初始配置。建议快速检查以确保 firefox 能够正常运行。在资源非常有限的情况下运行可能会导致问题。

容器启动后，通过远程桌面连接到 Ubuntu 容器，并打开一个终端(Shift + Ctrl + t)。运行以下命令进行初始配置:

```
sudo apt-get update && sudo apt-get install python3.7 nano python3-pip curl wget git && alias python=python3.7 && echo ‘alias python=python3.7’ >> /home/robot/.bashrc
# Remember that the password for robot is robot
python -m pip install pybotlib ipython
```

在这些命令中，我们安装了 python 3.7，将其设置为使用 python word 时调用的默认命令，并从 python 包索引中安装了 pybotlib。出于调试目的，我们还安装了 Ipython 解释器。我们可以打开交互式解释器，检查我们的安装是否正确。

首先通过运行以下命令启动 IPython:

```
python — m IPython
```

然后键入以下命令，并确保它成功执行。

```
import pybotlib
```

**RPA 开发**

让我们在虚拟桌面中设置一个目录结构，并将其命名为 edgar-RPA。我们将创建一个名为 run_RPA.py 的文件。
这都可以从命令行完成。在虚拟桌面内打开另一个终端，并执行:

```
mkdir /home/robot/Desktop/edgar-RPA
touch /home/robot/Desktop/edgar-RPA/run_RPA.py
```

我们将在 run_RPA.py 文件中写入我们的 RPA。您可以在您的主机上编辑文档，然后通过远程客户端将您的代码复制并粘贴到在虚拟桌面内运行的默认文本编辑器中，也可以直接在虚拟桌面内编辑。如果你选择后者，我推荐你下载 atom 到虚拟桌面上作为文本编辑器，因为默认程序对于编写代码来说是非常可怕的。如果你想回到过去，你可以使用 nano 或 vim 在虚拟桌面的终端中编写代码。

如果您想使用 nano，请确保从虚拟桌面的终端安装它。

```
sudo apt-get install nano
```

我倾向于在主机上使用 visual studio 代码或 atom，从编辑器中复制并粘贴文本。

让我们用前面几节中的初始 pybotlib 样板代码填充 run_RPA.py。我们将在末尾添加几行代码来退出 web 浏览器。

```
import os
from pybotlib import VirtualAgent
from pybotlib.utils import get_geckodriver
from time import sleep
# Downloads the geckodriver
get_geckodriver()
# Any file downloaded will be saved to the current working dir
# under a folder called bot_downloads
bot_download_dir = os.path.join(os.getcwd(),"bot_downloads")

mybot=VirtualAgent(
  bot_name="my_first_robot",
  downloads_directory=bot_download_dir,
  # In order to load specific profiles from host machine
  firefoxProfile="/home/robot/.mozilla/firefox" )
# Start a web browser that can be accessed via mybot.driver or other mybot methods
mybot.create_log_file()
mybot.initialize_driver()
mybot.get("http://www.google.com")
mybot.log("I am accessing google")
sleep(5)
mybot.log_bot_completion()
mybot.driver.quit()
```

现在，让我们通过在虚拟桌面的终端中运行以下命令来测试到目前为止一切正常。

```
python run_RPA.py
```

确保我们当前的工作目录是包含我们代码的文件夹。Firefox 应该会打开并访问 google，然后在 5 秒钟后关闭。您还可以通过键入 echo $来检查程序的退出状态。，除 0 之外的任何值都意味着出错。

在我们继续之前，让我们将项目文件夹初始化为一个 git 存储库，它可以在 Ubuntu 虚拟桌面的潜在实例中克隆和执行。
你可以使用任何你喜欢的在线回购，在这种情况下，我将在 github UI 中创建一个私有的 github 存储库，url 如下:【https://github.com/dkatz23238/testfinanceRPA.git ，然后在我目前工作的虚拟桌面中执行以下命令。

```
cd /home/robot/Desktop/edgar-RPA
git init
git add .
git commit -m “first commit”
git remote add origin [https://github.com/dkatz23238/testfinanceRPA.git](https://github.com/dkatz23238/testfinanceRPA.git)
git push -u origin master
```

现在我们可以跟踪代码的进度并对其进行版本化。

让我们从创建 RPA 的功能元素开始。我建议我们创建一个名为 RPA_activities.py 的单独文件，将我们的活动从该文件导入 run_RPA.py 文件，并从该文件执行我们的 RPA。

```
touch /home/robot/Desktop/edgar-RPA/RPA_activities.py
```

如前所述，RPA 将从 SEC 网站下载具体的财务报告。任何 RPA 通常都需要输入和输出业务数据，选择使用什么可以有所不同，但对于这个 RPA，我将选择使用 Google Sheets 电子表格作为输入，并使用 minio 对象存储作为文档输出。我们可以在 GSheet 中创建一个电子表格，并生成一个仅供 RPA 使用的查看 URL。这些数据可以由我们的 RPA 读入熊猫。数据帧使用:

```
pybotlib.utils.pandas_read_google_sheets(sheet_id)
```

我们的 RPA 的功能元素将是一个函数，它将 VirtualAgent 实例作为输入，并通过导航网站的前端从 SEC 网站下载一系列报告。将定义第二个函数来实例化我们的 VirtualAgent，然后将数据保存到我们选择的相应存储中。我们必须向 RPA 传递一些 env 变量，它将使用这些变量来访问不同的附加资源。现在，我们将在执行 RPA 之前在 shell 中定义它们。首先让我们对我们的文件进行一些修改，以便在一个文件中包含我们的活动，然后我们的第二个文件将简单地导入活动，并按照我们定义的方式执行它们。在本文中，我不会深入研究 RPA 如何下载报告的细节，这纯粹是通过 pybotlib 中提供的 selenium webdriver 和 helper 函数导航前端来完成的。

RPA_activities.py

run_RPA.py

如您所见，我们已经将代码和业务逻辑移到 RPA_activites.py 文件中，我们的 run_RPA.py 文件只是将 RPA 作为 RPA 的入口点来调用和运行。

关于我们导航 SEC 网页前端的具体方法，请参考 Selenium webdriver 文档以及 github 中 pybotlib 的 README.md。

此时，我们要做的就是定义环境变量并执行 RPA。请记住，我们有一个 minio 实例正在运行，RPA 将在其中保存数据。

我们的 RPA 代码利用了必须预定义的环境变量，以便 RPA 访问某些连接的资源或出于其他原因。环境变量是构建代码的一个很好的工具，它可以很容易地变得尽可能无状态，并使用某些信息(如 URL、密钥和密码)来访问附加的资源。这是 12 因素应用程序开发方法的基本概念，您可以从这里了解更多信息

从虚拟桌面终端内执行的以下代码应该可以端到端地运行我们的 RPA。

```
export MINIO_URL=minio;
export MINIO_ACCESS_KEY=123456;
export MINIO_SECRET_KEY=password;
export MINIO_OUTPUT_BUCKET_NAME=financials;
export GSHEET_ID=1pBecz5Db9eK0QDR_oePmamdaFtEiCaO69RaE-Ozduko;
export DISPLAY=:1;
python run_RPA.py
```

RPA 完成后，我们可以通过 minio URL 中的浏览器访问文件。
我们的最后一步是创建一个可以在虚拟桌面内的任何地方执行的单一 shell 脚本，并运行我们的 RPA。

我们需要做的就是将上述命令添加到 shell 脚本中，使我们成为 RPA 的单一入口点。我们可以将 RPA 的全部内容推送到 git 存储库，从头开始测试克隆存储库并重新执行 RPA。这将确保我们的 RPA 满足“可破坏条件”。让我们让我们的 RPA 在做最后一次推送之前不打印我们的 env 变量，从而使我们的 RPA 更加安全友好。

目前，此 RPA 足以演示本文的目的，但是对此 RPA 的增强将是添加测试、更好的异常处理和自动化部署。这些主题将在下一篇文章中讨论。

我们的最终项目将包含以下文件:

埃德加-RPA
—run _ RPA . py
—RPA _ activites . py
—run _ RPA . sh

将目录切换到文件夹并运行/bin/bash run_RPA.sh 应该可以运行我们的端到端流程。最后，我们可以添加初始命令来下载依赖项并为 run_RPA.sh 脚本分配别名，将其推送到 git，然后进行克隆和重新运行测试，以确保我们的 RPA 可以在运行我们特定虚拟桌面映像的任何 docker 容器中的任何基础架构上运行。

我们还可以向 run_RPA.sh 添加环境变量，以便从远程 shell 执行 RPA，而无需登录 GUI。为此，我们可以将以下环境变量添加到 run_RPA.sh 脚本中。这个上面已经加了。

```
export DISPLAY=:1
```

我们最终的 run_RPA.sh 脚本将如下所示

```
#! /bin/bash
export MINIO_URL=minio;
export MINIO_ACCESS_KEY=123456;
export MINIO_SECRET_KEY=password;
export MINIO_OUTPUT_BUCKET_NAME=financials;
export GSHEET_ID=1pBecz5Db9eK0QDR_oePmamdaFtEiCaO69RaE-Ozduko;
export DISPLAY=:1;
# Pip will only use the current users directory
python3.7 -m pip install — user robot — no-cache-dir pybotlib;
python3.7 run_RPA.py;
```

现在，我们可以从主机进入虚拟桌面，远程执行 RPA。
为了测试此功能，我们可以从我们的主机访问一个正在运行的 shell，以访问 RPA 要在其上运行的虚拟桌面。在包含 docker-compose-yml 文件的目录中，下面的命令将从我们的主机启动客户机中的一个 shell。注意，产生的新 shell 对应于容器，而不是主机。从这里，我们可以进入 code directory 文件夹，将 RPA 作为 bash 命令运行，并通过我们的远程桌面客户端观察我们的启动是否正常，就像您通过用户界面登录一样。

```
docker-compose exec — user robot virtual-desktop /bin/bash# From within the new shell we can navigate to our code directory # and run the shell script.
sudo -H -u robot bash -c bash run_RPA.sh
```

此时，我们可以将虚拟桌面映像部署到任何支持 docker 的基础架构，克隆我们新创建的 RPA 并执行它。这可能会让您节省大量计算时间成本，因为您可以根据需要扩展大量 RPA 来满足工作需求。

我希望你喜欢我的文章，并随时通过我的任何社交媒体档案与我联系。