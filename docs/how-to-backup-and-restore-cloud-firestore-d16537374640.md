# 如何备份和恢复云 Firestore

> 原文：<https://itnext.io/how-to-backup-and-restore-cloud-firestore-d16537374640?source=collection_archive---------0----------------------->

![](img/7210971ba0ea87db28867e8f9e199fe4.png)

****更新*** :我写了一个 [NPM 包](https://www.npmjs.com/package/firestore-export-import)，帮助你备份和恢复 firestore 数据。想的话可以试试。它还可以帮助备份子集合；)

[](https://www.npmjs.com/package/firestore-export-import) [## firestore-出口-进口

### 用于备份和还原 Firebase Firestore 的 NPM 软件包

www.npmjs.com](https://www.npmjs.com/package/firestore-export-import) 

***更新**:感谢[安德鲁的](https://medium.com/u/43638038564e?source=post_page-----d16537374640--------------------------------)信息。Firebase 只是更新了它们的导入和导出特性。可以在这里阅读:[https://firebase . Google . com/docs/firestore/manage-data/export-import](https://firebase.google.com/docs/firestore/manage-data/export-import)

但是，如果您愿意，您仍然可以使用此软件包:)

**先决条件**

你需要[节点](https://nodejs.org/en/download/)或者可以运行 JAVASCRIPT (JS)文件的东西。

从 Firebase 控制台的*项目设置>服务帐户*中获取 serviceAccount JSON 文件

用自己的 App 初始化时，更改 *databaseURL*

**从 Firestore 导出 JSON 数据**

该脚本将从您的 Firestore 中保存一个收藏，并将其另存为***Firestore-export . JSON***

您可以将该脚本命名为 export.js，并使用以下代码运行它

```
node export.js <your-collection-name>
```

记得在运行脚本之前安装 NPM 软件包

```
var admin = require("firebase-admin");
var fs = require('fs');
var serviceAccount = require("./serviceAccountKey.json");

var collectionName = process.argv[2];

// You should replae databaseURL with your own
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: "https://<your-project-name>.firebaseio.com"
});

var db = admin.firestore();

var data = {};
data[collectionName] = {};

var results = db.collection(collectionName)
.get()
.then(snapshot => {
  snapshot.forEach(doc => {
    data[collectionName][doc.id] = doc.data();
  })
  return data;
})
.catch(error => {
  console.log(error);
})

results.then(dt => {
  // Write collection to JSON file
  fs.writeFile("firestore-export.json", JSON.stringify(dt), function(err) {
      if(err) {
          return console.log(err);
      }
      console.log("The file was saved!");
  });
})
```

**将 JSON 数据导入 Firestore**

这将有助于将单个集合导入到您的 Firestore。这是我将用于导入的示例 JSON 文件

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

该脚本将以“*测试*”作为集合名称，以*“第一密钥”+“第二密钥”*作为文档导入。

您可以使用此命令导入 Firestore

```
node import.js import-to-firestore.json
```

脚本名为 *import.js* ，示例 JSON 文件为*import-to-firestore . JSON*

```
var admin = require("firebase-admin");
var fs = require('fs');
var serviceAccount = require("./serviceAccountKey.json");

var fileName = process.argv[2];

// You should replae databaseURL with your own
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: "https://<your-project-name>.firebaseio.com"
});

var db = admin.firestore();

var collectionName = '';

fs.readFile(fileName, 'utf8', function(err, data){
  if(err){
    return console.log(err);
  }

  // Turn string from file to an Array
  dataArray = JSON.parse(data);

  for(var index in dataArray){
    collectionName = index;
    for(var doc in dataArray[index]){
      if(dataArray[index].hasOwnProperty(doc)){
        db.collection(collectionName).doc(doc)
        .set(dataArray[index][doc])
        .then(() => {
          console.log('Document is successed adding to firestore!');
        })
        .catch(error => {
          console.log(error);
        });
      }
    }
  }

})
```

如果你想看看资源库，请看看我的 Github 账号:[https://github.com/dalenguyen/firestore-import-export](https://github.com/dalenguyen/firestore-import-export)