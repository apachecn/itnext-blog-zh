# 使用 Firebase 函数— HTTP 请求

> 原文：<https://itnext.io/working-with-firebase-functions-http-request-22fd1ab644d3?source=collection_archive---------0----------------------->

![](img/4c161f1fc512a9a6dfbb7a64de7de297.png)

如果您不熟悉 Firebase 函数，请阅读[函数入门文档](https://firebase.google.com/docs/functions/get-started)。我有机会玩一会儿 HTTP 请求和 Firebase 函数，有几个小技巧我想分享一下。

1.  **如何启动 HTTP 请求功能**

首先在 index.js 文件中声明它

```
exports.request = functions.https.onRequest((request, response) => {
  // ...
});
```

通过从浏览器、方法或使用 HTTP 服务直接调用该函数来触发该函数。有一篇关于[如何调度 HTTP 请求到你的 firebase 函数](https://medium.com/google-cloud/startups-now-cronjob-your-google-firebase-functions-securely-and-externally-without-paying-for-503b0f4ca539)的非常好的文章值得一读。

2.**如何检查请求方法**

Firebase 函数支持 GET、POST、PUT、DELETE 和 OPTIONS 方法，并且您可以检查哪种方法触发了您的函数。

```
// Check for POST request
if(request.method !== "POST"){
 res.status(400).send('Please send a POST request');
 return;
}
```

3.**如何从 POST 请求中获取数据**

数据(例如 JSON 类型)将在请求的头中。

```
let data = request.body;
```

4.**如何从你的应用/网站发送帖子请求**

这是我的 Angular 应用程序中的一个例子，它发送 POST 请求来触发 firebase 函数

```
// Using Angular HttpClient to send POST requestconst httpOptions = {
 headers: new HttpHeaders({
   'Content-Type':  'application/json',
   'Authorization': 'secret-key'
 })
};let data = {name: 'Dale Nguyen'};this.http.post('https://us-central1-dale-nguyen.cloudfunctions.net/request', data, httpOptions)
.subscribe(
 res => {
   console.log(res);
 },
 err => {
   console.log("Error occured");
 }
)
```

5.**为您的功能增加额外安全性的方法**

感谢上面关于 HTTP 请求时间表的帖子，这是一些可以为你的应用程序增加额外安全性的方法。

为了安全起见，您需要创建一个唯一的密钥

```
npm install -g crypto
node -e "console.log(require('crypto').randomBytes(20).toString('hex'))"
```

然后将密钥添加到 firebase 配置中

```
functions:config:set app_name.key='secret-key'
```

使用密钥验证 app 的方式可以和上面的文章类似，直接在 firebase server 上对比就可以检查了。

```
// POST headers before sending to firebase serverconst httpOptions = {
 headers: new HttpHeaders({
   'Content-Type':  'application/json',
   'Authorization': 'secret-key'
 })
};// firebase functions checklet key = functions.config().app_name.key;let request_key = request.get('authorization');if(key === request_key){
 console.log('Awesome!!!')
} else {
 response.status(400).send('You shall not pass!!!');
 return;
}
```

6.**处理 CORS—‘访问控制-允许-来源’**

感谢 [StackOverFlow](https://stackoverflow.com/questions/42755131/enabling-cors-in-cloud-functions-for-firebase) ，这很容易处理。

在导入 CORS 之前，我们需要先安装它。

```
npm install cors // Thanks John Theo for remind this step :)
```

然后把它放到 index.js 文件中

```
// Add CORS to your index.js
const cors = require('cors')({origin: true});// Put this line to your function
// Automatically allow cross-origin requests
cors(req, res, () => {});
```

**7。发送带参数的 GET 请求(Angular)**

```
/*
 * Send GET request with parameters from Angular
 * https://angular.io/api/http/RequestOptionsArgs
 */

import { Http, Headers } from '@angular/http';

// Prepare the header 
let headers: Headers = new Headers();
headers.set('parameter-name' , 'parameter-value);

// Send request with parameters            
this.http.get('url', {
  headers: headers
}).subscribe(res => resolve(res.json())); 
```

为了从 header 中获取参数，您可以在 firebase 函数中使用这段代码

```
// In order get the request value 
let params = req.headers['parameter-name'];
```

目前就这些。遇到另一个有趣的问题我会更新；)

请查看我的 github，了解更多关于 firebase 函数的有趣片段。

[](https://github.com/dalenguyen/firebase-functions-snippets) [## GitHub-dalen guyen/firebase-Functions-snippets:不可思议的 Firebase 函数片段…

### 这里包含了我所有涉及到编写 firebase 函数的作品，为了使用它，你应该知道如何…

github.com](https://github.com/dalenguyen/firebase-functions-snippets) 

[**在 Twitter 上关注我**](https://twitter.com/dale_nguyen) 获取 Angular、JavaScript & WebDevelopment 的最新内容👐