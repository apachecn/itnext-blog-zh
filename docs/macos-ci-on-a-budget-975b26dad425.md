# 预算上的 macOS CI

> 原文：<https://itnext.io/macos-ci-on-a-budget-975b26dad425?source=collection_archive---------3----------------------->

![](img/30b4b3c01f1f7ddece7d95e85c50a561.png)

作为开发人员，我们都喜欢观察绿色指示灯的光芒，以及构建和测试我们的应用程序的迷人的技术舞蹈。可惜在 iOS 的世界里，现实看起来并不那么美好。CI 服务对于我们的宠物项目来说是[负担不起的](https://www.bitrise.io/pricing)，而租用一台专用的构建机器甚至[更加昂贵](https://www.macstadium.com/pricing)。问题当然不是这些服务贪婪，而是我们被捆绑的平台。一个 iOS 应用只能建立在 macOS 上，就是这样。

在这篇文章中，我想提供一个解决这个问题的方法，并展示如何使用你自己的苹果电脑，你已经使用它作为一个 CI 机器。我们将使用 [GitHub Actions](https://github.com/features/actions) 作为 CI 框架，使用 [VirtualBox](https://www.virtualbox.org) 在后台运行我们的 CI 机器。唯一的要求是，你的电脑上必须有一台 x86 架构的苹果电脑和至少 100GB 的磁盘空间。据我所知，VirtualBox [不支持](https://forums.virtualbox.org/viewtopic.php?t=98742)M1 架构。不过，您可以使用其他虚拟化解决方案，如 [Parallels](https://www.parallels.com) 或 [VMware Fusion](https://www.vmware.com/products/fusion.html) ，来实现相同的结果。

# 放弃

运行虚拟化的 macOS 是一个历史上敏感的话题。据我所知，在苹果的 EULA 下，我们在这里做的一切都是合法的。我们不是越狱或运行黑客程序。我们在正版 macOS 上运行虚拟化 macOS，这在 EULA 的以下条款中有所提及。

[引用](https://www.apple.com/legal/sla/docs/macOSBigSur.pdf) : *(iii)在您拥有或控制的已经运行苹果软件的每台苹果电脑的虚拟操作系统环境中安装、使用和运行最多两(2)份苹果软件的附加副本或实例，目的是:(a)软件开发；(b)软件开发期间的测试；使用 macOS 服务器；或(d)个人非商业用途。*

当然，用你最好的判断，因为我不是律师，不能给你这方面的建议。

# 什么是 GitHub 动作

正如 GitHub 自己所说，*“GitHub Actions 是 GitHub 上因果的 API:编排任何工作流，基于任何事件”*。我将使用这个特殊的平台，因为如果你提供你自己的[构建机器](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)，它将免费为你提供大量的特性。在我们的情况下，这真的很令人兴奋，因为通过提供我们机器的一部分，我们可以获得全功能的 macOS CI。

GitHub Actions 不仅仅是构建你的项目。您可以用它做任何事情，包括优秀的 Bash 脚本。该平台使用 YAML 文件来设置由步骤定义的构建过程。这些步骤可以是任何东西，从你自己的脚本到社区编写的[大型库](https://github.com/marketplace?category=&type=actions)和官方步骤。很可能，您可能需要的一切都已经为您编写好了，这使得您的 CI 设置非常通用。

GitHub 还提供了自己的构建服务器，甚至还有免费的每月分钟数供你试用。不幸的是，在 iOS 世界里，问题依然存在。在 GitHub 上，构建时间对我们来说非常昂贵，与 Linux 分钟相比，它是以 10 的倍数计算的。

你可能会问，我们如何将我们的机器用作构建服务器。GitHub 让它变得非常简单。在你的机器上启动 GitHub Action Runner([我有另一篇关于如何在 Synology NAS](https://oleksandrkirichenko.com/blog/github-runner-on-synology/) 上运行它的文章)，并将其链接到你的帐户，完成。一旦您的机器被链接，您在 YAML 文件中指定您想要使用一个自托管的构建机器，那么一切都工作得很好。

正如我前面提到的，我们将在虚拟机上运行 GitHub Actions Runner。有几个原因可以解释你为什么想这么做。首先也是最重要的，是安全问题。因为您将运行社区提供的第三方代码，所以您不希望这些代码在您的主机上运行。其次，方便。我们不希望这个进程在我们的机器上不受控制地运行，产生进程并安装各种软件。通过将它转移到虚拟机，我们隔离了该环境，因此我们不需要太担心那里会发生什么。当然，虚拟机将拥有可管理的分配内存量，在后台整洁地运行。

听起来不错，对吧？现在让我们试着把这个付诸实践。

# 把所有的放在一起

安装过程可能相当繁琐，所以一定要按照每一步操作，不要跳过任何“不必要”的东西。在安装过程中，我们将创建一个可引导的 ISO 文件，用于在虚拟机上安装 macOS。接下来，我们将准备一个足以构建 iOS 项目的构建环境。一切准备就绪后，我们就可以将它链接到我们的 GitHub 帐户了。作为最后一步，我们将设置最棘手的部分，当我们的机器启动时，让一切在后台自动启动。

# 创建可引导的 ISO 文件

macOS 发行版[拥有创建可引导 ISO 映像所需的一切](https://support.apple.com/en-us/HT201372)，因此我们在这里不会使用任何不必要的第三方工具。你需要的只是官方的 macOS 发行版和终端。

1.  确保您有大约 30GB 的可用磁盘空间来执行此过程。
2.  从 [Mac 应用商店](https://apps.apple.com/de/app/macos-big-sur/id1526878132)下载 macOS。是的，如果你是新手，即使你已经在使用这个 macOS 版本，你也可以从 Mac App Store 下载 macOS。我在我的主机上使用的是 Big Sur 11.4，我打算在虚拟机上安装相同版本的 macOS。如果你用相同版本的 macOS 跟我学就更好了。
3.  创建一个临时磁盘映像，我们将使用它来创建 macOS 安装 ISO `hdiutil create -o /tmp/BigSur -size 16G -layout SPUD -fs HFS+J -type SPARSE`
4.  安装它`hdiutil attach /tmp/BigSur.sparseimage -noverify -mountpoint /Volumes/mac_install`
5.  使用 macOS 发行版中的`createinstallmedia`实用程序，使挂载的映像成为 macOS 安装映像`sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/mac_install`
6.  卸下它`hdiutil detach /Volumes/Install\ macOS\ Big\ Sur`。如果它总是占线，您可能需要`-force`标志。
7.  将稀疏图像转换成 ISO 图像`hdiutil convert /tmp/BigSur.sparseimage -format UDTO -o /tmp/BigSur.iso`
8.  将你的 ISO 文件移动到桌面，将扩展名从`cdr`改为`iso` `mv /tmp/BigSur.iso.cdr ~/Desktop/BigSur.iso`
9.  删除临时图像`rm /tmp/BigSur.sparseimage`

就是这样。现在您有了一个可引导的 ISO 文件来安装 macOS，我们可以进入下一步了。

# 创建虚拟机

我在撰写本文时的 VirtualBox 版本是 6.1.22，如果您想在相同的环境中遵循这些步骤，这可能很重要。

1.  安装 VirtualBox + VirtualBox 扩展包。可以从[官网](https://www.virtualbox.org/wiki/Downloads)获取，按照说明安装。在安装过程中，您需要在系统偏好设置- >安全&隐私中允许 Oracle 扩展。不需要额外的设置。
2.  如果看不到内存设置，请按 CMD+N 创建一个新的虚拟机，并将此屏幕切换到专家模式。
3.  在向导的第一个屏幕上，我使用了以下设置。名称:`BigSurCI`，类型:`Mac OS X`，版本:`Mac OS X (64-bit)`，内存大小:`4096 MB`，硬盘:`Create a virtual hard disk now`。
4.  接下来，在第二个屏幕上。文件大小:`160 GB`(这个很多，可以减小大小，但我会建议至少 100GB，Xcode 和 CI 会占很大空间)，类型:`VDI`，物理硬盘上的存储:`Dynamically allocated`(如果不在乎磁盘空间，固定大小大概会好一点)。
5.  点击创建按钮。现在让我们在安装 macOS 之前对这台机器做一些小的调整。
6.  按 CMD+S 打开虚拟机设置并更改以下参数。
7.  系统->处理器->处理器:`2`。
8.  显示->屏幕->显存:`128`。
9.  音频->启用音频:`off`。
10.  网络->适配器 1 ->高级->端口转发。添加以下规则:名称:`SSH Rule`，协议:`TCP`，主机 IP: `127.0.0.1`，主机端口:`2222`，访客 IP: `leave it blank`，访客端口:`22`。有了这些设置，您应该能够通过连接到`127.0.0.1:2222`经由 SSH 连接到您的虚拟机。稍后，您需要启用远程登录功能才能使其工作。
11.  Storage -> Optical Drive:通过单击磁盘图标选择您在上一步中创建的 BigSur ISO 文件。

# 安装 macOS

1.  启动虚拟机，并在对话框中选择您的 ISO 来启动机器。在这个阶段，您将被要求允许输入监控。
2.  等待几分钟让机器加载，然后选择您喜欢的语言。
3.  打开`Disk Utility`。
4.  选择`VBOX HARDDISK`介质并按下擦除按钮，设置如下。名称:`Macintosh HD`，格式:`Mac OS Extended (Journaled)`，方案:`GUID`。
5.  关闭`Disk Utility`，选择`Install macOS Big Sur`。
6.  仔细阅读许可协议，选择新格式化的驱动器。点击“继续”并等待。
7.  现在系统已经安装好了，您必须完成不断增长的 macOS onboarding 过程。性能很可能绝对无法使用。经历一下这个。这很好，因为它不会影响 CI 性能。入职完成后，系统启动并显示 dock 可能需要几分钟时间。耐心点。
8.  右击桌面->改变桌面背景->屏保:禁用屏保。
9.  系统首选项->能源服务器:禁用所有睡眠设置，并将睡眠定时器设置为`Never`。如果您仍然遇到计算机进入睡眠的问题，您可以随时使用[这个应用程序](https://apps.apple.com/de/app/amphetamine/id937984704?l=en&mt=12)来解决这个问题。
10.  系统系统首选项->用户和组->登录选项为您的用户启用`Automatic Login`。
11.  系统偏好设置->共享:启用`Remote Login`。

既然我们已经经历了这一切，我们终于可以开始收获我们所播种的了。让我们设置我们需要的一切，以便我们的 CI 可以建立一个应用程序，然后当我们看到它可以工作时，我们可以做最后的抛光。

1.  打开[苹果开发者门户](https://developer.apple.com/download/)，在该系统上下载并安装当前 Xcode 版本。记住安装后至少运行一次。
2.  最后，我们可以离开我们虚拟机的 UI，切换到终端。使用`ssh username@127.0.0.1 -p 2222`登录您的机器。当然，使用您的用户名和密码。如果您愿意，可以继续在虚拟机的终端中执行所有操作。
3.  Do xcode-select `sudo xcode-select -s /Applications/Xcode.app/Contents/Developer`。
4.  安装[自制](https://brew.sh)。这可能需要一些时间，因为 Xcode 的`Command Line Tools`需要和它一起安装。安装时可能会卡住。重新启动系统并再次执行此步骤可能会有所帮助。如果你在完成`Command Line Tools`安装步骤时一直有问题，你可以事先用一个单独的`xcode-select --install`命令来完成。
5.  安装[椰子](https://cocoapods.org)
6.  安装[浪子](https://fastlane.tools)

既然我们现在有了一个完整的构建环境，这应该足以构建几乎任何原生 iOS 项目。是时候安装 GitHub Actions Runner 了，它将在这台机器上构建您的项目。

我们将[将 GitHub Actions Runner](https://docs.github.com/en/actions/hosting-your-own-runners/adding-self-hosted-runners) 添加到一个存储库中，以简化这个过程。当然，您可以将其添加到您的组织中，以便在您的所有项目中共享。要将 GitHub Actions Runner 添加到您的存储库中，请在 GitHub 上打开您的存储库，进入设置- >操作- > Runners，单击添加 Runner。您将看到如何下载和激活您的跑步者的说明。按照这些说明，您将拥有一个完全配置好并准备运行的 CI。你现在可以试一试。

# 最后的步骤

虽然您的 CI 现在已经启动并运行，但还远远不够完美。你仍然有 VirtualBox 和一个虚拟机在 dock 中运行。更有甚者，如果你重启电脑，所有的魔力都会瞬间消失。让我们来解决这个问题，使这个设置尽可能无缝。为此，我们将在主机和虚拟机上都使用`launchd`。在我们的主机上，我们需要让虚拟机在后台自动启动。另一方面，虚拟机应该在启动时自动启动 GitHub Actions Runner。`launchd`文件的语法相当复杂。如果你想做一些定制，我可以推荐一个[极好的资源](https://www.launchd.info)，里面有所有这些设置的描述。

在使用`sudo`权限的虚拟机上，我们创建了一个`launchd`文件，当系统启动时，它将启动 GitHub Actions Runner。此外，我们将设置`KeepAlive`来保持其运行，并在出现故障时重启。创建文件`sudo vim /Library/LaunchAgents/org.my.actions.runner.plist`并将以下内容复制到其中。不要忘记将文件路径更新到虚拟机上正确的`run.sh`位置。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Disabled</key>  <false/>
    <key>RunAtLoad</key> <true/>
    <key>KeepAlive</key> <true/>
    <key>Label</key>     <string>org.my.actions.runner</string>
    <key>EnvironmentVariables</key>
      <dict>
        <key>PATH</key>
	<string>/bin:/usr/bin:/usr/local/bin</string>
        <key>RUNNER_ALLOW_RUNASROOT</key>
        <string>1</string>
        <key>LC_ALL</key>
        <string>en_US.UTF-8</string>
        <key>LANG</key>
        <string>en_US.UTF-8</string>
      </dict>
    <key>ProgramArguments</key>
        <array>
            <string>/Users/test/actions-runner/run.sh</string>
        </array>
</dict>
</plist>
```

重启你的虚拟机`sudo shutdown -r now`，确保 GitHub Actions Runner 已经启动。重启后，您可以通过重新登录到`ssh`并执行类似于`ps -ax | grep runner`的操作来验证它已经启动。或者，你可以在你添加它的页面上看到它在 GitHub 上运行。它应该出现在跑步者列表中，旁边有一个绿色指示器。如果它不能正常工作，我建议在我们刚刚创建的`plist`中设置日志路径，并在前面提到的网站上寻找更多的故障排除建议。

此时，您的虚拟机是完全自动化的。我们的下一个也是最后一个任务是让这台机器在你的计算机启动时在后台运行。我们将对`launchd`使用与 GitHub Actions Runner 相同的方法。

在主机上使用`sudo`权限创建一个包含以下内容的`launchd` `sudo vim /Library/LaunchAgents/org.github.actions.vm.launch.plist`文件。同样，记得检查 VirtualBox 的路径和虚拟机的名称。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Disabled</key>  <false/>
    <key>RunAtLoad</key> <true/>
    <key>KeepAlive</key> <true/>
    <key>Label</key>     <string>org.github.actions.vm.launch</string>

    <key>ProgramArguments</key>
        <array>
            <string>/Applications/VirtualBox.app/Contents/MacOS/VBoxManage</string>
            <string>startvm</string>
            <string>BigSurCI</string>
            <string>--type</string>
            <string>headless</string>
        </array>
</dict>
</plist>
```

重新启动计算机，打开 VirtualBox 应用程序，检查虚拟机是否在后台运行。此外，检查您的跑步者是否仍在 GitHub 上运行。

我想指出的是，除了切换`plist`中的`Disabled`标志并重启电脑之外，我没有找到在后台快速停止这台机器的方法。如果你只是在 VirtualBox 中关闭它，它会立即重新启动。如果你知道解决这个问题的更好的方法，我将非常感谢你的建议。

# 结论

如果您预算紧张，这可能是您获得真正的 macOS CI 的一个好选择。我尝试了一个月的设置，从我个人的经验来看，我遇到的唯一问题是在 DaVinci Resolve 中编辑视频，这种问题一直持续到我关闭虚拟机。你也必须牺牲少量的内存，但你可能不会注意到它。另一个值得一提的负面影响是，如果您在构建期间将机器置于睡眠模式，构建将会失败。这些是我个人注意到的唯一的问题，坦率地说，我预计会有更多的问题。反正选择权在你。我个人倾向于裸机设置，并希望 [type 1 虚拟机监控程序](https://en.wikipedia.org/wiki/Hypervisor#Classification)很快可以在苹果 M1 处理器上使用。保持健康，祝你实验好运！

# 提到的软件版本

电脑:`iMac 2019 3,2 GHz 6-Core Intel Core i7 16 GB`
MacOS: `BigSur 11.4`
虚拟盒子:`6.1.22`
GirHub Actions Runner:`actions-runner-osx-x64-2.278.0.tar.gz`
CocoaPods:`1.10.1`
fast lane:`2.185.1`
Xcode:`12.5`

*考虑开发一个 Flutter 或者 iOS 的 app，或者找咨询或者人力？从初创企业到公司层面，我都可以雇佣。只需* [*联系我*](https://oleksandrkirichenko.com/contact/) *看看我们能一起做些什么！*

*喜欢这篇文章？别忘了砸这个拍手*👏按钮让我感觉很棒，有动力写更多！

*原载于*[*https://oleksandrkirichenko.com*](https://oleksandrkirichenko.com/blog/macos-ci-on-a-budget/)*。*