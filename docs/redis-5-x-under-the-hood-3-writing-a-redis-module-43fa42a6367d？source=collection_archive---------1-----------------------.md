# Redis 5。幕后 x:3—编写 Redis 模块

> 原文：<https://itnext.io/redis-5-x-under-the-hood-3-writing-a-redis-module-43fa42a6367d?source=collection_archive---------1----------------------->

![](img/6a4b2816307975dd5ef0f065b98d9b3f.png)

**奥克兰海湾大桥** —这张照片是 2019 年 3 月 30 日在旧金山海湾拍摄的——当时我在写这篇文章，为 RedisConf 19 做准备

尽管 Redis 支持许多数据类型和丰富的特性，但有时您可能会被要求在数据上添加核心 Redis 不支持的定制逻辑。当这种情况发生时，您有两种选择来实现相同的需求，要么通过 Lua 脚本，要么通过 Redis 模块。Lua 脚本是一种“组合”现有数据结构和命令的方法，但不是一种将系统功能扩展到它没有设计覆盖的用例的方法。与 Lua 脚本相比，Redis 模块具有以下优势:

*   使用 Redis 模块可以创建新的数据结构，而使用 Lua 脚本只能使用现有的数据结构。
*   作为 C 库，Redis 模块运行速度更快，开销更少，同时还允许您调用外部第三方库。
*   通过 Redis 模块添加的命令可以通过 Redis 客户端直接调用，就好像它们是 Redis 的本地命令一样，而对于 Lua，脚本必须通过 EVAL/EVALSHA 调用。
*   暴露给 Redis 模块的 API 比暴露给 Lua 脚本的 API 丰富得多。

## Redis 模块为什么成功？

