# 使用 IndexedDB 开始使用持久脱机存储

> 原文：<https://itnext.io/getting-started-with-persistent-offline-storage-with-indexeddb-1af66727246c?source=collection_archive---------3----------------------->

![](img/0fa6784e6b96d0f405dacf3513e06a2c.png)

# 简介:寻找选择

随着 web 技术的进步，创建离线 web 应用程序的能力越来越接近现实。我写过一些小的在线 web 应用程序，但从来没有像桌面应用程序那样通过浏览器访问的。所以我决定研究离线存储选项，看看最适合在我想要使用的实际应用程序下创建一个数据库层。

在我的研究中，我发现了以下选择

*   局部存储器
*   应用程序缓存
*   Web SQL
*   索引 b

在这四种存储机制中，我不得不放弃本地存储，因为它只是一种键值存储机制，不太适合需要数据库功能的应用程序。应用程序缓存和 Web SQL 是不推荐使用的 API，我不想冒险在应用程序中使用它们，只是为了删除它们。

所以左 IndexedDB。让我们深入研究这个 API，看看我们能用它做些什么。

# 从 IndexedDB 开始

IndexedDB 是一个异步事务数据库系统，专门为存储大量结构化内容而设计。这很好，因为离线应用程序需要将所有内容存储在浏览器中。根据应用程序的不同，除了文本之外，我们还可能存储大文件(例如，pdf、图像)。

然而，IndexedDB 并不像 SQL 那样以基于表的列格式存储数据，而是存储在对象存储中，对象存储中存放着由键索引的 JavaScript 对象。这意味着您不必像创建 MySQL 那样担心创建预定义的模式，尽管您仍然需要考虑如何存储和组织数据，以便从数据库中高效检索。

因此，让我们创建一个小型数据库来保存发票。我们可能希望存储的对象类别有:

*   发票(元数据)
*   发票项目(发票上的行项目)
*   发票附件(用于发票、提货单等的扫描件。)

现在让我们来创建它！

# 实施索引 b

为了第一次开始使用 IndexedDB，我们需要运行以下代码:

```
const request = window.indexedDB.open("database", 1);// Create schema
request.onupgradeneeded = event => {
    const db = event.target.result;

    const invoiceStore = db.createObjectStore(
        "invoices",
        { keyPath: "invoiceId" }
    );
    invoiceStore.createIndex("VendorIndex", "vendor"); const itemStore = db.createObjectStore(
        "invoice-items",
        { keyPath: [ "invoiceId", "row" ] }
    );
    itemStore.createIndex("InvoiceIndex", "invoiceId"); const fileStore = db.createObjectStore(
        "attachments",
        { autoIncrement: true }
    );
    fileStore.createIndex("InvoiceIndex", "invoiceId");    
};
```

在第一行中，我们开始请求打开名为`database`的数据库版本 1。由于这是第一次加载数据库，浏览器将找不到具有该名称的数据库，因此请求将触发`onupgradeneeded`事件。我们希望创建一个事件处理程序来处理我们创建新数据库的逻辑。

我们从触发的事件中获取结果，该事件是一个表示数据库连接的对象，并将它存储在一个名为`db`的变量中。然后，我们在数据库中创建三个对象存储和三个索引。

对`.createObjectStore()`的每个调用都有两个参数:一个名称和一个可选的 options 对象。我们正在创建名为`invoices`、`invoice-items`和`attachments`的商店。但是，我们为每一个都传入了不同的 options 对象，所以让我们来看一下每一个。

