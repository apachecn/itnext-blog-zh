# 用节点发送短信。JS、TypeScript 和 Twilio

> 原文：<https://itnext.io/send-sms-with-node-js-typescript-and-twilio-32d1e631d656?source=collection_archive---------1----------------------->

![](img/1b5c9b1f541a1b5808b7d86b71b792c9.png)

我正在做一个项目，根据用户的行为向他们发送短信。该项目利用了 [Firebase 云函数](https://firebase.google.com/docs/functions/)、 [Twilio](https://www.twilio.com/) 和 [TypeScript](https://github.com/Microsoft/TypeScript) 。

您可以使用 Node。JS 与 JavaScript 来完成这个项目。用 TypeScript 做只是我个人的喜好。

我假设你的电脑上有节点和 NPM。如果你没有，你应该去 Node.js 网站下载。

在我们开始之前，你应该有一个 [Twilio](https://www.twilio.com/sms) 账户。之后，您登录到仪表板以检查帐户 SID 和身份验证令牌。

![](img/051c85564fea57d9d759858b7e6c08e3.png)

**第一步:入门**

您需要创建一个 NPM 项目并安装一些依赖项

```
npm init # you know what to do
```

安装特维利奥 NPM 软件包

```
// My current twilio version "^3.15.0"npm install twilio --save// ts-node package for running typescript files
 npm g install ts-node
```

**步骤 2:设置**

创建您发送短信的逻辑

```
// twilio.ts/**
 * Typescript
 * Twilio version: ^3.15.0
 */

import * as Twilio from 'twilio';

// getting ready
const twilioNumber = 'your-twilio-number';
const accountSid = 'AC-something';
const authToken = 'something-something';

const client = new Twilio(accountSid, authToken);

// start sending message

function sendText(){
    const phoneNumbers = [ 'phone-number-1', 'phone-number-2']    

    phoneNumbers.map(phoneNumber => {
        console.log(phoneNumber);

        if ( !validE164(phoneNumber) ) {
            throw new Error('number must be E164 format!')
        }

        const textContent = {
            body: `You have a new sms from Dale Nguyen :)`,
            to: phoneNumber,
            from: twilioNumber
        }

        client.messages.create(textContent)
        .then((message) => console.log(message.to))
    })
}

// Validate E164 format
function validE164(num) {
    return /^\+?[1-9]\d{1,14}$/.test(num)
}
```

在这段代码中，我创建了一个向多个号码发送消息的函数。电话号码分组在名为 **phoneNumbers** 的数组中。它还在发送之前检查电话号码 E164 格式。

配置完成后，您可以通过运行以下命令来测试发送 sms 功能:

```
ts-node twilio.ts
```

* *将在云火基功能中使用该功能的人将获得奖励。您应该将凭证存储在函数配置环境中。*

```
firebase functions:config:set twilio.sid = 'abc'
firebase functions:config:set twilio.token = 'xyz'
```

在您的函数内部，您可以通过调用

```
const accountSid = functions.config().twilio.sid
const authToken = functions.config().twilio.token
```

更多有趣的[云函数 sippets](https://github.com/dalenguyen/firebase-functions-snippets) 请查看我的 github 库。

[https://github.com/dalenguyen/firebase-functions-snippets](https://github.com/dalenguyen/firebase-functions-snippets)