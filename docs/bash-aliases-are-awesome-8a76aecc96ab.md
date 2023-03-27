# Bash 别名太棒了

> 原文：<https://itnext.io/bash-aliases-are-awesome-8a76aecc96ab?source=collection_archive---------3----------------------->

太棒了。太棒了。太棒了。我无法表达我有多爱 bash 别名和我的 Bash shell。

![](img/0b5c11fadbae79f45661c349daebfeb0.png)

好吧，“爱”可能有点*强*，我承认。但还是；仅次于我的 IDE 和[时间管理习惯](https://medium.com/@dSebastien/steps-towards-a-better-work-life-equilibrium-5557af620e42)，Bash 别名是我最大的时间节省者。多亏了这些，我每天都有大量的空闲时间。

在本文中，我不会解释什么是 Bash，为什么它如此流行或者如何编写 Bash 脚本。今天，我想解释什么是 bash 别名，以及为什么应该使用它们来提高生产率。这是面向高级用户的，但任何人都可以使用，不仅仅是软件开发人员或系统管理员。

注:对于那些喜欢[鱼壳](https://en.wikipedia.org/wiki/Friendly_interactive_shell)的人来说:向你致敬，你是最好的。我之前的一个同事差点就把我转化成功了，但是我还是没有花时间去“迁移”和习惯。虽然，我确实把它看作是我亲爱的巴什的潜在替代品…-)

让我们来看看为什么 Bash 别名为 ROCK。

# 精简命令与详细命令

我通常“讨厌”人们共享命令，比如:

```
tar -czvf compressed_files.tar.gz files/
```

这个真的很受欢迎，但是你真的明白为什么是“czvf”而不是别的吗？我当然不知道。

上面这个命令的问题是，只有在你知道具体的工具并且一直在使用它的情况下，它才能对你有意义。事情是这样的，几年后，一切开始变得模糊不清。我知道如此多的编辑器、工具和命令，以至于我把一切都搞混了。

为了帮助记忆，我更喜欢写扩展版，这(通常)更容易阅读和理解:

```
tar --create --gzip --verbose --file compressed_files.tar.gz files/
```

你怎么想呢?有了上面的内容，即使您不知道 tar 实用程序的具体细节，您也至少能够猜到它将创建一个包含“files”文件夹内部内容的 gzipped 文件。我个人更喜欢这个命令的详细版本。

当然，缺点(也是命令参数的短版本首先存在的原因)是键入起来很无聊/很慢。如果你每次都要输入整行，你会很痛苦。

一旦您习惯了，这个命令的精简版本就更加实用了。但是如果你停止使用它一段时间，或者只是偶尔需要，那么记住它是一种真正的痛苦(即嘿谷歌)。

但是如果我们可以简单地使用这个:

```
archive compressed_files.tar.gz files/
```

这个虚构的命令在我的系统中并不存在，但是我仍然能够执行上面的命令并获得我的文件的压缩版本。

那么这个存档命令是什么呢？一个 Bash 别名！

该别名的定义如下:

```
alias archive='tar --create --gzip --verbose --file'
```

很简单，对吧？

定义了这个 Bash 别名后，键入“archive”就等同于编写整个 tar 命令。另一方面，如果我需要细节，那么我可以简单地检查我的别名并看到完整的命令。

Bash 别名就是这么简单，它们允许执行字符串扩展(以及更多)。它们很便宜。你可以创造数以千计的东西而不会减慢速度。

我还喜欢创建命令名的简写版本。例如，“kubectl”是我经常使用的一个命令；有了下面的别名，使用它就不那么痛苦了:

```
alias k='kubectl'
```

许多其他实用程序也是如此:

```
alias d='docker'
```

这可能看起来很愚蠢，但是如果你一直使用一个命令，这样的别名是最好的；-)

我也倾向于为我一直使用的命令创建简短的别名。例如:

```
alias krm='kubectl delete -f'
```

上面的命令允许我使用 kubectl 轻松地从我的 Kubernetes 集群中删除一些东西。

以下是其他一些例子:

```
alias gitlog="git log --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit" # show git log
```

这给了我一个很好的 git 日志。

```
alias amend='git commit --amend --no-edit'
```

这允许我快速修改 git 提交。键入“修改”，输入，嘣，完成。

顺便说一下，由于别名是 Bash 脚本的一部分，您可以添加注释、链接等来帮助您记住为什么东西会在那里。干净利落。

# 别名的别名

如果你像我一样，对命令的记忆很差，怎么办？如果您忘记了“存档”别名的存在，该怎么办？嗯，您也可以给一个别名加别名，并基于前一个别名创建一个“压缩”别名:

```
alias compress='archive'
```

多有趣啊。现在，你不必太在意这些命令，你可以变得更“马虎”一些。对我来说，重要的是实现目标，而不是拘泥于必须使用的精确命令。全世界的系统管理员都会恨我写了这个，但是我一点也不关心命令(真的)。我想实现我的目标；其他的一点都不重要。如果我需要同一事物的两个别名，那就这样吧；-)

这不是万灵药，也有古怪，但这是一个有用的技巧。

