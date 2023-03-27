# 在索引数据库中搜索

> 原文：<https://itnext.io/searching-in-your-indexeddb-database-d7cbf202a17?source=collection_archive---------0----------------------->

![](img/af3fd96338f927c62f9bcc863c8983f9.png)

*这是我上一篇* *关于 IndexedDB 入门的* [*的延续。如果您不熟悉 IndexedDB，请阅读该文章以熟悉它的基础知识。*](/getting-started-with-persistent-offline-storage-with-indexeddb-1af66727246c)

IndexedDB 是一个非常棒的文档存储数据库，可以离线保存数据，并具有所有基本 CRUD 操作的原生功能。使用任何数据库的下一步(可能更重要)是如何在其中找到数据？我们已经看到，当我们知道创建数据的关键路径时，IndexedDB 如何检索数据。但是，当我们想要搜索包含特定文本或满足特定条件的文档时，该怎么办呢？

嗯，答案是…不像做 CRUD 那么容易。

IndexedDB 的不幸问题是它没有本地搜索功能。在使用了 Mongo 等其他 NoSQL 数据库后，这有点令人失望，因为使用他们的 API 查找文档太容易了。为了进行任何类型的搜索，我们必须自己使用我们可用的基本构件来实现它。

这就引出了对*光标*的讨论。

# 光标到底是什么？

简而言之，游标是一个指针，它遍历给定数据存储或索引中的所有文档，并且在每次迭代中显示当前“指向”的文档的数据。它还包含一些额外的元数据和一些方法。

