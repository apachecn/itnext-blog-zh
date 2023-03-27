# 用纱线修补 NPM 附属品

> 原文：<https://itnext.io/patch-an-npm-dependency-with-yarn-ddde2e194576?source=collection_archive---------1----------------------->

![](img/1686d623a2dcb20251e0e92d95efa0f4.png)

当您在第三方依赖关系中遇到错误时，通常您只有有限的选项来修复它。

您可以直接修改依赖项的本地代码并进行新的构建。如果这是一个严重的 bug，并且你必须尽快修复它，那么这个方法是有效的。但是，这不是最好的选择，因为您将在下一次`yarn`或`npm`安装时丢失修复。此外，您将无法与您的团队共享修复。

另一种选择是派生包，修复 bug，并创建一个 pull 请求。此时，您可以更新您的项目依赖项并使用您的 fork，直到包的维护者批准它并发布新版本。

```
"dependencies": {
  "buggy-package": "your_user/buggy-package#bugfix"
}
```

这看起来是一个不错的选择，现在您的团队成员将在更新项目依赖关系时获得修复。不利的一面是，您必须维护 fork 并确保它是最新的。

我们有更好的选择吗？是的，我们有。我们可以使用 yarn v2.0.0 中引入的`yarn patch`命令。这允许您即时对您的依赖项进行修复，而无需派生包。

让我们以`remix-run`框架为例。在开发过程中，我发现了会话存储的一个问题。我打开了一个 [pull 请求](https://github.com/remix-run/remix/pull/3113)来修复它，然后我使用命令直接在项目中修补这个包:

```
yarn patch @remix-run/node@npm:1.5.1 ➤ YN0000: Package @remix-run/node@npm:1.5.1 got extracted with success! 
➤ YN0000: You can now edit the following folder: /private/var/folders/xm/qntd4h_97zn6w88tc95bsvxc0000gp/T/xfs-bfc9a229/user 
➤ YN0000: Once you are done run yarn patch-commit -s /private/var/folders/xm/qntd4h_97zn6w88tc95bsvxc0000gp/T/xfs-bfc9a229/user and Yarn will store a patchfile based on your changes. ➤ YN0000: Done in 0s 68ms
```

提取包后，我们可以打开创建的文件夹，并在那里进行更改。一个缺点是，你必须修改产品代码，这可能是缩小和难以调试。在我的例子中，是对`fileStorage`模块的简单改变。

下一步是使用之前在控制台中显示的命令提交补丁

```
yarn patch-commit -s /private/var/folders/xm/qntd4h_97zn6w88tc95bsvxc0000gp/T/xfs-bfc9a229/user
```

此时，我们应该在`package.json`中看到以下变化

```
"resolutions" : { 
   "@remix-run/node@1.5.1" : "patch:@remix-      run/node@npm:1.5.1#.yarn/patches/@remix-run-node-npm-1.5.1-51061cf212.patch" 
}
```

在`.yarn/patches`中，你会找到补丁文件。

使用这种方法的好处是您的团队可以审查补丁，与 fork 相比，它不需要应用额外的工作。这应该用于关键的修复，如果你需要一个新的特性，我建议你用分叉来代替。另外，不要忘记打开一个问题，并在实际的包中创建一个拉式请求。