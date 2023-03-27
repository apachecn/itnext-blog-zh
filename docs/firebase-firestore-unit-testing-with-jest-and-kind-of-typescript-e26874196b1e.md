# 用 jest(和某种类型的脚本)进行 Firebase Firestore 单元测试

> 原文：<https://itnext.io/firebase-firestore-unit-testing-with-jest-and-kind-of-typescript-e26874196b1e?source=collection_archive---------3----------------------->

# 这花了一些时间，但我想我明白了

![](img/1b99fc0adba685fd2ac0c74269930dc4.png)

费伦茨·阿尔马西在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

前几天我在做一个 firebase 项目。项目是用 typescript 写的，挺好的。它基本上是一组与 Firestore 数据库交互的云功能。

我想编写单元测试，并且希望测试尽可能快地运行，而不依赖于真实的 Firestore 实例——所以我必须模拟 Firestore 调用。

很公平，应该很容易…嗯…

我浪费了太多的时间试图模仿 Firestore 类型的复杂结构，使用了太多的 Firestore 内部数据结构，当我决定我一定是错了。如果嘲笑很难，那你就做错了。

假设我想模拟一个文档的更新调用。

```
await collection('myCollection')
            .doc('myDoc')
            .update({ updated:true});
```

我必须模拟三个函数——我正在使用的 firestore 实例上的 collection、doc 和 update。所以如果我试着这样做:

```
import * as admin from 'firebase-admin';let update = jest.fn()
let doc = jest.fn(()=>({update})
jest.spyOn(admin.firestore(), 'collection').mockReturnValue({ doc });
```

编译器不喜欢它，它说:

```
Argument of type ‘{ doc: any; }’ is not assignable to parameter of type ‘CollectionReference<DocumentData>’.
 Type ‘{ doc: any; }’ is missing the following properties from type ‘CollectionReference<DocumentData>’: id, parent, path, listDocuments, and 16 more.
```

问题是，我真的不需要 Firestore 集合对象或文档对象的大部分功能，所以我不想浪费时间嘲笑它。

**解决方法很简单，不要嘲笑你不需要的东西。**

只需将返回值设为未知类型，然后将其设为任意类型。这是一种反类型脚本，但对于嘲笑外部依赖，我认为我可以接受。

```
import * as admin from 'firebase-admin';const update = jest.fn();
const doc = jest.fn(() => ({update}));
const collection = jest.spyOn(admin.firestore(), 'collection').mockReturnValue((({ doc } as unknown) as any);
```

**瞧啊**！

我现在可以期待我想要的这些模拟:

```
expect(collection).toHaveBeenCalledWith('myCollection');expect(doc).toHaveBeenCalledWith('myPost');expect(update).toHaveBeenCalledWith({updated:true});
```

开心嘲讽！

## 额外的东西

我已经介绍了如何模拟数据库，但是高级前端的东西呢？

如果你感兴趣，看看这篇很酷的文章:

[](https://yonatankra.com/how-to-test-html5-canvas-with-jest/?fbclid=IwAR1QxAysPZLwDCdwI_RPNqqbwy-6qXRoBPsW01W45LBnluHXFxYfF1arldQ) [## 如何用 jest 测试 HTML5 canvas？

### 在这篇短文中，您将了解到为 canvas 准备测试环境需要安装什么…

yonatankra.com](https://yonatankra.com/how-to-test-html5-canvas-with-jest/?fbclid=IwAR1QxAysPZLwDCdwI_RPNqqbwy-6qXRoBPsW01W45LBnluHXFxYfF1arldQ)