早在 2016 年，Salvatore Sanfilippo 在他的博客上写了关于模块作为系统功能[的文章:](http://antirez.com/news/106)

> 模块可能是系统中最有趣的特性，同时也是最有问题的特性:版本之间的 API 不兼容、低质量的模块使系统崩溃、缺乏可扩展系统的身份都是可能的问题。

Redis 模块 API 是为未来的兼容性而设计的，所以今天编写的模块可以在 4 年后使用相同的 API 工作，而不管 Redis 核心的变化。从这些基础出发，为了访问 Redis 数据空间，诞生了两种不同的 API:

*   一个是低级 API，提供非常快速的访问和一组操作 Redis 数据结构的函数。Redis 开发人员可以创建与 Redis 本地命令一样强大和快速的命令。
*   另一个 API 更高级，允许调用 Redis 命令并获取结果，类似于 Lua 脚本访问 Redis 的方式。

Redis 模块使 Redis 用户走得更快，而不会危及 Redis 核心及其开发，因为这两个基础 API 被承诺在未来几年不会有突破性的变化。API 将得到改进，但具有完全的向后兼容性，因为模块注册需要给定的 API 版本。

在本文中，我们将进一步检查底层 API，但是首先，我们将介绍如何加载现有的 Redis 模块。

我们将展示如何用一个操作列表编写一个简单的 Redis 模块。FILTER，这将使我们能够过滤列表中的低值和高值(实现通常所说的低值和高值过滤器)。

# 0.1 在我们深入探讨之前:

我们决定记录这个相互学习的旅程，以更好地理解核心 Redis 5 及其顶级扩展如何工作，讨论这个新版本中的新特性，因此阅读这一系列文章的人可以就它适合和不适合的不同用例做出明智的决定。

我们会认为你对 Redis 5 有一个整体的了解。x 和一个 Redis 5。x 服务器实例运行，正如我们在本系列的前几篇文章中所描述的:

— [Redis 5。幕后 x:1—Redis-服务器启动并运行](https://medium.com/@fcosta_oliveira/redis-5-x-under-the-hood-1-downloading-and-installing-redis-locally-3373fe67a154)

——[Redis 5。引擎盖下的 x:2—Redis 命令和数据结构介绍—第 1 部分](https://medium.com/@fcosta_oliveira/redis-5-x-under-the-hood-2-intro-to-redis-commands-and-data-structures-part-1-41f05501cb52)

— [直截了当的时间序列 DBs 与 Redis 5。X —第 1 部分::RedisTimeSeries 模块](https://medium.com/@fcosta_oliveira/straightforward-time-series-dbs-with-redis-5-x-part-1-redistimeseries-module-79aa616159eb)

# 1 —加载和构建模块

有许多 Redis 模块可以用来扩展 Redis 的功能。在撰写本文时，这些是 Github stars 的顶级模块:

*   [neural-redis](https://github.com/antirez/neural-redis) :作为 redis 数据的在线可训练神经网络
*   【Redis 搜索【Redis 上的全文搜索
*   [Redis JSON](https://github.com/RedisLabsModules/redisjson):Redis 的 JSON 数据类型
*   [rediSQL](https://github.com/RedBeardLab/rediSQL) :一个 Redis 模块，提供了一个快速的、内存中的 SQL 引擎(嵌入了 SQLite)
*   [redis-cell](https://github.com/brandur/redis-cell):Redis 模块，在 Redis 中以单个命令的形式提供速率限制
*   [RedisGraph](https://github.com/RedisLabsModules/RedisGraph) :使用稀疏邻接矩阵的基于密码的查询语言的图形数据库
*   [RedisML](https://github.com/RedisLabsModules/redisml) :机器学习模型服务器
*   [Redis Time series](https://github.com/RedisLabsModules/RedisTimeSeries):Redis 的时序数据结构。如果您还记得，我们曾经写过一篇文章，提供了深入的使用示例—[Redis 5 中简单的时间序列数据库。X — part 1 :: RedisTimeSeries 模块](https://medium.com/@fcosta_oliveira/straightforward-time-series-dbs-with-redis-5-x-part-1-redistimeseries-module-79aa616159eb)。

## 1.1—构建 RedisJSON 模块:

通过在您的终端上执行以下命令，从 GitHub 下载 RedisJSON 的源代码:

```
git clone --depth 1 [https://github.com/RedisLabsModules/redisjson.git](https://github.com/RedisLabsModules/redisjson.git) && cd redisjson
cd src
make
```

模块二进制文件将被生成为 **rejson.so**

## 1.2 —加载 RedisJSON 模块:

使用以下`redis.conf`配置指令加载模块:

```
loadmodule **/path/to/****rejson****.so**
```

最后:

```
redis-server **/path/to****/redis.conf &** redis-cli 
127.0.0.1:6379>
```

还可以在运行时使用 MODULE LOAD 命令加载一个模块，该命令带有我们要加载的 Redis 模块的路径:

```
MODULE LOAD **/path/to/****rejson****.so**
```

如果模块是通过模块加载命令加载的，那么当服务器停止时，它将被自动卸载。

要列出所有加载的模块，请使用:

```
127.0.0.1:6379> module list
1) 1) "name"
   2) "ReJSON"
   3) "ver"
   4) (integer) 10004
```

最后，您可以使用以下命令卸载(如果愿意，以后可以重新加载)模块:

```
MODULE UNLOAD ***name of the module***
```

你应该知道，如果我们加载的模块导出了一个或多个模块端的数据类型，就像**redis son**所做的那样，我们就不能在运行时卸载它。我们应该从`redis.conf`文件中删除配置指令，然后重启 redis-server。如果您尝试在运行时卸载 ReJSON，您应该会看到以下错误消息:

```
127.0.0.1:6379> **MODULE UNLOAD ReJSON**(error) ERR Error unloading module: the module exports one or more module-side data types, can't unload
```

# 2—编写我们的 Redis 模块

编写 Redis 模块最简单直接的方法是从 Redis 的官方[库](https://github.com/antirez/redis/blob/5.0/src/redismodule.h)下载一份`redismodule.h`头文件，将它包含在我们的源代码树中，编写模块，链接我们想要的所有库，并创建一个导出了`RedisModule_OnLoad()` 函数符号的动态库。

您也可以用 C++或任何具有 C 绑定功能的语言编写模块。

我们要用 C 语言实现一个 Redis 模块 LIST_EXTEND，带有一个新的命令过滤器，可以调用如下:

```
127.0.0.1:6379> **LIST_EXTEND.FILTER** source_list destination_list low_value high_value
```

该命令获取一个 Redis 列表 **source_list** 并尝试创建另一个 Redis 列表 **destination_list** ，其元素如果是数字，将被条件过滤:

*   低 _ 值≥元素值≤高 _ 值

如果 low_value 大于 high_value，则返回一个空列表。

low_value 和 high_value 可以是-inf 和+inf，这样您就可以只实现低过滤器、高过滤器或两者都实现。默认情况下，由 low_value 和 high_value 指定的间隔是封闭的(包含)。

**返回值** 整数回复:过滤操作后 destination_list 列表的长度。

例如，如果执行了以下命令:

```
127.0.0.1:6379> **LPUSH** source_list 20 2 30 1 40
(integer) 4
127.0.0.1:6379> **LIST_EXTEND.FILTER** source_list destination_list 2 20
(integer) 3
127.0.0.1:6379> **LRANGE** destination_list 0 -1
1) "1"
2) "2"
3) "20"
```

destination_list 将具有[1，2，20],因为 source_list 的其他值没有验证低值和高值过滤条件。

## 2 .1—编写我们的 LIST_EXTEND 模块

## 项目结构

让我们首先创建一个名为`list_extend.c`的文件，并包含名为`redismodule.h`的 C 头文件。

不要担心复制代码的每个要点。完整的模块项目文件夹可以在 GitHub 的以下[链接](https://github.com/filipecosta90/list_extend)中找到。

## 模块入口点

每个 Redis 模块的入口点是一个名为`RedisModule_OnLoad()`的函数，使它能够被初始化，注册它的命令，以及它使用的其他私有数据结构。对于模块来说，调用命令的一个好习惯是在模块名后面跟一个点，最后是命令名。在我们的例子中，我们的命令将以下面的方式调用`LIST_EXTEND.FILTER`。这样就不太可能发生碰撞。下面是我们的`RedisModule_OnLoad()`实现:

在`RedisModule_OnLoad()`内部，第一个要调用的函数应该是`RedisModule_Init()`。以下是函数原型:

```
int RedisModule_Init(RedisModuleCtx *ctx, const char *modulename, int module_version, int api_version);
```

`Init`函数通知 Redis 核心模块有一个给定的名称、它的版本(由`MODULE LIST`报告)以及愿意使用的 API 版本。

第二个函数名为`RedisModule_CreateCommand()`，用于将命令注册到 Redis 核心中，具有以下函数原型:

```
int RedisModule_CreateCommand(RedisModuleCtx *ctx, const char *name, RedisModuleCmdFunc cmdfunc, const char *strflags, int firstkey, int lastkey, int keystep);
```

标志集“strflags”指定了命令的行为，应该作为由空格分隔的单词组成的 C 字符串传递。因为我们的命令修改数据，所以我们添加了“写入”。我们的命令也将使用额外的内存，并且应该在内存不足的情况下被拒绝，因此我们添加了标志“deny-oom”。在我们的例子中，“strflags”将是“write deny-oom”。

除非有错误，否则 RedisModule_OnLoad()的返回值应该是 REDISMODULE_OK。

## 创建命令处理程序

要创建一个新命令，上述函数需要上下文、命令名和实现该命令的函数的函数指针，它必须具有以下原型:

```
int mycommand(RedisModuleCtx *ctx, RedisModuleString **argv, int argc);
```

在我们的例子中，实现该命令的函数将是`ListExtendFilter_RedisCommand()`，如下面的代码片段所示:

它的参数只是上下文，将传递给所有其他 API 调用、命令参数向量和用户传递的参数总数。参数作为指向特定数据类型`RedisModuleString`的指针(**argv)提供。这是一个不透明的数据类型，你有 API 函数来访问和使用，直接访问它的字段是不需要的。

## 2.1.3.1 描述命令逻辑

在函数内部，我们首先检查命令参数的数量是否正确。由于该命令需要四个参数(source_list、destination_list、low_value、high_value ),包括命令本身，因此命令参数的数量应为 5。

然后，我们要求 Redis 自动管理命令处理程序中的资源和内存，只需调用**Redis module _ auto memory**(CTX)。

接下来，我们通过使用低级 API **RedisModule_OpenKey** ()，开始访问 Redis 键 source_list，一个 Redis 列表。这个 API 返回一个指向 RedisModuleKey 的指针，它是 Redis 键的处理程序。

然后，我们通过使用 **RedisModule_KeyType** ()来检查键类型，以确保输入是列表或空的。现在我们知道我们有一个列表或空列表，我们使用 **RedisModule_ValueLength** ()检查元素的数量。如果键指针为空，由于类型为空，**RedisMoudule _ value length()**返回零，这正是我们的函数逻辑所需要的。如果有零个元素，我们通过**redis module _ ReplyWithLongLong()**向客户端发送一个 0 的整数回复来结束函数调用。

接下来，我们检查是否正确设置了限制，可能的值是参数 lower_limit ( argv[3])的一个**整数** **或** **-inf** ，以及参数 lower_limit ( argv[4])的一个**整数或+inf** 。由于我们使用了 long long limit 常量(LONG_MIN 和 LONG_MAX ),我们还需要包含< limits.h >头文件。

最后一步包括循环遍历**源列表**，并将每个元素的值分配给**目的列表**，如果它们都是整数并且在定义的限制内。我们通过 **RedisModule_ListPop()** 从列表尾部弹出 source_list 中的每个元素，通过**redismodulestringlonglong()**将返回的 RedisModuleString 转换为长整型，检查限制条件，然后如果满足条件，将长整型转换回 RedisModuleString，并通过 **RedisModule_ListPush** ()，将其推送到 **destination_list** 的头部。

我们调用**Redis module _ replicate verbal**(CTX)将命令传播到所有 Redis 副本。

最后，我们通过**redis module _ replywithlong()**向客户端发送一个整数回复，其中包含添加到 **destination_list** 中的商品数量。**redis module _ ReplyWithLongLong()**总是返回`REDISMODULE_OK`。

`list_extend.c`的最终版本可以在 GitHub 的以下[链接](https://github.com/filipecosta90/list_extend/blob/master/list_extend.c)中获得。

## 2.1.3.1 汇编列表 _ENTEXD

现在我们已经完全设置好了我们的模块，我们只需要编译它并将它加载到 Redis 中(如 1.2 节所示)。

```
gcc -fPIC -shared -std=gnu99 -o list_extend.o list_extend.c
```

下面是我们工作命令的一个例子:

```
127.0.0.1:6379> MODULE LOAD /Users/filipeoliveira/redis-porto/list-extend/list_extend.o
OK127.0.0.1:6379> lpush origin 1 20 25 40 50 string1 1.1 4
(integer) 8127.0.0.1:6379> list_extend.filter origin dest -inf 10
(integer) 2127.0.0.1:6379> lrange dest 0 -1
1) "4"
2) "1"
```

## 3 —后续步骤:

这个基本设置让您对 Redis 模块的功能有了一点了解。我们鼓励您尝试构建自己的模块。如果您喜欢我们的低值和高值过滤器命令，作为一个建议，尝试对排序的集合实现它。

在以下链接中查看 Redis 模块 API 参考和官方介绍:

*   [https://redis.io/topics/modules-api-ref](https://redis.io/topics/modules-api-ref)
*   [https://redis.io/topics/modules-intro](https://redis.io/topics/modules-intro)