在第一个例子中，我们简单地说，在我们将要存储的每个对象中都有一个名为`invoiceId`的属性，数据库应该将这个属性与对象本身配对(这样，当我们查找一个对象时，我们只需要说“get by `invoiceId`”。然而，键路径可以是复合的，所以在第二个 we 中，`invoiceId`和`row`将作为检索对象的键。最后，对于最后一个商店，通过只传递`autoincrement: true`，我们说没有关键路径，数据库应该为我们生成一个。

每个对象存储可以有选择地为一个或多个属性创建索引，这允许除了使用键路径之外的另一种检索对象的方法。每个`.createIndex()`调用都有一个索引名，以及应该索引什么属性。

最后要注意的一点是:如果用户再次访问站点并执行相同的连接请求，除非数据库版本增加到更大的整数(在我们的例子中是 2 或更大)，否则不会触发`onupgradeneeded`事件。

# 实现 CRUD 操作

所有积垢操作的工艺流程如下:

*   打开数据库连接
*   开始交易
*   指示要使用哪个对象存储
*   对该商店执行操作
*   打扫

## 创造

要在数据库中创建新的发票，我们需要执行以下操作:

```
const request = window.indexedDB.open("database", 1);
request.onsuccess = () => {
    const db = request.result;
    const transaction = db.transaction(
        [ "invoices", "invoice-items" ],
        "readwrite"
    );
    const invStore = transaction.objectStore("invoices");
    const itemStore = transaction.objectStore("invoice-items");

    // Add data
    invStore.add(
        { invoiceId: "123", vendor: "Whirlpool", paid: false }
    );
    itemStore.add({
        invoiceId: "123",
        row: "1",
        item: "Dish washer",
        cost: 1400
    });
    itemStore.add({
        invoiceId: "123",
        row: "2",
        item: "Labor",
        cost: 500
    }); // Clean up: close connection
    transaction.oncomplete = () => {
        db.close();
    };
};
```

如果打开数据库的请求成功，那么`onsuccess`事件处理程序将执行事务。我们首先将数据库分配给变量`db`，然后在该数据库上打开一个事务，指定它是一个`readwrite`操作。`db.transaction()`的第一个参数是我们希望事务能够看到的对象存储列表。在我们的例子中，因为有两种类型的数据来表示一张发票，所以我们使用了`invoices`和`invoice-items`存储。

然后我们访问每个对象存储和数据。一旦事务成功完成，我们就关闭数据库连接。

## 更新

为了更新，我们执行相同的开始步骤(连接到数据库、创建事务和访问对象存储)。假设我们想修改洗碗机系列产品的成本:

```
// ...open database, initiate transaction
const itemStore = objectStore.put({
    invoiceId: "123",
    row: "1",
    item: "Dish washer",
    cost: 1300
});
// ...clean up
```

该操作将确定键路径(按照上面的模式定义)，并更新该对象。对象本身被在`.put()`中指定的对象所覆盖，所以要确保包含所有的原始字段和原始数据，这样文档才不会丢失任何信息。

对于具有显式定义的键路径的对象存储，没有必要将键作为第二个参数添加。如果我们让数据库自动生成一个键，我们只需要指定这个键。因此，如果我们想要更新我们的`attachments`存储中的一个扫描文件，我们会想要执行

```
attachmentStore.put({ new data}, KEY_NUMBER);
```

还有，关于`.put()`的最后一点:它有双重功能。如果我们的对象存储中没有发票行`123`和行`1`，这个方法就会为我们创建它(充当上面的`.add()`)。

## 删除

如果我们想要删除发票上的第二行项目，我们所要做的就是指定我们想要从`invoice-items`商店中删除的密钥:

```
itemStore.delete([ "123", "2" ]);
```

## 阅读

从数据库中读取数据最不像表单中的其他三个操作，因为为了使数据可用，我们必须实现另一个事件处理程序。让我们获取之前添加到数据库中的发票:

```
// ...open database, initiate transaction
const getRequest = invStore.get("123");
getRequest.onsuccess = () => {
    // Do something with the data
    console.log(getRequest.result);
};
```

为了获得从我们的对象存储返回的数据，我们必须创建一个`onsuccess`事件处理程序。从数据库中提取数据并准备好进行处理后，将触发此事件。由于事务的异步性，有必要使用事件处理程序；我们不希望后面的代码行在数据还不存在的时候就假设数据已经准备好了。

我们有什么方法可以处理这种情况？我们可以在`onsuccess`处理程序的外部范围内创建一个变量:

```
let data;const getRequest = invStore.get("123");
getRequest.onsuccess = () => {
    data= getRequest.result;
};
```

我们仍然要担心这样一个事实，即`getData`在函数调用中使用之前可能没有分配给它的数据。因此，虽然它可以工作，但并不理想。

我处理这个问题的方法是在 get 请求逻辑周围创建一个包装器函数，它在`onsuccess`处理程序中执行一个回调。

```
const getFromDB(key, callbackFn) {
    const request = window.indexedDB.open("database", 1);
    request.onsuccess = () => {
        const db = request.result;
        const transaction = db.transaction("invoices", "readwrite");
        const invStore = transaction.objectStore("invoices"); const get = invStore.get(key);
        get.onsuccess = () => {
            callbackFn(get.result);
        };
    };
};
```

通过这种方式，您可以封装处理这些数据所需的所有业务逻辑，并且由于 IndexedDB 事务的异步特性，您可以确保正确地处理这些数据。

关于从 IndexedDB 数据库读取数据，还有一点需要说明:我们也可以使用我们在对象存储上创建的任何索引来完成这项工作。

例如，如果我们想获得某个特定供应商的所有发票的列表，我们可以查询索引:

```
// ...open database, initiate transaction
const invStore = transaction.objectStore("invoices");
const invIndex = invStore.index("VendorIndex");
const getRequest = invIndex.getAll("Whirlpool");getRequest.onsuccess = () => {
    // Do something with the data
    console.log(getRequest.result);
};
```

我们只需访问对象存储中声明的索引，并运行`.getAll("Whirlpool")`从 Whirlpool 中提取所有发票。我们也可以使用`.get("Whirlpool")`，但是这只会返回一个找到的匹配。

# 结论

我们简要介绍了如何初始化一个新的数据库，并在其上执行基本的 CRUD 操作。IndexedDB 的内容比本文所能给出的要多得多(只要浏览一下 [MDN 的文档](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)就能发现它有这么多的用处)，随着我了解得越来越多，我希望能写更多关于它们的文章。我将把[这个要点](https://gist.github.com/ryandabler/41c458871cb6b64f599fb182ba88daba)作为基本 IndexedDB 用法的完整示例。