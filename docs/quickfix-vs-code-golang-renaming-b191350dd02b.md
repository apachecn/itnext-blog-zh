# 快速修复:Vs-code Golang 重命名

> 原文：<https://itnext.io/quickfix-vs-code-golang-renaming-b191350dd02b?source=collection_archive---------2----------------------->

![](img/6f1cebb319ebc9a356c8f0542de1a86e.png)

vs-code 中的 Golang 重命名不可用，这里有一个修复方法！

重命名在任何编码语言中都是一个基本工具，而 vs-code 是一个非常流行的 Go IDE，所以令人惊讶的是这个问题从未得到解决。在网上查的时候发现 golang 的 vs-code 插件只是运行 golang 自己的重命名工具。Golang 相当快，而且大多数 golang 工具都构造得当，为什么会这么慢呢？

问题是这个工具经常认为它需要在* **整个 GOPATH*** 中搜索任何可能需要重命名的依赖项**，这绝不是任何按 F2 键的人所期望的**。搜索 git 问题，我发现有一个修复，@github.com/parteek 已经注释掉了全局搜索代码，但在编译他的 gorename 时，我仍然遇到了同样的问题，并且没有编译的二进制文件。当我构建它的时候，它不知何故仍然使用 golang.org/x/tools/refactor/rename 包…

所以我想让它更简单，我将代码重组到一个包中，并测试它的工作情况。与 go 1.11 代码的比较表明，“工具/重构/重命名”代码从那以后就没变过。

# 安装:

```
go install github.com/amitbet/gorename
```

现在您应该在＄GOPATH/bin 目录中有了一个新的 gorename.exe。

现在，在 vs-code 中重命名只需两秒钟(由于在运行 gorename 之前进行了代码检查和审查)

**原 hack:**[https://github . com/prateek/tools/commit/72 b 0 e 988 ce 6957 db 328582 decda 8829154 c 66545](https://github.com/prateek/tools/commit/72b0e988ce6957db328582decda8829154c66545)

# 有什么问题吗？

如果你仍然得到一个 100% CPU 的 gorename.exe 进程，你是从错误的目录中使用它，在任务管理器中检查进程的命令行，或者运行

窗口:

```
where gorename.exe
```

或者在 mac 上:

```
which gorename.exe
```

然后更改 PATH 变量，以便调用正确的 gorename.exe，或者用＄GOPATH/bin 目录中的新变量覆盖另一个变量。