# 我希望 python 打包从 JVM Jars 中学到的东西

> 原文：<https://itnext.io/things-i-wish-python-packaging-learned-from-the-jvm-jars-aed19133c121?source=collection_archive---------1----------------------->

我们在 python 中的依赖性管理方面取得了很大进步，但是仍然缺少一个关键部分。

我已经分享了 python 依赖管理工具的经验，但是你必须承认 python 依赖管理已经走了很长的路。像[诗歌](https://python-poetry.org/)这样的工具，基于一系列的 pep，给了我们 NPM 的依赖锁定能力和管理一个真正简单的包构建流程的能力。不再有神秘的“显现”和设置。py 黑魔法，而是干净，清晰和简单的“诗歌构建”,你就有了你的轮子。

![](img/33073d23e85f03ffacfad65e255cbdf6.png)

[约公元前 3150 年的卢布尔雅那沼泽车轮](https://en.wikipedia.org/wiki/Ljubljana_Marshes_Wheel)(世界上最古老的精确放射性碳年代测定的木制车轮部件的复原模型)。

还有[轮子](https://realpython.com/python-wheels/)都很棒！但是..安装时，下载所有指定的依赖项，这也是目前大多数用例的逻辑解决方案，除非..你的库包依赖于一个私有的回购包？还是打算安装在没有互联网的“安全”环境中？那我们就有麻烦了。

我个人是最近才撞见这个的。我的大多数依赖项安装在 docker 构建中，在托管 CI 环境中，我可以提供私有的 repo 凭证，而不会在实际的工件上公开它们。但我需要在我的 Databricks installed wheel 中添加一个带有 spark 工作的私有包，突然我遇到了麻烦。

有一些变通办法，您可以在安装 wheel 之前专门安装您的私有依赖项，您可以在生产环境中提供凭证。或者设置一个镜像/缓存来用您的包替换 pypi。其他选项包括类似于[旅行车](https://github.com/cloudify-cosmo/wagon)或 [Shiv](https://pypi.org/project/shiv/) 的项目，或者手动捆绑依赖关系的所有车轮

**进入优步罐**

JVM 生态系统很久以前就遇到了这个问题，并为此开发了工具。汇编插件存在于所有主要的构建工具(gradle、maven、sbt for scala 等)中，允许你创建包含所有依赖项的“优步罐”,并将它们一起交付给生产。它们还允许您控制不包括什么，例如，如果您在每个集群中预装了 spark，您就不需要它成为 UberJar 的一部分，如果需要，还可以使用工具来解决冲突。

**后记**

我很乐意接受纠正，并找到一种简单的方法将我的依赖项打包到一个锁死环境的轮子中。但是如果没有，我真的希望 python 依赖管理工具能给我们提供一个解决方案。我敢肯定有人已经在努力了。

特别感谢 [Shai Berge](https://twitter.com/shaib_il) r、 [Tal Einat](https://twitter.com/taleinat) 、 [Meir Kriheli](https://twitter.com/mkriheli) 、 [Avraham Serour](https://github.com/tovmeod) 以及整个 [Pyweb-IL group](https://groups.google.com/g/pyweb-ilhttps://groups.google.com/g/pyweb-il) 的建议和投入，感谢他们这么多年来为我们提供了一个很好的学习场所和社区。