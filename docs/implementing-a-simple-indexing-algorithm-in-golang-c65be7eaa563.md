# 用 golang 实现一个简单的索引算法

> 原文：<https://itnext.io/implementing-a-simple-indexing-algorithm-in-golang-c65be7eaa563?source=collection_archive---------5----------------------->

![](img/fbfc3bb3ae9e91f0dd71b5e2626431ec.png)

安东尼·马蒂诺在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

在过去的几周里，我在做一个兼职项目，让 outlook 会议预约变得更加容易。

所以我开始在 golang 做一个简单的 [MS 交流会组织者(为什么是 golang，因为我是新手，需要多练习)。](https://github.com/mhewedy/mego)

我已经在一个[单独的仓库](https://github.com/mhewedy/ews)中实现了 EWS (Exchange Web 服务)客户端，并将主项目放在另一个[仓库](https://github.com/mhewedy/mego)中

我已经完成了大部分工作，除了一件重要的事情，如何搜索联系人的名字？

![](img/e8b8233a2a7c1cd4f973fcb5a4f4577c.png)

MEGO 中的搜索界面

几年前，我们在用 Java 做全文搜索，当时我和一位杰出的软件架构师一起工作，他向我们介绍了 Apache Lucene。

Lucene 使用了倒排索引的思想，在倒排索引中，令牌以链接到保存相关结果的文档的方式存储。所以我决定用同样的想法做一个简单的实现。

> 我将使用所谓的 **N-grams** 来将文本分成**索引时间**可搜索项。

因为我的例子中的数据不是很大，可能有几千个联系人，所以我决定将索引保存在内存中(作为包级别的变量):

```
var indexDB = make(map[token][]*Attendee)
```

下面是**索引**函数的实现:

```
func index(attendees []Attendee) {
    for i, aa := range attendees { // index the DisplayName field
        doOnToken(aa.DisplayName, func(t token) []Attendee {
            indexToken(t, &attendees[i])
            return nil
        }) // index the EmailAddress field
        email := strings.Split(aa.EmailAddress, "@")[0]
        doOnToken(email, func(t token) []Attendee {
             indexToken(t, &attendees[i])
             return nil
        })
    }
}
```

我们在这里索引**display name**和 **EmailAddress** 字段，所以对于其中的每一个字段，我们调用 **doOnToken** 函数，该函数将其输入转换为 **Token** ，然后对每个 token 执行 **indexToken** 。

> 注意， **doOnToken** 函数是在索引和搜索的输入上调用的，因此被索引的相同输入也将被搜索。

```
func doOnToken(input string, fn func(t token) []Attendee) [][]Attendee { lower := strings.ToLower(input)
    clear := substituteVowels(lower)
    fields := strings.Fields(clear) ii := make([][]Attendee, 0) for _, field := range fields {
        tokens := tokenize(field, 4) for _, t := range tokens {
            ii = append(ii, fn(t))
        }
    }
    return ii
}
```

我们在 doOnToken 函数中有 3 个过滤器，lower、substituteVowels 和 fields 过滤器，分别用于降低、替换元音和在空间上分割。

> 起初，该算法用于完全删除元音，结果不太好，但在一个朋友的建议下，我用替换元音来代替，所以每当 doOnToken 函数表示的索引器/搜索器遇到“a”、“e”或“u”时，我就用“e”来替换，每当遇到“o”或“u”时，就用“o”来替换。

该索引现在应该包含如下数据:

```
moh => [mohammad, mohannad ...] 
mad => [mohammad, emad, ...]
han => [hanan, mohannad, ...]
....
```

现在让我们来看看搜索函数，它既简单又高效，它使用相同的 doOnToken 函数来计算令牌，然后因为我们的 **indexDB** index 是基于散列表的，所以检索操作应该和 ***O(1)*** 一样快。

```
func search(input string) []Attendee { temp := doOnToken(input, func(t token) []Attendee {
        attendees := indexDB[t]
        result := make([]Attendee, len(attendees))
        for i, a := range attendees {
            result[i] = *a
        }
        return result
    }) return sortByOccurrence(temp)
}
```

这里一个棘手的部分是，我们通过事件搜索结果。例如，如果我们有以下结果:

```
moh => [mohammad, mohannad] 
mad => [mohammad, emad, ahmad]
```

然后，输出将按出现次数排序，因此结果将排序为:

```
mohammad
emad
ahmad
```

能够省略出现次数< n.

Here’s the full implementation of the indexer:

[](https://github.com/mhewedy/mego/blob/5607bad4b5fd8c07312e43dd07281fa40c153cc8/mego-api/attendess/indexer.go) [## mhewedy/mego

### You can't perform that action at this time. You signed in with another tab or window. You signed out in another tab or…

github.com](https://github.com/mhewedy/mego/blob/5607bad4b5fd8c07312e43dd07281fa40c153cc8/mego-api/attendess/indexer.go) 

[https://github.com/mhewedy/mego](https://github.com/mhewedy/mego)的名字