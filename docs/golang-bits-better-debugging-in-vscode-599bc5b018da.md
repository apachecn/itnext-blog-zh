# golang bits:vs code 中更好的调试

> 原文：<https://itnext.io/golang-bits-better-debugging-in-vscode-599bc5b018da?source=collection_archive---------0----------------------->

VSCode 太棒了！但是对于 golang 来说，重命名和调试只是…嗯？，继续读下去！

![](img/f4e7d10dcca36a8f5dd860ab36de23a3.png)

# 调试器—最大字符串大小和检查嵌套限制:

VSCode 中的 golang 调试体验很差，因为当你到达第三级时，查看深度嵌套的变量被切断，查看字符串被限制为 64 个字符，这几乎永远不够，数组也是如此，然而，这让我不太困扰。

vscode 团队增加了对 delve v2 APIs 的支持，但在当前版本(v1.25.0)中，这些仍然是默认关闭的。要打开它，并控制这些讨厌的变量，请更改 vscode 用户设置以停止使用 delve v1 API，并控制 MaxStrLen 和 MaxVeriableRecurse 变量，如下所示:

文件->首选项->设置

```
"go.delveConfig": {
  "useApiV1": **false**,
  "dlvLoadConfig": {
    "followPointers": true,
    "maxVariableRecurse": **3**,
    "maxStringLen": **400**,
    "maxArrayValues": **400**,
    "maxStructFields": -1
  }
}
```

*只要记住“maxVariableRecurse”**会**让你的调试变慢，那就玩玩它，看看什么是你能接受的。

# 缓慢调试启动:

关闭 windows defender 防病毒实时保护:

开始菜单->搜索“病毒”，点击“病毒和威胁防护”->点击“病毒和威胁防护设置”->关闭“实时防护”

在我的 windows 10 机器上，当按下 F5 时，它需要很长时间才能启动，以至于我每次启动调试会话时都要等待大约 40 秒，关闭实时保护使它再次恢复正常。

# 慢速重命名:

vscode 使用 golang 自己的重命名工具来完成这项工作，但该工具会搜索整个 go 路径，其中可能有大量通常不相关的代码。这个 gorename 由 prateek 派生，禁用全局搜索并进行重命名

```
go get github.com/prateek/gorename
```

这将从源代码编译 gorename，现在您应该测试它是否正确安装在您的路径中

```
(on linux/mac:) which gorename
(on windows:) where gorename
```

还要检查 gorename 的创建日期，它应该是全新的。

# 在 golang 中创建 JSON 是一件痛苦的事情

在大多数情况下，Go 是一种非常容易编写的语言，但是与 typescript 相比，将 json 形式化为类型是非常乏味的,“粘贴到 JSON”vs code 插件使用一个名为 [quicktype](https://github.com/quicktype/quicktype) 的开源工具来为给定的 JSON 示例创建非常好的类型，它甚至可以找到相似的类型而不会创建重复的类型，如果你是严格的一方，你可能只需要在最后对几个内部类型进行表面上的重命名，但是现在重命名工作正常了，你可以这样做了..`

*   安装“将 JSON 粘贴为代码”
*   重新加载
*   创建新的。转到文件
*   复制你的 json
*   按:ctrl + shift + p
*   键入“将 json 粘贴为代码”，按回车键
*   出现提示时，设置根对象的名称
*   在需要的地方添加*，省略*