# 为命令设置默认值

我们可以用 Bash 做的另一件很酷的事情是创建现有命令的别名，只是为了替换它们的缺省值或根据您的喜好改进/调整它们的行为。

例如，当我使用“ls”命令时，我更喜欢颜色和人类可读的输出。我没有把命令参数记在脑子里，每次都把它们全部输入，而是创建了一个别名，如下所示:

```
alias ls='ls --color=auto --human-readable -al' # colored and human readable sizes
```

多亏了这一点，每当我使用 ls 时，我都能得到我想要的输出。

# 功能

您还可以使用 Bash 函数创建 Bash“别名”。这对于更高级的场景很有用。

这里有一个例子:

```
uuid() {
 local N B C='89ab'for (( N=0; N < 16; ++N ))
 do
  B=$(( $RANDOM%256 ))case $N in
   6)
    printf '4%x' $(( B%16 ))
    ;;
   8)
    printf '%c%x' ${C:$RANDOM%${#C}:1} $(( B%16 ))
    ;;
   3 | 5 | 7 | 9)
    printf '%02x-' $B
    ;;
   *)
    printf '%02x' $B
    ;;
  esac
 doneecho
}
```

顾名思义，上面的函数生成了一个 [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) 。这很酷，因为我不需要任何工具。当然，我不会太相信这些标识符的全局唯一性，但是出于测试的目的，它完成了工作，这就是我所需要的。

这里还有一个:

```
mkcd()
{
 mkdir -p $1 && cd $1
}
```

这个挺酷的；它会创建一个新的目录并直接将 cd 放入其中。在这个别名中，我使用了“$1”，它对应于提供给函数的第一个参数。所以它会把我在“cd”后面输入的任何单词作为文件夹名。

您还可以使用$*来获取所有参数:

```
calc(){ awk "BEGIN{ print $* }" ;}
```

在上面的例子中，我们手边有一个简单的计算器..；-)

# 在文件系统中更快地导航

您是否已经在文件系统树中向上移动了 5 个文件夹？你当时感觉如何？对我来说，那样做总是超级讨厌。

为了减轻痛苦，我使用…你猜对了，Bash 别名！:)

```
..() {
  N=$(($1))
  if [ $N -lt 1 ]; then
 N=1
  fi
  while ((N)); do
 cd ..
 let N-=1
  done;
}
```

有了上面的别名，我可以简单地输入“..5 英寸，再升 5 级。

但是你可以从中获得更多的乐趣:

```
alias .1='cd ..' # see .. function
alias .2='.. 2'
alias .3='.. 3'
alias .4='.. 4'
alias .5='.. 5'
alias .6='.. 6'
alias .7='.. 7'
alias .8='.. 8'
alias ..1='.1'
alias ..2='.2'
alias ..3='.3'
alias ..4='.4'
alias ..5='.5'
alias ..6='.6'
alias ..7='.7'
alias ..8='.8'
alias ...='.. 2'
alias ....='.. 3'
alias .....='.. 4'
alias ......='.. 5'
alias .......='.. 6'
alias .......='.. 7'
alias ........='.. 8'
alias cd..='cd ..'
alias cd...='.. 2'
alias cd....='.. 3'
alias cd.....='.. 4'
alias cd......='.. 5'
alias cd.......='.. 6'
alias cd........='.. 7'
alias cd.........='.. 8'
```

有了这些别名，你可以变得更懒，输入“. 5”，”..5”或“cd……”。不管怎样，这只是为了好玩。

另一种快速到达目的地的补充方法是用特定的路径定义别名。我这样做是为了我的开发工作空间和写作项目。

例如:

```
export DEV_HOME=/home/sebastien/wks
alias wks='cd $DEV_WKS_HOME'
alias w='wks'
```

现在，只需输入“w ”,我就能到达我想去的地方。干净简单。

# 运行程序非常快

