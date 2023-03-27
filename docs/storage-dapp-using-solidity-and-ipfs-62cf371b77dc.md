# 使用可靠性和 IPFS 的存储 DApp

> 原文：<https://itnext.io/storage-dapp-using-solidity-and-ipfs-62cf371b77dc?source=collection_archive---------0----------------------->

想象一下一个分散式存储应用程序，我们彼此共享磁盘，中间没有任何服务器。现在想象一下，这个分散式应用程序中的每个文件都可以很容易地共享，并且只需一个简单的散列就可以找到。听起来很有希望，对吗？这是一个正在进行的项目，我将解释它是如何工作的。

![](img/81c73249d41e5dbb034f3d77a0e39a80.png)

由 [Shubham Dhage](https://unsplash.com/@theshubhamdhage?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

# 介绍

> TL；DR:你可以从 [*G* ithub 库](https://github.com/MCarlomagno/persssist)访问代码(并留下一个星号*😉*)。

这一切都是从一个问题开始的，当我试图找出如何在不丢失 dApp 定义的情况下在 dApp 中存储大文件时，这个问题出现了。如果应用程序将文件存储在一个集中的存储服务中，那么应用程序就不再是分散的了。

所以经过一番研究，我找到了一个令人震惊的解决方案:[**IPFS**](https://ipfs.io/)**🤯。**

## **IPFS 的一段话**

**IPFS 是一个 ***对等超媒体协议*** ，意思是跨计算机共享媒体的协议。通过使用这个协议，你可以很容易地建立 p2p 网络，在节点间共享信息(其中一个节点可以是你的计算机)。**

**幸运的是，存在一个用于访问该协议并与之交互的高级 Javascript 包，这意味着您可以构建一个前端应用程序，使用您的计算机作为一个节点，通过网络共享和访问文件。我猜你已经知道这个应用程序是如何工作的了😉。**

**但接下来的问题是如何组织文件、用户，以分散的方式引用文件元数据，并创建普遍可访问的信息。这就是区块链和智能合约来帮助我们的地方。**

## **智能合同**

**智能合同是一个简单的程序，通过在区块链网络中运行交易的地址来标识。我不打算深究这个概念，但简而言之，我们可以将智能合约作为一个微型数据库，因为它们具有不可改变的性质。**

# **构建应用程序**

## **创建可靠的智能合同**

**第一部分包括创建一个关于 Solidity 语言的智能合同，用于存储上传到应用程序的文件信息。我们将保存关于它们的一般信息，如文件名、类型、大小等。**

**如你所见，代码非常简洁直接，我们只有三个变量，一个函数，一个事件和一个表示系统中一个文件的结构。代码是不言自明的，但是为了更好地理解，我添加了一些注释。**

## **使用 Truffle 和 Ganache 的智能合约部署**

**调试和测试智能合约的最方便程序员的方式是使用本地开发环境，如 [Truffle](https://trufflesuite.com/) 来轻松地为我们的项目创建良好的管道。**

**调试智能合同的另一个非常重要的工具是[Ganache](https://trufflesuite.com/ganache/)(Truffle 套件的一部分)，这是一个本地区块链，你可以运行它来部署和调试你的合同，它们提供了一组开箱即用的帐户和配置，有足够的以太来做你在项目中想要做的几乎任何事情。**

**所以下一步是在我们的项目上设置 truffle，因为这样我们可以简单地用 npm 安装它。**

```
npm install -g truffle
```

**然后你可以用你想要的任何框架/库创建你的前端项目，在我的例子中是 next js。一旦你创建了它，你就可以在它上面初始化一个 truffle 环境，把控制台放在项目的根文件夹中，输入:**

```
truffle init
```

**这将为你的项目创建一个基本的配置，在其他文件中会创建一个 *truffle-config.js* ，这个文件在项目期间会特别重要。**

> **你可以在这里查看项目配置[。](https://github.com/MCarlomagno/persssist/blob/main/truffle-config.js)**

**为了在项目中实际设置我们的智能合约，我们应该运行一个迁移，您可以按照[这些步骤](https://trufflesuite.com/docs/truffle/getting-started/running-migrations/)来完成。**

## **测试合同**

**这一部分是智能合约开发中最重要的实践之一，智能合约应该尽可能好地进行测试，以便放心地部署它。由于智能契约的本质是不可变的，部署一个有缺陷的契约的成本是非常高的。**

**为了测试合约，松露配有摩卡和茶，这使得测试变得容易得多。**

**让我们来看一些智能合约测试的基本例子，你可以在这个测试文件中看到全套的测试。**

****测试合同部署正确:****

****测试上传有效文件:****

## **使用元掩码验证**

**一旦我们完成了应用程序的基本配置，我们就可以开始使用元掩码进行用户身份验证，为此，我们有一个 js API 来连接应用程序和扩展，而不需要任何库。**

**为了创造流畅的用户体验，我们可以创建一种方法，在存在现有帐户时自动连接。**

**另一种强制请求的方法是打开元掩码弹出窗口进行身份验证。**

## **用智能合同连接前端**

**在这个阶段，我们已经有了一个与元掩码扩展连接的前端应用程序和一个与 Ganache 一起运行的智能契约，现在我们需要找到一种方法以 API 的形式与 Solidity 契约进行交互。**

**为此，我们将需要一些库，如 [Web3.js](https://web3js.readthedocs.io/en/v1.7.3/) 或 [Ethers.js](https://docs.ethers.io/v5/) 。我选择了 Web3.js。**

**导入 web3 并创建合同对象**

> **注意:有一个特殊的文件叫做‘Persssist’，这就是 **abi** 。JSON 格式的智能契约表示，作为 Javascript 和 Solidity 之间的接口。**

## **将智能合同与 IPFS 连接**

**一旦我们有了合同对象，我们就可以开始使用 IPFS 上传文件。这里的技巧是，我们可以使用唯一的路径引用 IPFS 的文件，该路径作为在 IPFS 文件系统中创建的每个文件的 Id。**

**处于智能合同状态的每个文件都将引用 IPFS 文件系统中的一个路径，通过这种方式在合同和存储系统之间创建一个可靠的连接。**

**中间唯一的元素是为每个用户在本地浏览器上运行的前端应用程序，这样就创建了一个分散的存储应用程序。**

## **上传文件**

**该任务由 2 个步骤组成:**

1.  **上传文件到 IPFS 获得其唯一的路径。**
2.  **使用结果在智能合同中创建文件记录。**

**首先我们创建 IPFS 连接，这不是每次用户上传文件都需要的，但是我在这个函数中这样做只是为了让它更清晰。**

**然后，我们只使用缓冲区和文件类型上传文件，结果我们获得了 IPFS 文件系统上的文件路径。**

**第二步是将文件信息实际存储在智能合约中，以便将来轻松下载该文件。**

## **获取和下载文件**

**对于这一步，我们将做相反的过程。我们需要获取存储在智能合约中的文件，然后使用惟一的路径从 IPFS 文件系统下载文件。**

**为了创建一个用户友好的交互，我们可以获取所有的文件，只下载用户选择的文件。**

**在用户选择一个文件进行下载后，我们可以开始下载过程。这是最困难的部分之一，因为 IPFS 只下载压缩格式的文件，所以我们需要做一些变通来获得正确的文件格式，在这种情况下通过添加 **untar** 库。**

# **结论**

**分散式应用程序是增强互联网用户和 web 2.0 开发者能力的绝佳机会。不需要在两个人中间有什么东西来创建一个可信的和流畅的交互，这只是(我希望)我们在不久的将来将拥有的去中心化互联网的无限例子之一。**

**如果你在这里，我相信你和我一样热爱编程，所以我强烈建议你检查代码，如果你敢，贡献并从中做出更大更好的东西。非常感谢你的阅读！**

> **注意:你可以从 [*G* ithub 库](https://github.com/MCarlomagno/persssist)访问代码(并留下一个星号*😉*)。**