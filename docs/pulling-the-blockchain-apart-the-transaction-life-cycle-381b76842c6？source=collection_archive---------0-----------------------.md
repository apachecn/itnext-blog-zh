# 拉开区块链。交易生命周期。

> 原文：<https://itnext.io/pulling-the-blockchain-apart-the-transaction-life-cycle-381b76842c6?source=collection_archive---------0----------------------->

## 打开神奇东西的引擎盖

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fpulling-the-blockchain-apart-the-transaction-life-cycle-381b76842c6)

解开区块链是什么，它是如何工作的，以及它的好处是非常困难的。我花了好几个星期才大致了解情况。

因此，我将分享我的旅程和理解，以帮助其他人获得更多这方面的知识。这样在和区块链的顾问谈话时就不会有人卖给我们蛇油了。

区块链背后的思想是各种复杂思想和系统的结合。虽然这种组合提供了重要的价值，但由此产生的复杂性使得很难在短时间内理解这个想法。

![](img/a6ddb24037ea85b12ab78af3dfba44bd.png)

[https://medium . com/the-mission/a-brief-history-of-区块链-an-investors-perspective-e9b 6605 aad 68](https://medium.com/the-mission/a-brief-history-of-blockchain-an-investors-perspective-e9b6605aad68)

# 把它拆开

![](img/c0a776b452119546e95b94b40354e426.png)

[https://bitsonblocks . net/2015/09/09/a-gentle-introduction-to-区块链-technology/](https://bitsonblocks.net/2015/09/09/a-gentle-introduction-to-blockchain-technology/)

在前一篇博客中，我说过区块链只不过是一个事务的集合。这些事务被封装在 X 个事务的包中。这些包被称为以特定顺序链接的块，因此得名区块链。区块链相当于菊花链:)

![](img/15425bf346898a3f2f7611499673c917.png)

[https://commons.wikimedia.org/wiki/File:Daisy_chain.JPG](https://commons.wikimedia.org/wiki/File:Daisy_chain.JPG)

要理解组件，就要求把握全貌。让我们先来看看区块链上的事务的生命周期。

# 事务生命周期

下面我们有一些区块链(和比特币)交易的可视化。

总的来说，步骤的顺序是:

1.  有人**通过一个叫做钱包的东西请求交易**。
2.  **交易被发送**(广播)到特定区块链网络中的所有参与计算机。
3.  网络中的每台计算机都根据特定区块链网络的创建者设置的一些验证规则来检查(**验证**)交易。
4.  经过验证的事务**存储在块**中，并用锁(哈希)密封。
5.  当网络中的其他计算机**验证**该块上的锁是否正确时，该块成为区块链的一部分。
6.  现在交易是区块链的一部分，不能以任何方式改变。

在下面的图形中说明了这些步骤，我也写下了这些步骤。每个人都用不同的方式命名这些步骤，但总体内容是相同的。

## 保险基金 a

![](img/61c1e8c4b381aa7244fe98c6005400a5.png)

[http://insurancefunda.in/bitcoin-cryptocurrency/](http://insurancefunda.in/bitcoin-cryptocurrency/)

这是理解区块链步骤的最清晰和最简单的概述。这些步骤是:

1.  有人请求交易
2.  向 P2P 计算机(节点)广播的事务。
3.  验证，矿工验证交易。
4.  组合成一个数据块的事务。
5.  添加到现有区块链的新区块。
6.  交易完成。

## 德勤

![](img/c8b7eeb3e483e890da61df3d409c40da.png)

[https://dupress . Deloitte . com/dup-us-en/focus/tech-trends/2016/区块链-全球经济中的应用和信任. html](https://dupress.deloitte.com/dup-us-en/focus/tech-trends/2016/blockchain-applications-and-trust-in-a-global-economy.html)

当你点击[信号源](https://dupress.deloitte.com/content/dam/dup-us-en/articles/blockchain-applications-and-trust-in-a-global-economy/DUP3039_TT16Blockchain_Figure1.jpg)时，图像具有高分辨率。所描绘的步骤或元素是:

1.  交易
2.  确认
3.  结构
4.  确认
5.  区块链挖掘
6.  链条
7.  内置防御。

## 华尔街分析师

![](img/cc563c1339358782ebd000ac1ba94b05.png)

[http://wallstanalyst . com/bit coin-progression-against-world-acceptance/](http://wallstanalyst.com/bitcoin-progression-towards-world-acceptance/)

这篇文章中的描述非常清楚。这里确定的步骤是:

1.  发送付款的目的地地址。
2.  支付的目的地地址被公布到网络上，让所有人都能看到。
3.  从起始地址到目的地址的支付交易安全完成。
4.  交易由网络和区块链确认、处理和保护。

## i-Scoop &埃森哲

![](img/eb923d18326fda6519125202349c1373.png)

[https://www . I-scoop . eu/fin tech/区块链-distributed-ledger-technology/](https://www.i-scoop.eu/fintech/blockchain-distributed-ledger-technology/)

这张图片是埃森哲制作的一张更大的信息图的一部分。这里确定的步骤是:

1.  向网络提交交易请求。
2.  网络验证交易。
3.  经验证的事务与其他经验证的事务组合成区块链中的一个块。
4.  交易完成。

## 班博拉

![](img/f3f144216b67f87130d6011e6bacad6e.png)

[https://www.bambora.com/en/ca/blog/bitcoin-explained/](https://www.bambora.com/en/ca/blog/bitcoin-explained/)

1.  打开你的钱包，扫描你想要汇款的地址。
2.  选择金额并发送交易。
3.  钱包保证支付，所以你知道钱的发送者。
4.  交易由网络验证，并成为挖掘过程的一部分。
5.  采矿正在进行中，当一个矿工赚到一个比特币时就完成了。
6.  网络验证挖掘过程的结果。
7.  收款人收到交易成功的确认。

## 泰克诺骆驼

![](img/e9130efa29ddf90edc9cf6988e05a51f.png)

[http://www . technola . co . uk/区块链-权力下放的挑战](http://www.technollama.co.uk/blockchains-and-the-challenges-of-decentralization)

这个博客很抽象，但也能给人新的见解。定义如下:

1.  有从 A 向 b 寄钱的意向。
2.  事务被放入一个块中。
3.  该块被发送给网络的所有成员。
4.  网络验证该块。
5.  该块被添加到链中。
6.  钱从 A 地转移到 b 地。

## 零对冲

![](img/29f83c44f924561cbbefa17f6b8cc56f.png)

[http://www . zero hedge . com/news/2013-05-12/visualizing-how-bit coin-transaction-works](http://www.zerohedge.com/news/2013-05-12/visualizing-how-bitcoin-transaction-works)

这是一个很好的路线图，但是对于没有任何知识的人来说太过宽泛:

1.  **钱包&地址:** 交易的发送方和接收方都有一个钱包，里面装着有钱的地址，需要的时候可以创建新的地址。
2.  **创建新的(接收)地址:** 创建一个新的接收交易款项的地址。
3.  **提交支付:** 在这里，发送者告诉钱包应该向接收地址发送多少钱，这被转换成交易。
4.  **验证交易:** 在该步骤中，交易由网络中的计算机进行验证，并被捆绑在交易块中
5.  **密码散列:** 用密码散列将块锁定在一起。
6.  **交易已验证:** 交易作为区块链的一部分进行验证，因此无法更改。