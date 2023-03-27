# 递归清理 Firestore FieldValue.delete()

> 原文：<https://itnext.io/recursively-clean-firestore-fieldvalue-delete-60b7dff00bf4?source=collection_archive---------3----------------------->

## 如何递归地从你刚刚更新并存在内存中的文档对象中移除删除方法。

![](img/cda6f4def6e13c76606ea5360b2f5097.png)

照片由[在](https://unsplash.com/@thecreative_exchange?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的创意交流拍摄

今天早上我必须改进我们在 [DeckDeckGo](https://deckdeckgo.com) 中使用的一个函数，以便在持久化之后递归地清理对象。因为我目前很忙，但不想过多地忽视我的博客习惯，所以我认为这个小小的“黑客”将是一篇新博文的好主题🤗。

# 介绍

当您使用[云 Firestore](https://firebase.google.com/docs/firestore/) 时，为了从文档中删除特定字段，您必须在更新文档时使用`FieldValue.delete()`方法(如[文档](https://firebase.google.com/docs/firestore/manage-data/delete-data)所示)。

例如，您的数据库包含如下所示的文档:

```
{
  description: 'Hello World'
}
```

你必须使用上面的方法来删除它，因为将它设置为`null`不会删除属性，但是“仅仅”将它的值设置为`null`。

```
import * as firebase from 'firebase/app';
import 'firebase/firestore';

const firestore = firebase.firestore();

const ref = firestore.collection('users').doc('david');const user = {
  description: firebase.firestore.FieldValue.*delete*()
};await ref.update(user);
```

由于这种方法，上述文档的例子在数据库中变成了`{}`。

# 问题

这种方法非常有效，但是会导致一个问题。事实上，如果你在本地对象更新后不刷新它，它仍然会包含方法`FieldValue.delete()`，这并不反映它在数据库中的有效值。

具体来说，在上面的例子中，如果我们将`user`输出到控制台，它的输出将如下所示。

```
{
  description: n {h_: n}
}
```

如果对象更新后仍在使用，特别是当它是一个状态时，这可能会导致应用程序中出现一些意外的行为。

为了克服这个问题，一个解决方案是显式地从 Firestore 获取最新更新的文档，如果您已经开发了一些轮询来获取信息，或者如果您正在使用诸如 [AngularFire](https://github.com/angular/angularfire) 或 [RxFire](https://github.com/firebase/firebase-js-sdk/tree/master/packages/rxfire) 之类的库，这也会自动发生。

```
import * as firebase from 'firebase/app';
import 'firebase/firestore';

const firestore = firebase.firestore();

const ref = firestore.collection('users').doc('david');let user = {
  description: firebase.firestore.FieldValue.*delete*()
};await ref.update(user);user = ref.get();console.log(user); // {}
```

这种解决方案的优点是让您的对象与数据库保持同步，但缺点是需要额外的查询。

事实上，当你使用 [Cloud Firestore](https://firebase.google.com/docs/firestore/) 时，你会根据你执行的读取、写入和删除次数[付费。因此，根据其频率，多一个查询会导致更多的成本。](https://cloud.google.com/firestore/pricing)

这就是为什么我想到了递归清理方法`FieldValue.delete()`，为什么我有了“黑客”的想法😎。

# 解决办法

下面的函数`filterDelete`迭代一个对象的所有`keys`，并识别出那些必须被忽略的(`shouldAttributeBeCleaned`)，这些包含了方法`FieldValue.delete()`。

如果没有被忽略，那么它递归调用当前子节点的函数`filterDelete`，直到所有子节点都以同样的方式被处理。

此外，当缩减器用空对象`{}`初始化时，它还必须检查对象的有效值是否不为空，以便不将空叶子添加到累加器中。

```
export function filterDelete<T>(obj: T): T {
  if (typeof obj !== 'object' || Array.isArray(obj)) {
    return obj;
  }

  return Object.keys(obj)
    .filter((key) => !shouldAttributeBeCleaned(obj[key]))
    .reduce((res, key) => {
      const value: T = filterDelete(obj[key]);

      if (value && typeof value === 'object') {
        if (Object.keys(value).length > 0) {
          res[key] = value;
        }
      } else {
        res[key] = value;
      }

      return res;
    }, {} as T);
}

function shouldAttributeBeCleaned<T>(attr: T): boolean {
  if (typeof attr !== 'object' || Array.isArray(attr)) {
    return false;
  }

  return JSON.stringify(attr) === JSON.stringify(firebase.firestore.FieldValue.*delete*());
}
```

多亏了这个函数，我能够实现与从数据库中获取更新文档完全相同的行为。

```
import * as firebase from 'firebase/app';
import 'firebase/firestore';

const firestore = firebase.firestore();

const ref = firestore.collection('users').doc('david');let user = {
  description: firebase.firestore.FieldValue.*delete*()
};await ref.update(user);console.log(filterDelete(user)); // {}
```

# 限制

这种策略的主要限制是它依赖于 [Firebase](https://github.com/firebase/firebase-js-sdk) 库。每次更新后，检查它是否仍然工作是值得的，因为方法`FieldValue.delete()`的检测可能必须在版本之间改变。我以前也遇到过这种情况，所以如果你想使用这个功能，一定要小心。

如果您使用它，我还可以建议您特别注意更新和清理之间的错误处理，因为您可能希望避免本地对象的值不等于它们的数据库值(“不同步”)的情况。

# 结论

您可能会注意到上述解决方案中的一些潜在改进。DeckDeckGo 是开源的，因此我将非常乐意得到您对此函数的[代码源的贡献。毕竟现在还是 2020 年的 Hacktoberfest😎。](https://github.com/deckgo/deckdeckgo/blob/master/studio/src/app/utils/editor/firestore.utils.tsx)

到无限和更远的地方！

大卫

在 Twitter 上联系我，为什么不试试在 T2 的 DeckDeckGo 上做你的下一次演讲。

它将你的幻灯片作为渐进式网络应用程序部署在网上，甚至可以将你的幻灯片的源代码推送到 GitHub。