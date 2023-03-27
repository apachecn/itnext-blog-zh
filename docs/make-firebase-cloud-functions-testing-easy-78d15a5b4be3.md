# 让 Firebase 云功能测试变得简单

> 原文：<https://itnext.io/make-firebase-cloud-functions-testing-easy-78d15a5b4be3?source=collection_archive---------2----------------------->

![](img/4c161f1fc512a9a6dfbb7a64de7de297.png)

[Firebase 云功能](https://firebase.google.com/docs/functions/)是自动响应由 Firebase 功能和 HTTPS 请求触发的事件的绝佳工具。

有几种方法可以测试 firebase 的功能，而不必在每次需要检查时都部署它。这需要很多时间。坏事可能会发生，诸如此类…

合适的方式是局部测试。你可以在 firebase 网站上阅读关于[如何本地测试 firebase 功能](https://firebase.google.com/docs/functions/local-emulator)的 google 官方指南。

```
// Spoiler alert. You will run this command firebase server # to emulate both HTTP functions and hosting
```

另一种方法是自己安装 [Firebase Admin SDK](https://firebase.google.com/docs/admin/setup) 。我会用 TypeScript 来做——我喜欢 TypeScript:)

可以在 [my github](https://github.com/dalenguyen/firebase-functions-snippets/blob/master/local-test-easy.js) 上查看 JavaScript 版本。

你需要做的第一件事是启动一个项目

```
npm init # You what to do next
```

安装 **firebase-admin** npm 包

```
npm install g ts-node # For running typescript files
npm install firebase-admin --save
```

创建您的 **index.ts** 文件

```
/**
 * Typescript version (index.ts)
 */ 

import * as admin from 'firebase-admin';
import { serviceAccount } from './serviceAccountKey';

admin.initializeApp({
    credential: admin.credential.cert({
        projectId: serviceAccount.project_id,
        clientEmail: serviceAccount.client_email,
        privateKey: serviceAccount.private_key
    }),
    databaseURL: serviceAccount.database_url
})
```

将您的 **serviceAccountKey.json** 编辑为 **serviceAccountKey.ts**

```
// serviceAccountKey.ts

export const serviceAccount = {
  "project_id": "your-project-id",  
  "private_key": "-----BEGIN PRIVATE KEY-----blah-blah-blah=\n-----END PRIVATE KEY-----\n",
  "client_email": "dalenguyen@.iam.gserviceaccount.com",  
  "database_url": "https://dalenguyen.firebaseio.com"
}
```

您可以从项目设置中的**服务帐户**选项卡生成新的私钥。

![](img/348d2b09d94c27601a2c803cdf35dbc1.png)

设置完成后，现在您可以通过运行以下命令在本地测试您的功能

```
ts-node index.ts 
```

更具体地说，您可以使用**console . log(' any ')**来测试您的方法。它会显示在你的终端上。

— — — — — — — — — — — — — — — — — — — — — — — —

请检查 Firebase 函数片段的存储库。我会经常更新它；)

[https://github.com/dalenguyen/firebase-functions-snippets](https://github.com/dalenguyen/firebase-functions-snippets)