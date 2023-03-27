# 调试 Python 中的代码— pdb 与 rpdb

> 原文：<https://itnext.io/debugging-your-code-in-python-pdb-vs-rpdb-e7bb918a8ac3?source=collection_archive---------2----------------------->

调试是解决编程问题的重要部分。尽管这是一个有争议的话题，但有很多工具比打印报表好得多。

![](img/40b6cd9d0012621ba84211a3cd1ed407.png)

由[约尔·德苏尔蒙特](https://unsplash.com/@yoal_des?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

Python 附带了一个内置的交互式调试器，称为 **pdb** 。

Pdb 允许您在源代码行级别设置断点和单步执行，检查堆栈帧，列出源代码，并在任何堆栈帧的上下文中评估任意 Python 代码。事后调试也可以在程序控制下执行。

然后是 **rpdb** ，一个基于 pdb 的远程调试器。它将 *stdin* 和 *stdout* 重新路由到一个套接字处理程序，以便您可以调试服务器进程。

# 我什么时候在 rpdb 上使用 pdb？

Pdb 应该在本地应用程序中使用，而 rpdb 远程运行，可以用来调试 Flask 这样的 web 框架。安装 rpdb 后，您可以远程启动服务器并运行服务器代码。

下面我将向您展示在一些简单的应用程序中运行的 pdb 和 rpdb，一个作为脚本运行，另一个启动 Flask 服务器。确保在 Python Repo 中克隆[调试，以运行本教程中的示例。](https://github.com/diazjf/learn-pydebug)

```
$ git clone [https://github.com/diazjf/learn-pydebug](https://github.com/diazjf/learn-pydebug)Cloning into 'learn-pydebug'...
remote: Enumerating objects: 18, done.
remote: Counting objects: 100% (18/18), done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 18 (delta 0), reused 15 (delta 0), pack-reused 0
Receiving objects: 100% (18/18), 103.60 KiB | 573.00 KiB/s, done.$ cd learn-pydebug
```

**注意:**Python Repo 中的[调试有关于安装和运行应用程序的信息，包括并展示 pdb 和 rpdb。代码中有关于问题所在以及调试如何帮助的注释。](https://github.com/diazjf/learn-pydebug)

# 使用 pdb

我们将下面的代码添加到应用程序中希望启动 pdb 的位置。[***pdb/main . py***](https://github.com/diazjf/learn-pydebug/blob/main/pdb/main.py#L20)*中的代码包含下面一行:*

```
***import** **pdb**; pdb.set_trace()*
```

*运行 python 应用程序时，当那个***pdb . set _ trace()***被命中时，pdb 会自动启动。*

```
*$ python pdb/main.py
Enter some words: I am a cat> /Users/fernandodiaz/Desktop/debugging/pdb/main.py(23)<module>()
-> num_chars = count(input)
(Pdb)*
```

*您将看到 pdb 已启动。Pdb 包含以下[基本功能](https://docs.python.org/3/library/pdb.html)，可以在提示符下运行。以下是一些命令运行后的 pdb 输出示例。*

*   ***l(ist):列出当前行周围的行***

```
*(Pdb) l18       input = input("Enter some words: ")
19
20       import pdb; pdb.set_trace()
21
22       # count the number if characters
**23  ->     num_chars = count(input)**
24       print("Number of Characters: %s" % num_chars)*
```

*   ***w(这里):显示我们当前所在的文件和行号***

```
*(Pdb) w> /Users/fernandodiaz/Desktop/learn-pydebug/pdb/main.py(23)<module>()
**-> num_chars = count(input)***
```

*   ***s(步进):进入当前行的功能***

```
*(Pdb) s--Call--
> /Users/fernandodiaz/Desktop/learn-pydebug/pdb/main.py(3)count()
**-> def count(message=""):***
```

*   ***n(ext):执行当前行***

```
*(Pdb) n> /Users/fernandodiaz/Desktop/learn-pydebug/pdb/main.py(4)count() **-> msg = message.split()***
```

*   ***a(rgs):打印当前函数的参数列表***

```
*(Pdb) a**message = 'I am a cat'***
```

*   ***p(rint) <名称>:打印<名称>** 的值*

```
*(Pdb) n> /Users/fernandodiaz/Desktop/learn-pydebug/pdb/main.py(9)count()
**-> if len(msg) > 3:**(Pdb) p msg**['I', 'am', 'a', 'cat']***
```

***注意:**我在 pdb 测试应用程序上按顺序运行了所有这些命令。*

*通过运行这些命令，我们可以继续进入 ***count*** 函数中的下一行，直到我们发现错误。在这个例子中，我们可以看到*

```
*if len(msg) > 3: return “TOO MANY WORDS, TRY AGAIN”*
```

*这表明，我们要么需要向消息中添加更多的细节，让用户知道什么是可接受的，要么我们可以将字数调整为字符数。无论我们如何解决它，我们现在可以知道问题在哪里。*

*![](img/6cba5821e643135a16950f2e7876c243.png)*

*照片由[西格蒙德](https://unsplash.com/@sigmund?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄*

*虽然在这里可能看起来不那么令人印象深刻，但 pdb 在有很多事情要做的大型代码库中确实很有用。*

# *使用 rpdb*

*我们将以下内容添加到应用程序中要启动 rpdb 的位置:*

```
***import rpdb;** rpdb.set_trace()*
```

***注意:**要识别该命令，必须安装 rpdb。它安装在[先决条件](https://github.com/diazjf/learn-pydebug#prerequisites)中。*

*如果你看一下[***rpdp/remote/routes . py***](https://github.com/diazjf/learn-pydebug/blob/main/rpdb/remote/routes.py#L13)，可以看到调试器被添加到了 ***/count*** 路由中。我们可以通过安装[先决条件](https://github.com/diazjf/learn-pydebug#prerequisites)来运行 Flask web 服务器，然后运行:*

```
*$ python rpdb/run.py* Serving Flask app 'remote' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on [**http://192.168.1.147:5000/**](http://192.168.1.147:5000/) (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 926-264-267*
```

*在单独的终端中，我们可以向运行应用程序的端点发送请求。*

```
*$ curl -X POST[**http://192.168.1.147:5000/count**](http://192.168.1.147:5000/count) -d '{"message": "This is a message"}'* 
```

*这里我们发送一条消息，这条消息最初导致服务器抛出一个 400。在调试器存在的情况下，这个请求导致应用程序挂起，并允许我们检查调试器，看看发生了什么。为此，我们可以在单独的终端中运行以下命令:*

```
*$ nc 127.0.0.1 4444> /Users/fernandodiaz/Desktop/debugging/rpdb/remote/routes.py(15)add_note()
-> if not msg:
(Pdb)*
```

*现在，我们可以像使用 pdb 一样运行命令，以确定为什么我们的消息会出现 400。*

*本教程提供了基础知识。还有其他使用这些调试器的方法，以及更多的命令，可以在 [pdb](https://docs.python.org/3/library/pdb.html) 和 [rpdb](https://pypi.org/project/rpdb/) 文档中看到。*

*![](img/5f66ea0881ed209fe4530afa85e89f20.png)*

*照片由[詹姆斯·哈里森](https://unsplash.com/@jstrippa?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄*

*感谢阅读，我希望你喜欢！要了解更多内容，并跟上我写的内容，请在 twitter 上关注我🐦*