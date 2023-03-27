# SwiftUI 键盘快捷键和 macOS 菜单栏。

> 原文：<https://itnext.io/swiftui-keyboard-shortcuts-and-menu-bar-be22abbb3791?source=collection_archive---------1----------------------->

![](img/c5d338cebf0b18bebfcee4a0522f928d.png)

使用 SwiftUI 2，向操作添加键盘快捷键和向应用程序的菜单栏添加按钮非常容易。

# 键盘快捷键

您可以简单地通过将`.keyboardShortcut(_:)`添加到按钮来为动作添加键盘快捷键:

您几乎可以将每个角色设置为`KeyEquivalent`。但是请注意，你不能通过简单地按“B”来切换按钮。当使用像“v”这样的字母时，你需要按“⌘⇧V”，当使用像“6”这样的数字时，你需要按“⌘6".”你也可以指定你自己的修饰键，把它们传递给`modifier`参数。

您可以定义一个，甚至多个。

苹果开发者网站上有所有事件修改器的列表。

在访问`KeyEquivalent`属性时，您不仅可以将字母和数字设置为快捷方式，还可以设置其他按钮，如 space 或 page up。

苹果开发者网站上有一份所有等同物的列表。您也可以通过按下 escape 按钮或 enter 按钮来切换按钮:

或者

# 。命令()

使用 SwiftUI 2，您还可以向应用程序的菜单栏添加按钮。您可以在工具栏中添加新菜单，也可以在现有菜单中添加按钮。在 app 协议中，命令被直接传递到场景。

有两种选择，要么将命令直接传递给`.commands()`修饰符，要么将它们放在一个单独的结构中。

然后你用`.commands { AppCommands() }`把命令传到现场。

# 命令菜单

命令菜单在菜单栏中创建新菜单。

这将创建一个名为“menu”的菜单，它有两个按钮，用于执行一个操作。您还可以向按钮添加键盘快捷键，该快捷键显示在标签旁边。`Divider()`可用于将动作分组在一起。当然你也可以在命令菜单中添加一个`Picker()`。但是您需要将`@State`作为`@Binding`传递给`AppCommands`结构。当你想添加子菜单时，添加一个`Menu()`。

# 命令组

可以使用命令组向现有菜单添加按钮。您可以将它们添加到现有按钮之前或之后，也可以替换现有按钮。

用您想要的按钮替换`CommandGroupPlacement`。一些例子是`.appInfo`、`.pasteboard`或`.undoRedo`。完整的列表在[苹果开发者网站](https://developer.apple.com/documentation/swiftui/commandgroupplacement)上。

# 更大的

还有一组预定义的命令。