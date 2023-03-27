# Node.js 配置和机密管理

> 原文：<https://itnext.io/node-js-configuration-and-secrets-management-acd84375ca7?source=collection_archive---------0----------------------->

![](img/36ccfd422ce17964fa372b4ee2c31fcd.png)

照片由[米勒·佩龙](https://unsplash.com/photos/xrVDYZRGdw4?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/coding?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fnode-js-configuration-and-secrets-management-acd84375ca7%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

在过去的几年里，我参与了许多 Node.js 项目，其中一个反复出现的问题是人们管理配置和秘密的方式。

首先，这些是我试图通过配置/机密管理实现的主要方面。

## 灵活性

允许软件开发人员更改配置。大多数情况下，管理配置是运营部门的职责，但这会增加不必要的开销，并减慢调试/修复过程。很多时候是开发人员定义配置选项，因此他们最了解正确的配置值。考虑以下情况:

Scala (Akka)内置的应用程序使用`typesafe-config`进行服务器配置，并使用它指定线程池/调度程序选项。在某一点上，应用程序已经开始面临严重的性能问题。由于运营部门对应用程序的内部架构没有足够的了解(他们不了解线程池，更不知道哪些参与者/未来在哪些池中运行)，修复流程如下所示:

1.  软件工程师将在本地重现该问题
2.  软件工程师会找到问题的原因，经过一番试验后，会为线程池提出正确的设置
3.  软件工程师将联系运营部门，以相应地更改生产配置

在这个过程中，步骤 3 是不必要的。此外，如果建立了适当的 CI/CD 渠道，并且实施了适当的开发实践，步骤 3 将不会通过同行评审或者以任何方式被跟踪

更普遍的情况是，开发人员需要提高特定服务的调试级别。为什么不给开发者一个自己动手的选项呢？

鉴于存在同行评审，这可能会像任何其他代码更改一样被破坏

## 安全性

秘密值(数据库密码、API 访问密钥等)不应出现在回购协议中。句号。

这一点与“灵活性”有点矛盾，但对于大多数配置库来说，这很容易解决。怎么会？在 repo 中有所有非秘密值，然后在部署过程中注入秘密值(我更喜欢使用 ENV 变量)

## 冗长

避免为每个环境重新定义整个配置非常有用。大多数配置系统都允许通过定义配置文件的处理顺序来做到这一点。它们通常从最通用的配置文件开始，然后用更具体的配置文件覆盖配置值。如果图书馆允许这样做，我随时都可以接受

示例:您的数据库连接池设置在所有环境中都是相同的(假设最小值:2，最大值:10)。但是对于生产环境，您需要更积极的池(最小值:10，最大值:80)。他们说你会用`config`做那件事

**default.js:**

```
**const** config = {
  database: {
    client: 'mysql',
    connection: {
      host: 'localhost',
      user: '',
      password: '',
      database: '',
    },
    pool: {
      min: 2,
      max: 10,
    },
  },
}

module.exports = config;
```

**production.js:**

```
**const** config = {
  database.pool: {
     min: 10,
     max: 80,
  },
}

module.exports = config;
```

## 简单

> 一个设计师知道他已经达到了完美，不是当没有什么可以添加的时候，而是当没有什么可以拿走的时候。
> 
> 安托万·德·圣埃克苏佩里

当然，至少有一些解决上述问题的方案。问题是，您真的想在基础架构中安装额外的组件来解决这个问题吗？额外的组件带来了复杂性的增加，这导致了错误率的增加、调试时间和反应时间的增加

我用过的一些解决方案:

1.  hashi corp Vault——适合秘密管理，但有点大材小用，除非你在几十个场景中有超过 9000 个微服务
2.  配置管理的主厨/傀儡/负责人——根据我的经验，这些都是设置服务器的很好的工具，但是当涉及到定制软件配置时，总会出现问题
3.  共识系统——动物园管理员/etcd/consul。有些人用它们来存储所有的配置，并让应用程序与它们交互。大多数时候，当人们已经有了这些系统中的一个时，他们就会这么做(例如，运行 Kubernetes 的人有现成的`etcd`)。

我已经完成了大部分的成功案例。有一个著名的俄罗斯习语“用显微镜敲钉子”，我总是觉得那正是我正在做的

## 我的路

在节点中。JS 有一个很好的[配置库](https://www.npmjs.com/package/config)为我做了所有的诡计和欺骗。项目结构:

`/
— package.json
— /src
— /config
— — /local.js
— — /default.js
— — /development.js
— — /production.js
— — /<otherenv>.js
— — /custom-environment-variables.js`

文件内容如下:

*   local.js —包含完整的配置，应该放在`.gitignore`下，永远不要出现在 repo 上
*   default.js —包含由所有环境共享的顶级通用配置密钥，保密值为空白
*   <env.js>—包含特定环境的特定覆盖。使用`NODE_ENV`环境变量定义环境</env.js>
*   custom-environment-variables . js—将秘密配置密钥映射到稍后在运行时注入的 env 变量

我通常和 Kubernetes 一起使用，Kubernetes 允许[存储秘密值](https://kubernetes.io/docs/concepts/configuration/secret/)，并以一种方便的方式使用环境变量注入它们

回购的例子是这里的[和](https://github.com/schikin/example-node-config)

快乐黑客和不要重新发明轮子！