# Firebase Functions Helper —一种改进的云 Firestore 使用方法

> 原文：<https://itnext.io/firebase-functions-helper-an-improved-method-to-work-with-cloud-firestore-9b5d116452cc?source=collection_archive---------1----------------------->

![](img/7210971ba0ea87db28867e8f9e199fe4.png)

你正在阅读这篇文章，这意味着你知道什么是云 Firestore。如果你不知道，请阅读[Google Firestore Docs](https://firebase.google.com/docs/firestore/)——它基本上只是一个云托管的 NoSQL 数据库。

老实说，我真的很喜欢使用 Firestore，尤其是通过实时监听器跨客户端应用程序同步的能力，甚至是对移动和网络的离线支持。*请注意，在撰写本文时，Cloud Firestore 目前处于测试版。*

虽然我喜欢与 Firestore 合作，但谷歌团队需要改进的功能很少，例如[备份和恢复 firestore](https://github.com/dalenguyen/firestore-backup-restore) 。实时数据库已经实现了这一点。更重要的是，每当我需要做一些事情时，这个查询就有点重复我自己，我不喜欢重复我自己。

所以我写了一个 NPM 包，减少了代码行，改进了它们所缺乏的特性。

[](https://www.npmjs.com/package/firebase-functions-helper) [## 消防基地-功能-助手

### 用于 Firebase 云函数的辅助 NPM 包

www.npmjs.com](https://www.npmjs.com/package/firebase-functions-helper) 

请记住，该项目仍处于开发过程中。如果要在生产中使用这个，风险自担。

让我们看看如何从 Firestore docs 获取文档

```
var docRef = db.collection("cities").doc("SF");

docRef.get().then(function(doc) {
    if (doc.exists) {
        console.log("Document data:", doc.data());
    } else {
        // doc.data() will be undefined in this case
        console.log("No such document!");
    }
}).catch(function(error) {
    console.log("Error getting document:", error);
});
```

我想当你想得到另一份文件时，你必须重复所有的事情。现在让我们来看看从 [firebase functions helper](https://github.com/dalenguyen/firebase-functions-helper) 获取文档的过程

```
firebaseHelper.firestore
.getDocument(db, 'collection-name', 'document-id')
.then(doc => console.log(doc));
```

这在我的情况下要干净得多，我已经为你做了所有的捕捉工作；)

如果你仍然对这个项目感兴趣。请尝试一下，报告错误并在[问题跟踪器](https://github.com/dalenguyen/firebase-functions-helper/issues)中提出功能请求，派生并创建拉式请求！

1.  **安装 Firebase 函数助手**

```
npm install firebase-functions-helper
```

2.**初始化 Firebase 应用程序**

```
const firebaseHelper = require('firebase-functions-helper'); 
const serviceAccount = require('./serviceAccountKey.json'); // Initialize Firebase App 
const app = firebaseHelper.initializeApp(serviceAccount, databaseURL); // Get Firestore DB 
const db = app.firestore;
```

准备好 firestore 后，您可以对该包做很多事情，例如:

***2.1 获取 Firestore 收藏与子收藏***

```
// Start exporting your collection 
firebaseHelper.firestore
.backup(db, 'collection-name', 'sub-collection-optional')
.then(data => console.log(JSON.stringify(data)))
```

有了这个特性，您可以从一个集合中获取并遍历所有文档

```
firebaseHelper.firestore
.backup(db, 'collection-name')
.then(data => {         
  let docs = data['collection-name'];     
  for (const key in docs) {         
     if (docs.hasOwnProperty(key)) {                         
       console.log('Doc id: ', key);             
       console.log('Document data: ', docs[key])                             
     }     
  } 
})
```

***2.2 导入数据到 firestore***

```
// Start exporting your data 
firebaseHelper.firestore.restore(db, 'your-file-path.json');
```

JSON 的格式如下。集合名称为**测试**。**第一键**和**第二键**为文件 id。

```
{
  "test" : {
    "first-key" : {
      "email"   : "dungnq@itbox4vn.com",
      "website" : "dalenguyen.me",
      "custom"  : {
        "firstName" : "Dale",
        "lastName"  : "Nguyen"
      }
    },
    "second-key" : {
      "email"   : "test@dalenguyen.me",
      "website" : "google.com",
      "custom"  : {
        "firstName" : "Harry",
        "lastName"  : "Potter"
      }
    }
  }
}
```

***2.3 在 firestore 中创建 id 为的文档***

```
firebaseHelper.firestore
.createDocumentWithID(db, 'collection-name', 'document-id', data);
```

***2.4 新建一个没有 ID 的文档***

```
firebaseHelper.firestore
.creatNewDocument(db, 'collection-name', data);
```

**2.5*更新一个文档***

```
let data = {key: value}; 
firebaseHelper.firestore
.updateDocument(db, 'collection-name', 'document-id', data);
```

***2.6 删除文档***

```
firebaseHelper.firestore
.deleteDocument(db, 'collection-name', 'document-id');
```

***2.7 检查单据是否存在***

这将返回一个承诺<boolean></boolean>

```
firebaseHelper.firestore
.checkDocumentExists(db, 'collection-name', 'document-id')
.then(exists => console.log(exists));
```

***2.8 从 firestore 查询数据***

这将返回一份承诺<array>的文件</array>

```
// Search for data ( <, <=, ==, >, or >= ) 
const queryArray = ['website', '==', 'dalenguyen.me']; firebaseHelper.firestore
.queryData(db, 'collection-name', queryArray)
.then(docs => console.log(docs));
```

如果你对此感兴趣，请查看我的 github 资源库的更新:[https://github.com/dalenguyen/firebase-functions-helper](https://github.com/dalenguyen/firebase-functions-helper)