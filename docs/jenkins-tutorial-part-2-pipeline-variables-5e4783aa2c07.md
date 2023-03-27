# Jenkins 教程—第 2 部分—管道变量

> 原文：<https://itnext.io/jenkins-tutorial-part-2-pipeline-variables-5e4783aa2c07?source=collection_archive---------0----------------------->

![](img/c518273176f996c36d381a98cf53ea9a.png)

在本文中，我将介绍在管道中定义和使用变量的方法。如果你还没有阅读[之前的](/jenkins-tutorial-part-1-pipelines-bd1397cf5509)部分。变量是管道中最重要的部分，它使我们能够制作可重复的管道，并将代码从配置中分离出来。这就是为什么你应该学习如何定义和使用变量。那么，我们开始吧。

# 在继续阅读之前:

如果您需要一个完整的 Jenkins 堆栈，其中包含一组工具和插件，您可以部署我在我们公司、培训课程和本系列文章中使用的那个。这个堆栈可用于 Kubernetes 和 Docker。

[](https://github.com/ssbostan/jenkins-stack-kubernetes) [## GitHub-ssbo stan/jenkins-stack-kubernetes:在 Kubernetes 上部署 Jenkins 的脚本和清单

### 注意:此存储库是“DevOps with Saeid”类的一部分。部署 Jenkins 的脚本和清单…

github.com](https://github.com/ssbostan/jenkins-stack-kubernetes) [](https://github.com/ssbostan/jenkins-stack-docker) [## GitHub-ssbo stan/Jenkins-stack-Docker:Docker-compose 版本的 jenkins-stack-kubernetes

### Permalink 无法加载最新的提交信息。docker-撰写版本的詹金斯-栈-kubernetes。要查看…

github.com](https://github.com/ssbostan/jenkins-stack-docker) 

# 詹金斯管道环境变量:

管道环境变量可在`environment`部分定义。这个部分可以在整个管道上全局定义，也可以在特定的管道阶段定义。您可以同时在全局和每个阶段中定义环境变量。全局定义的变量可以在所有阶段中使用，但是阶段定义的变量只能在该阶段中使用。可以使用`NAME = VALUE`语法定义环境变量。要访问变量值，您可以使用这三种方法`$env.NAME`、`$NAME`或`${NAME}`，这三种方法没有区别。这是一个包含环境变量的完整管道示例。

# Jenkins 凭证和环境变量:

一个有用的助手方法可以用作环境变量的值来访问预定义的 Jenkins 凭证。所有众所周知的凭证——秘密文本、秘密文件、用户名/密码和带私钥的 SSH 以及由其他插件安装的其他凭证类型——都可以使用`credentials(id)` helper 进行访问。这里有一个例子。

**重要提示:**

当我们使用**“用户名/密码”**或**“带私钥的 SSH”**凭证之一时，除了我们定义的变量名之外，Jenkins 还自动定义了另外两个环境变量。这两个自动定义的变量被命名为`VARNAME_USR`和`VARNAME_PSW`，定义这些额外的变量是因为这些类型的凭证不是单值凭证。这些额外自动定义的变量的值是我们最初定义的变量所访问的整个凭证的一部分。

在**“用户名/密码”**凭证的情况下，假设你定义了一个名为`MYUSERPASS`的变量，分别是`MYUSERPASS`的值为`username:password`、`MYUSERPASS_USR`的值为`username`和`MYUSERPASS_PSW`的值为`password`。这里有一个例子。

在**“带私有密钥的 SSH”**凭证的情况下，假设您定义了一个环境变量`MYSSHINFO`,`MYSSHINFO`的值是 SSH 密钥文件的位置。如果该凭证有一个 SSH 用户名和密码，那么会自动定义`MYSSHINFO_USR`和`MYSSHINFO_PSW`，并将这些值分别分配给这些变量。

在哪里可以使用环境变量，哪些 Jenkins 方法/函数可以接受环境变量以及其他定义/访问变量的方法将在以后的文章中介绍。如果你感兴趣，请订阅我的个人资料。

关注我的领英【https://www.linkedin.com/in/ssbostan 