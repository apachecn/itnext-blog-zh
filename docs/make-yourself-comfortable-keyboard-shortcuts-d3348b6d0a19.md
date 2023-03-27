# 让自己舒服:键盘快捷键

> 原文：<https://itnext.io/make-yourself-comfortable-keyboard-shortcuts-d3348b6d0a19?source=collection_archive---------1----------------------->

# 介绍

![](img/64f014e62fd8d91828f1650d17b90472.png)

照片由 [Stefen Tan](https://unsplash.com/@stefentan?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

你得到了一个全新安装了你最喜欢的发行版的系统。那你会怎么做？毫无疑问，让自己舒服。如果你不从捷径开始，我打赌这是你优先考虑的事情之一。谁不喜欢在菜单的缎带和汉堡中寻找你需要的那个小按钮，猜猜因为你点击了边框之外，整个菜单都折叠起来了。这时你会意识到笨拙的 UX 正在夺走你的生产力。在没有 UI 的情况下，很明显你需要为你日常工作中使用的程序和工具设置一个快捷方式。

资料来源:giphy.com

因此，设置快捷方式可以大大节省时间和精力，提高您的工作效率，而不是苦苦寻找正确的菜单来点击。

幸运的是，我们的背包里有很棒的工具，比如`xvkbd`**`xdotool``xev``xbindkeys`和 Windows `[AutoHotkey](https://www.autohotkey.com/)`来帮助设置我们喜欢的快捷方式。**

# **如何安装**

**首先，使用您最喜欢的软件包管理器(如 apt)安装软件包。**

```
$ sudo apt-get install xvkbd xdotool xbindkeys
```

**然后生成你家的默认设置**

```
$ xbindkeys --defaults-guile > ~/.xbindkeysrc
```

**最后使用重载`xbindkeys`**

```
$ killall -s1 xbindkeys
$ xbindkeys -f ~/.xbindkeysrc
```

**在 Windows 上，使用`winget`或`scoop`或您喜欢的软件包管理器安装`AutoHotkey`。**

```
$ wiget install AutoHotkey
```

```
$ scoop install autohotkey
```

**要在 Windows 上启动时运行快捷方式，请在您的首选位置创建一个`.ahk`文件。通过`AutoHotkey`将文件设置为始终打开。然后，右键单击该文件并在桌面上创建一个快捷方式。最后，将文件移动到启动路径`%AppData%\\Microsoft\\Windows\\Start Menu\\Programs\\Startup`**

# **获取您输入的密钥**

**为了有效地创建快捷方式，知道您输入的键码是很有帮助的。这在映射功能键或鼠标键时尤其重要。**

**`xbindkeys -k`识别您输入的输入键。它还支持多键输入**

```
$ xbindkeys -k
Press combination of keys or/and click under the window.
You can use one of the two lines after "NoCommand"
in $HOME/.xbindkeysrc to bind a key.
"(Scheme function)"
    m:0xc + c:46
    Control+Alt + l
```

**对于`AHK`，打开快捷脚本后，双击开始托盘中的程序图标。当窗口打开时，按下`Control + K`打开*按键历史*窗口，您可以看到打开窗口后所按按键的历史。使用`F5`刷新来更新列表。**

# **扩展文本**

**你能得到的一个最简单的好处就是把一个单词扩展成一个更长的句子，比如`@@`，，到你的邮箱。**

```
(xbindkey-function '(shift 2)
                   (let ((count 0))
                     (lambda ()
                       (set! count (+ count 1))
                       (if (> count 1)
                           (begin
                            (set! count 0)
                            (run-command "xvkbd -no-jump-pointer -xsendevent -text 'email@domain.com'"))))))
```

**使用`AHK`你可以实现同样的扩张**

```
::@@::email@domain.com
```

# **获取窗口类名**

**使用`xprop`获取一个窗口的类名。运行程序后，它等待你点击你想要的窗口。**

```
$ xprop | grep WM_CLASS
WM_CLASS(STRING) = "JetBrains Toolbox", "jetbrains-toolbox"
```

**在 Windows 上，`AHK`提供“Windows Spy”来获取关于 Windows 的信息。只需右击开始托盘中的`AHK`，选择“Windows Spy”然后专注于你打算获取信息的窗口。**

# **启动程序**

**要启动您最喜欢的应用程序，请使用应用程序名称和它需要的参数。**

```
(xbindkey '(control 1) "nautils")
(xbindkey '(control 2) "xterm")
(xbindkey '(control 3) "xterm")
```

**同样，使用`AHK`**

```
^1::Run, code
^2::Run, winword.exe
```

# **再变换**

# **模拟按键**

**无论您是否过于频繁地使用长组合键，或者您的键盘布局不适合某个特定的组合，将您经常使用的组合重新映射到一个更方便的组合都会节省您的大量时间。**

**例如，在`xbindkeys`中将`escape`映射到`control-q`**

```
(xbindkey '(control q) "xvkbd -no-jump-pointer -xsendevent -text '\\e'")
```

**在 Windows 上也是如此**

```
^q::Send {Esc}
```

# **将按键发送到特定程序**

**如果目标窗口在后台，应该指定目标窗口。想象一下停止屏幕录制或下载过程。**

**通过为`xvkbd`指定`-window`参数，只有指定的程序接收输入。**

**为了修改之前的快捷方式，只将`escape`发送到`vi`窗口，我将`-window "*vi*"`添加到`xvkbd`。**

```
(xbindkey '(control q) "xvkbd -window '*vi*' -no-jump-pointer -xsendevent -text '\\e'")
```

**查看`AHK`文档，我们看到相关函数`ControlSend`有如下定义**

```
ControlSend [, Control, Keys, WinTitle, WinText, ExcludeTitle, ExcludeText]
```

**它有两种行为。它要么寻找由类名和实例号(这里是`NN`)组成的`ClassNN`，要么作为文本模式进行文本匹配。**

**如果您喜欢在脚本请求时在`Persistent`块中使用文本匹配，通过设置`SetTitleMatchMode`定义您想要的匹配算法。**

```
#Persistent 
SetTitleMatchMode, 2
return
```

**模式 1 必须以指定的文本开始匹配。只满足包含子串是模式 2。为了精确匹配，请使用模式 3。从版本 1.0.45 开始，通过将`SetTitleMatchMode`设置为`RegEx`可以使用正则表达式匹配。**

```
^q::ControlSend, Windows.UI.Input.InputSite.WindowClass1, {Esc}, vi,
```

# **重新映射鼠标键**

**重新映射鼠标键与重新映射键盘快捷键没有什么不同。使用`AHK`“窗口间谍”识别 Windows 上的按键，使用`xeu`识别`xbindkeys`。**

```
~MButton::Send, ^!{tab}
```

**运行`xeu`并按下目标按钮，`xeu`将显示按钮键**

```
$ xeu
ButtonRelease event, serial 37, synthetic NO, window 0x4000001,
    root 0x1fe, subw 0x0, time 6743289, (131,61), root:(231,225),
    state 0x210, button 2, same_screen YES
```

**然后在这里使用这个关键代码作为`b:MOUSE-BUTTON-CODE`，例如在您的`xbindkeys`脚本中使用 2。**

```
(xbindkey '(control "b:2") "xvkbd -no-jump-pointer -xsendevent -text '\\M'")
```

# **媒体键**

**如果您为了娱乐或教育而听音乐或看视频，媒体键对您来说很方便。许多键盘没有媒体键，或者媒体键的布局非常不一致。这使得为媒体键设置一些快捷键变得更加必要。**

**从 VLC 和 RhythmBox 到 Spotify，是你控制各种媒体播放器的好朋友。**

**对于安装，请使用 apt**

```
$ sudo apt install playerctl
```

**`playerctl`允许你确定你的目标玩家。假设你想暂停 VLC，但不想暂停 Spotify。**

```
# play only vlc
$ playerctl --player=vlc play
# next track on Spotify but not VLC
$ playerctl --player=spotify --ignore-player=vlc next
# seek all players forward 3s
$ playerctl --all-players 3+
# seek all players backward 3s
$ playerctl --all-players 3-
```

**现在是时候使用`xbindkeys`将它们绑定到快捷键了**

```
(xbindkey '(control /) "playerctl --all-players play-pause")
(xbindkey '(control .) "playerctl --all-players position 3+")
(xbindkey '(control ,) "playerctl --all-players position 3-")
```

**在 Windows 上实现相同的功能是通过模拟媒体键来完成的。同样，您可以像以前一样将这个模拟按键发送到您想要的窗口。**

```
^!/::Send       {Media_Play_Pause}
^!,::Send       {Media_Prev}
^!.::Send       {Media_Next}
```

# **重新映射 Microsoft Surface Pen**

**你对你的 Surface pen 的默认功能不满意吗？这部分是给你的。细长笔 2 发出`F18`、`F19`、`F20`用于长按、双按和单按。**

**首先，在 Windows 设置中，将笔压动作设置为无。然后，让我们使用`AHK`将它们映射到例如媒体键。**

```
<#F20::
Send       {Media_Play_Pause}
return

<#F19::
Send       {Media_Prev}
return

<#F18::
send, {Media_Next}
return
```

# **最后的想法**

**![](img/a8fcac3089d62d880432ceb76e055353.png)**

**资料来源:giphy.com**

****总之，键盘快捷键不仅有用，而且很有趣**！通过使用组合键而不是鼠标，您可以节省时间，感觉像一个超级英雄，并为您的日常计算机程序添加一点奇思妙想。所以下次你使用电脑的时候，试着使用一些键盘快捷键，看看你的体验会变得多么愉快和高效。如果你需要一些想法，看看这里的。如果你想做得更多， [*输入映射器*](https://github.com/sezanzeb/input-remapper) 应该会让你感兴趣。**

**我会多写 CS 的文章；因此，如果你觉得有趣和有帮助，请跟随我的媒介。另外，请随时通过 [LinkedIn](https://www.linkedin.com/in/ali--moezzi/) 直接联系我。**