您可以在 MDN，上阅读 cursor [的完整规格，但出于我们的目的，我们对以下属性和方法感兴趣:](https://developer.mozilla.org/en-US/docs/Web/API/IDBCursor)

*   `primaryKey`:定义了存储中某个对象的关键路径
*   `value`:文件本身
*   `continue()`:移动到下一个文档的指令
*   `delete()`:删除当前文档的指令
*   `update()`:更新当前文档的指令

其中一些(尤其是我在 MDN 中遗漏的)或多或少会很重要，这取决于您的用例是什么，但是我想展示几个最重要的技术，因为尽管 IndexedDB 有点笨拙，但光标是一个强大的工具。

# 在我们的数据库里找到一个光标

在本文的其余部分，我们将使用我上一篇文章中的数据库(这里是[代码](https://gist.github.com/ryandabler/41c458871cb6b64f599fb182ba88daba))以及来自[这个要点](https://gist.github.com/ryandabler/3a0b2999e9e96045689de7ea0a7d5a3b)的数据集作为我们数据库的种子，这样我们就有东西可以搜索了。

现在，我们的数据库中有了数据，让我们打开一个游标。

```
const request = window.indexedDB.open('database', 1);request.onsuccess = () => {
    const db = request.result;
    const transaction = db.transaction(['invoices'], 'readonly');
    const invoiceStore = transaction.objectStore('invoices');
    const getCursorRequest = invoiceStore.openCursor(); getCursorRequest.onsuccess = e => {
        // Cursor logic here
    }
}
```

记住 IndexedDB 是基于事件的，所以我们所有的请求都会触发`onsuccess`事件，我们需要在这些事件中监听和实现我们的应用程序逻辑。在上面的代码中，如果数据库成功打开，我们将经历典型的 IndexedDB 链，即启动一个事务并提取一个对象存储。一旦我们有了商店，只需在上面打开一个光标。

但是，因为打开游标是一个请求，所以我们必须处理它的`onsuccess`来实现我们的游标逻辑。所以让我们打印出发票存储中的对象:

```
const request = window.indexedDB.open('database', 1);request.onsuccess = () => {
    const db = request.result;
    const transaction = db.transaction(['invoices'], 'readonly');
    const invoiceStore = transaction.objectStore('invoices');
    const getCursorRequest = invoiceStore.openCursor(); getCursorRequest.onsuccess = e => {
        const cursor = e.target.result;
        if (cursor) {
            console.log(cursor.value);
            cursor.continue();
        } else {
            console.log('Exhausted all documents');
        }
    }
}
```

关于正在发生的事情，有几件重要的事情需要注意:

*   我们将光标从`onsuccess`事件上移开。
*   我们可以通过`value`属性访问光标所指向的对象存储中的文档，这可以从控制台日志中得到验证。
*   我们调用光标上的`continue()`方法来移动到数据库中的下一项。
*   我们需要检查光标是否确实存在。这很重要，因为当我们迭代到最后一个项目，并且调用了`cursor.continue()`时，我们以一个`null`值结束，表明我们已经遍历了存储中的每个文档，我们完成了。在上面的例子中，我们打印出我们已经用完了所有的文档，然后简单地退出事件处理程序。

这是一个相当无聊的例子，所以让我们用游标做一些有趣的事情。

# 选择性地修改我们数据库中的文档

## `update()`

让我们假设我们的供应商 GE 与宝洁公司合并，成为一个新的实体 P&GE。为了使我们的数据库与这次合并保持同步，我们希望更新所有的发票以反映新的名称。

我们将在发票存储上打开一个光标，检查供应商的名称是否是 GE，如果是，则更新为 P&GE。

```
const request = window.indexedDB.open("database", 1);request.onsuccess = () => {
    const db = request.result;
    const transaction = db.transaction(['invoices'], 'readwrite');
    const invStore = transaction.objectStore('invoices');
    const cursorRequest = invStore.openCursor(); cursorRequest.onsuccess = e => {
        const cursor = e.target.result;
        if (cursor) {
            if (cursor.value.vendor === 'GE') {
                const invoice = cursor.value;
                invoice.vendor = 'P&GE';
                const updateRequest = cursor.update(invoice);
            }
            cursor.continue();
        }
    }
};
```

在上面的例子中，当光标遍历商店时，我们检查文档的`vendor`属性，看它是否匹配。如果是这样，我们从光标的`value`键创建一个新的发票文档，更新我们想要的字段，并将这个新文档传递给光标上的`update()`方法。

调用`update()`实际上会返回一个事务请求，所以如果我们想做任何进一步的动作(打印到控制台，调用另一个函数来更新应用程序状态，等等),我们可以在`updateRequest`上放置一个`onsuccess`处理程序。).我们不必要地将请求赋给了一个变量，以表明如果我们选择使用某样东西，它就在那里。

# 从索引中获取游标

我们上面的例子可以用一种稍微不同的方式来实现，这种方式实际上比遍历整个商店更有效。请记住，索引的存在是为了将特定键值都相等的文档组合在一起。因此，在上面，我们可以在一个索引上打开一个游标来定位这个特定的值。

```
const request = window.indexedDB.open("database", 1);request.onsuccess = () => {
    const db = request.result;
    const transaction = db.transaction(['invoices'], 'readwrite');
    const invStore = transaction.objectStore('invoices');
    const vendorIndex = invStore.index('VendorIndex');
    const keyRng = IDBKeyRange.only('GE');
    const cursorRequest = vendorIndex.openCursor(keyRng); cursorRequest.onsuccess = e => {
        const cursor = e.target.result;
        if (cursor) {
            const invoice = cursor.value;
            invoice.vendor = 'P&GE';
            const updateRequest = cursor.update(invoice);

            cursor.continue();
        }
    }
};
```

如您所见，我们做了一些小改动:

*   我们在发票存储上打开了先前定义的供应商索引的光标
*   我们在打开光标时传递了一个可选的`IDBKeyRange`参数，以定位我们想要的供应商。请注意两点:(1)如果我们没有传入这个参数，我们将会遍历整个对象存储，其中的文档按照索引的值按字典顺序排序(而在对象存储上打开的游标是按照其键路径的值排序的)。有许多可能的方法来定义一个键范围；完整列表可在 [MDN](https://developer.mozilla.org/en-US/docs/Web/API/IDBKeyRange) 获得。(2)当我们在对象存储上定义游标时，我们也可以使用键范围。
*   我们不再需要在游标的每次迭代中检查供应商名称，因为我们保证只拥有我们想要更改的文档。

# 同时打开多个游标

游标的一个好处是可以同时打开多个。事实上，如果你想的话，你可以打开无限的号码(并且有足够的存储空间)。

因此，让我们假设我们的供应商 Frigidaire 倒闭了，我们不再需要向他们支付我们的发票。我们的数据库的构造方式意味着我们需要遍历我们的发票库(有选择地只选择 Frigidaire 发票)，对于该库中的每张发票，删除它以及它在发票项目库中的所有行项目。

```
const request = window.indexedDB.open("database", 1);request.onsuccess = () => {
    const db = request.result;
    const transaction = db.transaction(
        ['invoices', 'invoice-items'],
        'readwrite'
    );
    const invStore = transaction.objectStore('invoices');
    const invItemStore = transaction.objectStore('invoice-items'); // Get invoice cursor
    const invoiceCursorRequest = invStore.index('VendorIndex')
        .openCursor(IDBKeyRange.only('Frigidaire')); invoiceCursorRequest.onsuccess = e => {
        const invCursor = e.target.result;
        if (invCursor) {
            // Get invoice item cursor
            const invItemCursorRequest = invItemStore
                .index('InvoiceIndex')
                .openCursor(
                    IDBKeyRange.only(invCursor.value.invoiceId)
                ); invItemCursorRequest.onsuccess = e => {
                const invItemCursor = e.target.result;
                if (invItemCursor) {
                    invItemCursor.delete();
                    invItemCursor.continue();
                }
            }
            invCursor.delete();
            invCursor.continue();
        }
    }
};
```

在本例中，我们将发票项光标嵌套在外部发票光标的`onsuccess`处理程序中，每个处理程序调用其`delete()`方法，这将启动一个删除事务(就像上面的`update()`调用一样)来删除底层文档。我们通过不将`delete()`操作赋给变量来简化代码，因为我们不需要担心操作的任何成功/错误处理。

# 结论

这实质上是使用游标和 IndexedDB。这里的用例比我们在这里讨论的要多得多——比如在我们的文档上计算聚合数据——但是理解这里讨论的概念将使您能够实现数据库系统的所有功能。我将把这个要点作为我们对数据库所做的所有事情的一个工作示例。