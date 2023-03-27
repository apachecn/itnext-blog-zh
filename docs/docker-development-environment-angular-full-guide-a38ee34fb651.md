# Docker 开发环境:Angular —完整指南

> 原文：<https://itnext.io/docker-development-environment-angular-full-guide-a38ee34fb651?source=collection_archive---------1----------------------->

![](img/51457f6d314662ef9629aaba047590d7.png)

让我们假设您刚刚格式化了您的 windows 笔记本电脑，并且在下载 Nodejs LTS 版本之前享受了您的全新安装。另一个有趣的旅程是在本地机器上安装 Docker、VSCode、enable WSL。当然，事实上，我们使用 Windows 而不是 Unix 的替代品将“花费”更多的步骤，但最终的外观和感觉将是完美的，笔记本电脑比以往任何时候都干净。

这些步骤如下:

*   [x]全新 Windows 安装
*   [x]咖啡
*   [ ]下载&ιinstall Docker(启用 WSL 2)、Git、WSL、VSCode
*   [ ]配置 WSL、Docker、Git、VSCode
*   [ ]最后步骤和奖励

*我们开始吧！*

# **下载&安装提示**

**码头工人**

*   在 Docker 安装期间，您应该检查*安装 WSL 2 所需的 Windows 组件*
*   确保在 *Docker >设置> General* 中选择**使用基于 WSL 2 的引擎**

**饭桶**

*   安装 Git for windows 并设置所需的凭据，以便从/向您的存储库克隆/拉/推。使用**个人访问令牌**并将其存储在 **Windows 凭证**中

**VSCode**

*   将 VSCode 添加到 Windows 路径
*   打开 VSCode 安装 **Remote — WSL 扩展(微软)**

**WSL**

*   以管理员身份运行 CMD 或 PowerShell 并键入`wsl -l -o`。您将看到可用的发行版
*   通过运行`wsl --install -d Ubuntu-20.04`安装一个，例如 Ubuntu-20.04

# **配置**

**WSL**

*   运行`wsl -l -v`你会注意到 Ubuntu 发行版的版本是 1，默认子系统是 docker-desktop。
*   用`wsl --set-version Ubuntu-20.04 2`升级 v2 中的发行版
*   将 v2 设置为 furure 安装的默认版本`wsl --set-default-version 2`
*   为 Docker `wsl --set-default Ubuntu-20.04`设置默认发行版

**码头工人**

*   导航到*Docker>Settings>Resources>WSL INTEGRATION*并通过点击切换开关启用您的新发行版，然后点击 *Apply & Restart*
*   运行`docker build -t angular12 .`这是我的 Docker 文件，用 Angular 创建一个 Docker 图像:

```
FROM node:14-alpineRUN npm install -g @angular/cliUSER nodeWORKDIR /appEXPOSE 4200 49153CMD npm start
```

*   打开 cmd 并运行`wsl`。您可以在主目录中为您的项目创建一个文件夹
*   现在用`docker run -it -v $(pwd):/app -p 4200:4200 -p 49153:49153 --name ng angular12 sh`创建一个 Docker 容器(在 linux 发行版的项目文件夹中)
*   运行`ng new my-app --skip-git`和`chmod -R 777 ./*`
*   打开一个新的 cmd，键入`wsl`，导航到新项目的文件夹并运行`code .`(如果您的路径中没有 VSCode，请按照 Remote-WSL 扩展中的步骤操作，以便在上面的位置打开它)

# **最终步骤&奖金**

*   打开 **package.json** 文件，将`”start”: “ng serve”,`替换为`”start”: “ng serve --host 0.0.0.0 --poll”,`
*   从容器内部运行`npm start`并导航至`http://localhost:4200`
*   通过运行以下命令在 WSL 中配置 git:

```
git config --global user.name “Your Name”git config --global user.email “youremail@domain.com”git config --global credential.helper “/mnt/c/Program\ Files/Git/mingw64/libexec/git-core/git-credential-manager-core.exe”
```

*   运行`git init`，**提交**项目中的变更，并**将**推至远程

# **奖金**

*   当 WSL 运行时，您可以使用 windows 资源管理器在`\\WSL$`位置导航
*   更新自动生成的*。编辑器配置*文件。找到`[*]`，在它下面增加一条新规则:`end_of_line=crlf`。还包括一个新的类别`[*.sh]`并将`end_of_line=lf`放在其下

> 如果你更喜欢带有 Dockerfile 和 README 的 git repo，而不是这篇文章，请在此处查看[](https://github.com/stavrosdro/angular-docker-guide)*。*

**享受*🚀🚀🚀*