# 区块链和哈希技术

> 原文：<https://itnext.io/blockchain-and-hashing-technology-1691d9710f6e?source=collection_archive---------1----------------------->

区块链技术的基础部分是散列函数。如果你了解哈希函数，你就能更容易理解相关概念。

![](img/a1a8a403e5c599a954b6640875a2181b.png)

区块链和哈希技术

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fblockchain-and-hashing-technology-1691d9710f6e)

## 哈希是什么？

为了得到固定长度的输出，我们使用散列函数。哈希函数不会考虑给定输入的长度。哈希函数是在哈希算法的帮助下执行的。

## [哈希技术在区块链](http://www.blockchainexpert.uk/blog/hashing-technology-in-blockchain)中是如何使用的？

为了将新的交易写入区块链，我们使用哈希，这是一个数学过程。我们必须确保以下特性，才能将哈希函数视为安全函数:

即使我们多次执行相同的输入，输出或散列应该是相同的。它应该很快产生输出。从 H(A)中找到一个输入‘A’应该是不切实际的。随着输入的小变化，散列应该会有很大的变化。散列值应该是唯一的。

## 数据结构

指针和链表主要是区块链使用的两种数据结构。指针还存储包括地址在内的前一个块的哈希值。

区块链是一个链表，每个节点存储一个哈希指针和一个数据头。

## 采矿过程

这个过程用于向区块链添加新的区块。如果是有效交易，矿工将交易添加到区块链。即时创建更多的哈希函数可能会导致哈希冲突。

访问 [BlockchainExpert](http://www.blockchainexpert.uk/) 获取更多详细描述。

*原载于 2017 年 11 月 24 日*[*medium.com*](https://medium.com/blockchainexpert-blog/blockchain-and-hashing-technology-d3a4701ddea0)*。*