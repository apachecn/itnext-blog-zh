# Firebase 云功能:验证用户令牌

> 原文：<https://itnext.io/firebase-cloud-functions-verify-users-tokens-d4e60e314d1a?source=collection_archive---------3----------------------->

## 仅向经过身份验证的用户授予对 Firebase 云功能的访问权限。

![](img/002b2380921e71ff43a0f13fa9f45924.png)

奈杰尔·塔迪亚恩多在 [Unsplash](https://unsplash.com/s/photos/you-shall-not-pass?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

我昨天通过声明一个新的 [Firebase 云函数](https://firebase.google.com/docs/functions/)开始了对 [DeckDeckGo](https://deckdeckgo.com) 的核心函数之一的重构，它可以通过 [HTTP 请求](https://firebase.google.com/docs/functions/http-events)来触发。

因为我想保护它的访问，为了避免偷偷摸摸的请求，我按照我之前的一篇博文[在一个人的帮助下保护它。](https://medium.com/better-programming/protect-your-http-firebase-cloud-functions-adf23c45765e)

一旦我测试了这个特性的第一步，我实际上注意到它不是我用例的正确解决方案。我不得不使用用户令牌授予访问权限。

# 验证用户在云函数中的令牌

对于那些知道解决方案的人来说，这可能听起来很傻，但实际上我花了相当多的时间来找到如何在 Firebase Cloud 函数中验证用户的令牌。

我在尝试实现该解决方案时走错了路，因为我分别在后端实现它，如[使用库](https://developers.google.com/identity/sign-in/web/backend-auth) [google-auth-library](https://github.com/googleapis/google-auth-library-nodejs) 向后端服务器验证所示。我花时间实施该解决方案，并寻找在哪里可以找到我的项目所需的 OAuth `CLIENT_ID`信息，最终在我尝试该过程时面临以下错误:

```
No pem found for envelope: {"alg":"RS256","kid":"...","typ":"JWT"}
```

最后，经过多次尝试，我接受了失败，并在谷歌上寻找解决方案。幸运的是，对我来说，在一个 [Stackoverflow 问题](https://stackoverflow.com/questions/61937587/how-to-get-valid-token-from-react-firebase-f%c3%bcr-nodesjs-server-verification/61937783#comment112690479_61937783)的结尾，由于 [Will](https://stackoverflow.com/users/8535518/will) 的回答，我发现有一种更容易验证令牌的方法。

事实上，如果我知道[管理文档](https://firebase.google.com/docs/auth/admin/verify-id-tokens)，我会发现 Firebase 是解决这一需求的内置方法。

> Firebase Admin SDK 有一个用于验证和解码 ID 令牌的内置方法。如果提供的 ID 标记具有正确的格式、未过期并且经过正确签名，则该方法返回解码后的 ID 标记。您可以从解码的令牌中获取用户或设备的 uid。

一旦我发现了这块宝石，一旦我的大脑终于灵光一现，我就能够实现一个小的实用功能:

```
import * as admin from 'firebase-admin';
import * as functions from 'firebase-functions';

export async function verifyToken(
                request: functions.Request): Promise<boolean> {
  try {
    const token: string | undefined = await getToken(request);

    if (!token) {
      return false;
    }

    const payload: admin.auth.DecodedIdToken = 
                   await admin.auth().verifyIdToken(token); return payload !== null;
  } catch (err) {
    return false;
  }
}async function getToken(request: functions.Request): 
                       Promise<string | undefined> {
  if (!request.headers.authorization) {
    return undefined;
  }

  const token: string = 
        request.headers.authorization.replace(/^Bearer\s/, '');

  return token;
}
```

注意，我测试了`payload`是否不是`null`来认为令牌是有效的，但是我认为它可能是不需要的。方法`verifyIdToken`在无效时抛出一个错误。

此外，您还可以注意到，我将用户令牌作为 HTTP 请求的`headers`进行传递，并以关键字`Bearer`为前缀。

例如，给定一个令牌 ID `975dd9f6`，HTTP POST 请求将如下所示:

```
#!/bin/sh
curl -i
     -H "Accept: application/json"
     -H "Authorization: Bearer 975dd9f6"
     -X POST https://us-central1-yolo.cloudfunctions.net/helloWorld
```

# 仅授权非匿名用户

任何人都可以试用 [DeckDeckGo](https://deckdeckgo.com) ，如果你只是想试一试，没有强制登录或预先登录。这对我们来说真的很重要，我们不是在追逐数据或用户数量，我们正在开发一个用于演示的编辑器，供用户使用或不使用😉。

也就是说，如果用户想要公开分享他们的演示文稿，因为我们不想公开发布太多“这是一个测试”或“Yolo”卡片，如果可能的话，分别避免没有意义的公开内容，我们会将我们的“发布流程”(我们将演示文稿作为渐进式网络应用程序在线转换和部署的流程)限制为签名用户。

对于这些进程，我们使用 Firebase 提供的能力来使用匿名用户。

这就是为什么，除了验证令牌，我还添加检查这一信息。幸运的是，这也可以很容易地解决，因为由`verifyToken`函数提供的`payload`确实包含这样的信息。

```
const payload: admin.auth.DecodedIdToken = 
                   await admin.auth().verifyIdToken(token);return payload !== null &&
       payload.firebase.sign_in_provider !== 'anonymous';
```

# 带承载的呼叫功能

如果你感兴趣的话，下面是我如何在 TypeScript 和使用 Firebase Auth 的应用程序中为函数调用提供上面的`bearer`。

```
import * as firebase from 'firebase/app';
import 'firebase/auth';helloWorld(): Promise<void> {
  return new Promise<void>(async (resolve, reject) => {
    try {
      const token: string = 
            await firebase.auth().currentUser.getIdToken(); const functionsUrl: string = 
           'https://us-central1-yolo.cloudfunctions.net'; const rawResponse: Response = 
            await fetch(`${functionsUrl}/helloWorld`, {
        method: 'POST',
        headers: {
          Accept: 'application/json',
          'Content-Type': 'application/json',
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify({
          something: 'a value'
        }),
      });

      if (!rawResponse || !rawResponse.ok) {
        reject('Post failed etc.');
        return;
      }

      resolve();
    } catch (err) {
      reject(err);
    }
  });
}
```

# 顶端的樱桃:CORS

因为我实现了第一个处理 HTTP 请求的函数，所以我必须处理 CORS。快速的谷歌搜索和由 [CoderTonyB](https://github.com/CoderTonyB) 提供的[要点](https://gist.github.com/mediavrog/49c4f809dffea4e00738a7f5e3bbfa59#gistcomment-2585600)提供了一个解决方案。

[expressjs/cors](https://github.com/expressjs/cors) 应安装在功能项目中。

```
npm i cors --save && npm i @types/cors --save-dev
```

最后，在有效实现之前，应该使用一个处理程序来处理 CORS 请求。

```
import * as functions from 'firebase-functions';import * as cors from 'cors';export const helloWorld= functions.https.onRequest(myHelloWorld);async function helloWorld(request: functions.Request,
                          response: functions.Response<any>) {
  const corsHandler = cors({origin: true});

  corsHandler(request, response, async () => {
      response.send('Yolo');
  });
}
```

# 拿走

不用说，错误地开始一个新特性的开发并很快失去时间是很容易的。我很想说深呼吸或者休息一下是关键，但是偶尔会发生一些事情😉。然而，如果你有很棒的技巧和诀窍来避免这种情况，请告诉我，我很想听听这些！

如果你对结果感到好奇，请关注我们的 Twitter，因为我们可能会在下周为开发者发布一个超级酷的功能🚀。

到无限和更远的地方！

大卫