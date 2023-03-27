# Firebase + Algolia 搜索

> 原文：<https://itnext.io/firebase-algolia-6c58fba930b7?source=collection_archive---------4----------------------->

![](img/478943acb5aa435a7366d60095eb0b52.png)

以非凡的速度和索引搜索您的 firestore 数据库

我开始创建一个应用程序来帮助其他人学习基本的[波兰语变体](https://polish-verbs.firebaseapp.com)。我希望这款应用程序具有出色的搜索能力，因为我们学习语言的方式各不相同，我们可能会记住部分单词，甚至是单词的发音。

Algolia 让我能够非常快速地索引我的数据库，它非常强大。Elasticsearch 是一个备选方案，但我想用这个应用程序开发一个真正的“无配置/无服务器”体验。

我创建了一个通过 url 执行的 firebase 函数，该函数将索引我的 firestore 收藏。然后，我使用 algolia 客户端 API 搜索这些索引，同样不必混淆任何重要的 API 或管理密钥，因为没有管理密钥，任何人都无法写入 algolia 索引。

让我们开始吧。

**第一步:设置** [**firebase**](https://firebase.google.com/docs/functions/get-started)

```
$ npm install -g firebase-tool$ firebase login$ mkdir firebase-functions && cd firebase-functions$ firebase init functions
```

遵循 CLI 过程的其余部分，并设置您的 firebase functions 项目。

**第二步:设置您的项目**

在 firebase-functions 项目中，安装并保存 algoliasearch。

```
$ npm i algoliasearch --save
```

**第三步:编写你的函数**

> 注意:您需要从 spark 计划升级到使用外部 API 的功能。

确保您已经设置了 [Algolia](https://www.algolia.com/doc/) 。

```
// functions/index.jsconst functions = require('firebase-functions');
const admin = require('firebase-admin');
const algoliasearch = require('algoliasearch');

const ALGOLIA_APP_ID = 'XXXXXXXX';
const ALGOLIA_ADMIN_KEY = 'XXXXXXXX';
const ALOGOLIA_INDEX_NAME = 'XXXXXXXX';

admin.initializeApp(functions.config().firebase);

exports.firestoreToAlgolia = functions.https.onRequest((req, res) => {
    const arr = [];
    admin.firestore().collection('COLLECTION_NAME').get().then(docs => {
        docs.forEach(doc => {
            const verb = doc.data();
            verb.objectID = doc.id;
            arr.push(verb);
        });

        const client = algoliasearch(ALGOLIA_APP_ID, ALGOLIA_ADMIN_KEY);
        const index =  client.initIndex(ALOGOLIA_INDEX_NAME);

        index.saveObjects(arr, (err, content) => {
            if (err) res.status(500);

            res.status(200).send(content);
        });
    });
});
```

**第四步:部署&执行功能**

现在您已经编写了 firebase 函数，是时候部署和执行了。

```
$ firebase deploy --only functions
```

一旦您成功部署了您的函数，您将会看到类似如下的消息:

```
Function URL (firestoreToAlgolia): [https://us-central1-MY_PROJECT.cloudfunctions.net/](https://us-central1-MY_PROJECT.cloudfunctions.net/addMessage)firestoreToAlgolia
```

或者，您可以在 firebase 项目中的函数链接下找到它的 url。

将该 URL 复制并粘贴到您的浏览器中，几秒钟后，您的数据将被 algolia 搜索编入索引。

**第五步:配置客户端**

我在我的 react 项目中创建了一个 Algolia“类”,如果你愿意，你可以用另一种方式来实现。

```
// utils/algolia.jsimport algoliasearch from 'algoliasearch';

const ALGOLIA_APP_ID = 'XXXXXXXX';
const ALGOLIA_API_KEY = 'XXXXXXXX';

const client = algoliasearch(ALGOLIA_APP_ID, ALGOLIA_API_KEY, {
  protocol: 'https:'
});
const index = client.initIndex('COLLECTION_NAME');

export default index;
```

然后我在我的反应“动词”容器中使用它

```
// containers/verb/index.jsimport React, { useState } from 'react';
import algolia from 'utils/algolia';const getVerbs = (verbs, setVerbs, query) =>
  algolia.search({ query, hitsPerPage: 50 }).then(res => setVerbs(res.hits));const Verbs = ({ history }) => {
  const [loading, setLoading] = useState(true);
  const [verbs, setVerbs] = useState([]);
  const [query, updateQuery] = useState('');
  const [fetching, updateFetch] = useState(true);

  if (fetching)
    getVerbs(verbs, setVerbs, query).then(() => {
      setLoading(false);
      updateFetch(false);
    });

  return (
    <>
      <SearchBar updateQuery={updateQuery} updateFetch={updateFetch} />
    </>
  );
};...
```

就是这样，现在您的应用程序中有了一个极其强大的搜索组件，当添加新数据时，只需执行一个 URL 就可以进行索引。

请留下评论，分享经验！我很想听听你对这篇文章的看法，以及你是否有任何建议与我分享。