几年前，我们可以使用像 [Launchy](https://www.launchy.net/) 、 [ULauncher](https://ulauncher.io/) 或 [Albert](https://albertlauncher.github.io/) 这样的工具来快速启动程序。幸运的是，这些年来，操作系统已经有所改进，并且已经集成了快速启动程序的内置方法。

尽管如此，Bash 别名对于这个用例仍然很有用。有时，您想要运行的程序不在操作系统菜单中，或者需要特定的参数。当然可以加，但是很扫兴。

就我个人而言，事情很简单，我的 Bash shell 就是我的*家*，所以我能从那里做的越多越好。使用 Bash 运行程序非常容易。首先，路径上的任何可执行文件当然都是可以直接访问的。其次，很容易创建别名来启动带有特定参数的程序，并在需要时将它们转换成后台进程。

这里有一个例子:

```
alias visualparadigm='(~/Visual_Paradigm_CE_15.2/Visual_Paradigm) &'
alias vp='visualparadigm'
```

上面的别名允许我在后台启动 Visual Paradigm(即保持我的 shell 可用),只需输入“vp”。

我对 IntelliJ 也有同样的看法:

```
alias intellij='/opt/idea-IU/bin/idea.sh > /dev/null 2>&1 &'
alias ij='intellij'
```

在这种情况下，我将程序的所有输出重定向到/dev/null，以避免它用无意义的内容污染我的 shell。

当然，也可以通过创建更高级别的别名来同时启动一组工具:

```
alias tools='(intellij) && (visualparadigm)'
```

有了上面的代码，我可以简单地输入“tools ”,点击回车，打开我的 IDE 和建模工具。当然，你可以配置/启动任何你想要的东西。

关于程序的最后一点是，你确实可以通过使用函数和函数参数来传递参数，就像我在上一节解释的那样。这里有一个很酷的例子:

```
enfr(){ (google-chrome "[https://translate.google.com/?sl=en&tl=fr&text=$*](https://translate.google.com/?sl=en&tl=fr&text=$*)" )& }
```

有了这个别名，我可以快速查找从英语到法语的翻译。例如，输入“enfr 华丽”，谷歌会告诉我它对应于“华丽”。整洁！当然，您可以对其进行改进，以便在 bash 终端上获得正确的翻译，但是我没有做到这一点..还没有。

了解了所有这些，您可能会意识到您可以创建复杂的场景，如:

*   转到您的工作文件夹
*   打开你的工具
*   在某个文件中写入时间戳
*   发送邮件
*   …前途无量；-)

在我的例子中，每当我打开我的 shell 时，我配置我的工作环境(例如，NodeJS 版本、npm 版本、Java 版本等)，转到我的工作文件夹，列出内容，等等。

# 只针对 Linux？没有。

如果您是 Windows 或 Mac 用户，不要错误地认为您不能使用 Bash 和 Bash 别名/函数。你可以。

在 [OSX 卡特琳娜上，苹果已经用](https://www.theverge.com/2019/6/4/18651872/apple-macos-catalina-zsh-bash-shell-replacement-features) [Z shell (zsh)](https://en.wikipedia.org/wiki/Z_shell) 代替了 bash 作为默认 shell，但据我所知，Zsh [支持和 bash 相同的别名语法](https://unix.stackexchange.com/questions/25243/difference-between-alias-in-zsh-and-alias-in-bash)。

在 Windows 上，您还有多种选择。第一个是 Git，它与 Git Bash 一起出现，这是 Bourne shell 的一个端口，它的存在要归功于 [MinGW](https://en.wikipedia.org/wiki/MinGW) 和它的 MSYS 组件。另一个选择是使用[Windows Subsystem for Linux(WSL)](https://docs.microsoft.com/en-us/windows/wsl/faq)，它为您提供了一个全功能的 Linux 发行版，在 Windows 上可以访问您的 Windows 文件系统(甚至可以直接从 WSL 执行 Windows 二进制文件)。

我个人在操作系统之间共享我的大部分别名，没有大的障碍…

# 但是..我该怎么办？

如果有需求，我会写一篇关于如何/在哪里定义 Bash 别名的文章。无论如何，如果你有动力，已经有很多关于这个的文章了。

# 有什么值得混的？

一旦开始，为了帮助您确定需要哪些别名，您可以利用 Bash 历史文件。该文件包含您以前在终端中使用过的命令；这就是允许您使用“向上/向下”箭头来重用命令的原因。

有几个环境变量可用于控制历史文件:

```
PROMPT_COMMAND='history -a' # append current session history to the content of the history fileexport HISTFILE=$HOME/.bash_history
export HISTCONTROL=ignoreboth # ignore duplicates and spaces in .bash_history
export HISTSIZE=20000 # Big history
export HISTFILESIZE=20000 # Big history
export SAVEHIST=20000 # Big history
shopt -s histappend # append to the history file, don't overwrite it
```

可以解析/分析历史文件来找到您最常用的命令。这很有用，因为这些命令可能值得别名化；-)

```
alias typeless='history | tail -n 20000 | sed "s/.* //" | sort | uniq -c | sort -g | tail -n 100' # top 100 commands
alias tl='typeless'
```

上面的别名对我帮助很大，帮助我弄清楚我需要自动化什么来赢得时间。

# 结论

在本文中，我试图让您了解什么是 Bash 别名/函数，以及为什么它们如此令人敬畏。

Bash 别名的出色之处在于它们定义简单、易于维护、易于理解(您可以编写完整的命令)并且便宜。

我给了你一些你可以做的很酷的事情的例子，但是还有很多。在以后的文章中，我会分享我最喜欢的别名。

既然您已经了解了 Bash 别名，那么就继续使用它们吧。如果你有，那就分享你正在做的很酷的事情吧！；-)

今天到此为止！

# 喜欢这篇文章吗？点击下面“喜欢”按钮查看更多内容，并确保其他人也能看到！

PS:如果你想学习大量关于软件开发、Web 开发、TypeScript、Angular、React、Vue、Kotlin、Java、Docker/Kubernetes 和其他酷主题的其他酷东西，那么不要犹豫去[拿一本我的书](https://www.amazon.com/Learn-TypeScript-Building-Applications-understanding-ebook/dp/B081FB89BL)并订阅[我的简讯](https://mailchi.mp/fb661753d54a/developassion-newsletter)！