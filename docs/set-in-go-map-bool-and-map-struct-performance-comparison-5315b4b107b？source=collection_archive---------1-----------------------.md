# Set in Go、map[]bool 和 map[]struct{}性能比较

> 原文：<https://itnext.io/set-in-go-map-bool-and-map-struct-performance-comparison-5315b4b107b?source=collection_archive---------1----------------------->

![](img/c8be7508964ca4629be9b5bbabed97b7.png)

> TL；当遇到大数据集时，DR `map[]struct{}`比`map[]bool`在时间上快 5%,内存消耗少 10%。

围棋中没有内置的 Set 已经不是什么秘密，所以围棋开发者用`map`来模仿 Set 的行为。使用`map`实现 Set 意味着 map 的值并不重要，我们只需要关注 key 的存在。大多数时候，人们可能会选择`bool`，因为它是内存消耗最少的类型，但在围棋中，还有一个选择是使用空的`struct`。在这篇文章中，我们将对它们进行基准测试，看看是否有什么不同。

# 设置

为了获得足够的数据来进行比较，我们首先声明不同类型的`map`,然后用相应的类型从 0 到 2 写/设置它的键⁴ -1，并观察它需要多少时间和内存。

我们可以使用 golang 中内置的基准测试机制来实现。只有几行代码，我们就可以开始了:

> `map[uint]interface{}`是加成，预计会比较慢。

由于任何键的读性能应该是相似的，这里我们只对写/设置操作进行基准测试。

执行命令可以产生统计数据:

现在让我们检查结果。

# 结果

```
name                        time/op 
SetWithBoolValueWrite        3.27s ± 0% 
SetWithStructValueWrite      3.12s ± 0% 
SetWithInterfaceValueWrite   5.96s ± 0% 

name                        alloc/op 
SetWithBoolValueWrite        884MB ± 0% 
SetWithStructValueWrite      802MB ± 0% 
SetWithInterfaceValueWrite  1.98GB ± 0%
```

从上面的结果来看，`*map[]struct{}*`在大数据集上比`*map[]bool*`快 5%,内存消耗少 10%。(使用`map[]interface{}`是一场灾难，因为它增加了很多开销，几乎使时间和内存翻倍。)

# 结论

虽然`map[]struct{}`速度更快，占用内存更少，但是到了大套的时候就更明显了。所以如果你很急，资源不是你最关心的，使用`map[]bool`应该完全没问题。如果你追求的是极致的性能，键的类型是 uint，那么更推荐 bitset。

希望你喜欢这篇文章，并随时检查这个[回购](https://github.com/jeromewu/go-set-benchmark)的源代码。😃