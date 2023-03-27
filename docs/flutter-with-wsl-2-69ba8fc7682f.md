# 与 WSL 2 一起颤动

> 原文：<https://itnext.io/flutter-with-wsl-2-69ba8fc7682f?source=collection_archive---------1----------------------->

![](img/ebf199e6eb6120a033633d77b24a65ff.png)

成功就像一场派对。

在假期中，我有时间查看我的开发环境，并决定是时候使用 WSL2 了。

WSL2 确实给了我承诺的性能改进，但我在用 flutter 进行前端开发时遇到了一个问题(我认为这也会影响 react native)，所以我想我应该帮助其他想实现这一飞跃的人。

# 环境设置

因为有很多关于不同部分的文章，所以我不打算花时间重复细节，只是给你一个我的开发环境的概要。

*   Windows 10 专业版
*   WSL 2 (Ubuntu 19.04)
*   VS 代码(1.41.1)
*   安卓工作室(3.5.3)

# Visual Studio 代码远程 WSL

我使用 Visual Studio 代码进行大部分开发，所以当我找到访问远程 WSL 的扩展时，我非常兴奋。如果您想使用 WSL 2 并获得文件系统性能提升，这是绝对必要的。

[](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) [## 远程- WSL

### Remote - WSL extension 扩展允许您将 Windows Subsystem for Linux (WSL)用作您的全职开发…

marketplace.visualstudio.com](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) 

# Android SDK

我想让所有东西都在 WSL 2 容器中运行，所以我在 Ubuntu 中安装了 Android SDK。它也设置在我的 Windows 端，但我的目标是尽可能不使用它。

下面是我做的设置。您需要将导出内容移动到您的。bash_profile 一旦你得到这个工作。

```
cd ~
sudo apt-get install unzip zip
wget https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip
unzip sdk-tools-linux-4333796.zip -d Android
rm sdk-tools-linux-4333796.zip
sudo apt-get install -y lib32z1 openjdk-8-jdk
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
printf "\n\nexport JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64\nexport PATH=\$PATH:\$JAVA_HOME/bin" >> ~/.bashrc
cd Android/tools/bin
./sdkmanager --install "platform-tools" "platforms;android-26" "build-tools;26.0.3"
export ANDROID_HOME=~/Android
export PATH=$ANDROID_HOME/tools:$PATH
export PATH=$ANDROID_HOME/tools/bin:$PATH
export PATH=$ANDROID_HOME/platform-tools:$PATH
printf "\n\nexport ANDROID_HOME=~/Android\nexport PATH=\$PATH:\$ANDROID_HOME/tools\nexport PATH=\$PATH:\$ANDROID_HOME/platform-tools" >> ~/.bashrc
sdkmanager --update
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install gradle 5.5.1
gradle -v
```

# 摆动

我需要的最后一件事是让 flutter linux 安装运行起来。仅仅使用标准的 linux flutter 安装对我来说就足够了。

[](https://flutter.dev/docs/get-started/install/linux) [## Linux 安装

### 要安装和运行 Flutter，您的开发环境必须满足这些最低要求:操作系统:Linux…

颤振. dev](https://flutter.dev/docs/get-started/install/linux) 

安装完成后，请务必运行 flutter doctor，看看您是否有任何问题。我不得不更正 SDK 的位置并接受一些许可，但不需要安装 Android Studio。

此外，确保启动 VS 代码并使用远程 WSL 扩展连接到您的环境。一旦你连接上了，你就需要安装 flutter 扩展以及你想要运行的任何其他扩展，看起来远程 WSL 扩展在 ubuntu 环境中单独设置 vs 代码。

# Android 模拟器

到目前为止，一切都很顺利。我能够拥有一个正常工作的 flutter 环境，但却无法将我的应用程序编译到仿真器或设备上。我找到了两个解决方案，一个是如果你愿意使用真实的设备，另一个是如果你想使用模拟器。

# 亚行 TCPIP

如果你正在使用一个真实的设备进行测试，这是一个非常有用的设置方法。它让你告诉设备，客户端连接到它将通过 TCP 来代替 USB。要进行设置，您只需将您想要测试的设备连接到 windows。在 Windows cmd 提示符下，确保使用以下命令连接设备

> adb 设备

如果您看到想要连接的设备，请按

> 亚行 tcpip 5555

这样做将使设备准备好通过 wifi 接收 adb 命令。在 WSL 方面，您将发出命令

> 亚行连接 <ip of="" phone="">:5555</ip>

如果你成功连接，你就可以开始了。您可以通过查看 VSCode 来验证这一点，注意它现在在右下角状态栏中显示一个设备。

# 用于仿真器的 ADB

因此，我将告诫这一点，即使我有它的工作，它是非常缓慢的建立在模拟器上，但想添加我的发现，以防别人已经使它运行得更好。

这个设置基本上使 Ubuntu adb 连接到运行在 windows 主机上的 adb 服务器。这工作，但正如我所说的是非常缓慢。

在 windows 主机上，您需要让 adb 监控传入连接的端口

> adb -a -P 5037 nodaemon 服务器

一旦从 Ubuntu 端完成了这些，你需要添加一个新的环境变量来设置 adb 连接到远程服务器而不是本地服务器。

> 导出 ADB _ SERVER _ SOCKET = TCP:192 . 168 . 1 . 97:5037
> ADB kill-SERVER

一旦完成，你应该能够运行 adb 设备，并看到你的设置的 windows 端托管的设备。

需要注意的一点是，您只能通过 CLI 运行 flutter 应用程序，而不能使用 VSCode 调试选项。和调试用的天文台 URL 有关系。

# 结论

WSL 2 中的开发设置非常好。它很快，很容易使用，但就像我说的 adb 设置使用模拟器非常慢。一个构建可能需要 30 分钟才能成功传输到模拟器。我的希望是，当其他人试图建立这种联系时，我们可以找到一种更好的方法来建立这种联系，使它成为一个更加无缝的过程。理想情况下，WSL 2 将能够直接连接到设备，但我相信这是一个长期的梦想。