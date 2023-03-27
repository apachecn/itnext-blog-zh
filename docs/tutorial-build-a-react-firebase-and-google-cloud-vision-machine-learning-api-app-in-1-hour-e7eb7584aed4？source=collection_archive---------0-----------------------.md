# 教程:构建一个 React、Firebase 和谷歌云视觉机器学习 API 应用程序

> 原文：<https://itnext.io/tutorial-build-a-react-firebase-and-google-cloud-vision-machine-learning-api-app-in-1-hour-e7eb7584aed4?source=collection_archive---------0----------------------->

我将学习哪些技术/库？

*   Firebase (Firestore、功能、存储、谷歌认证、托管、数据库安全、存储安全)
*   谷歌云视觉机器学习 API
*   使用 Firebase 的谷歌认证
*   反应(创建-反应-应用程序)
*   无服务器功能(使用 Firebase 功能)
*   异步/等待

**我将建造什么？**

演示:【http://doppelganger-app.firebaseapp.com/ 

代号:[https://bitbucket.org/bochrane/doppel-app](https://bitbucket.org/bochrane/doppel-app)

designs:[https://drive . Google . com/file/d/1H _ c 7 ibzg 5 ggvns 1 mppqn 5 tqik 5 w3 CFB/view？usp =分享](https://drive.google.com/file/d/1H_c7IBZG5Ggvns1MmPPQn5Tqik5w3Cfb/view?usp=sharing)

下面的视频展示了我们将要建造的东西

这是您将要构建和部署的内容(29 秒的剪辑)

你将使用 firebase 函数创建一个 react 应用程序，上传你自己的照片，然后返回一个长得像你的人的图像列表——根据谷歌认为你长得像谁:)

**这个教程有多长？**如果你独自编码，1-2 个小时，如果你只是想阅读，大约 20-30 分钟

我刚刚构建了一个企业应用程序，它使用 Firebase 为一家建筑公司预订建筑设备，它所提供的东西给我留下了深刻的印象，我想展示一些 Firebase 在几年前可能需要几个月才能完成的东西。

**注意:如果你希望看到与 Google Cloud Vision API 对话的完整工作的无服务器函数，你需要在教程的后面输入一个有效的信用卡到 Firebase——不要担心，虽然你不会被收费，除非你使用大量的函数调用，我们在本教程中不会接近这些函数调用。**

那么，我们如何构建这个应用程序，这些二重身图像来自哪里？

**下面是我们将要使用的:**

*   Google Cloud Vision API(图像的机器学习 API)将提供一个预先训练的模型，分析我们上传的任何图像，并提供一个与我们上传的图像相似的图像列表——非常整洁！
*   Firebase 函数(是的，我们也将在本教程中学习无服务器函数——如果你还没有遇到它们，我们很快就会知道它们是什么)
*   Firebase Storage(用于存储我们上传的图像)和 Firebase Firestore(Firebase 的新实时非 sql 数据库)
*   为前端应用做出反应
*   Google 登录(使用 Firebase 内置的 Google 认证)

我还包含了草图设计文件和 Invision 可点击原型——我发现使用这两个工具来提前设计和制作我想要构建的原型要容易得多。

![](img/3af0c774d201e601c0121ad1228b5dc8.png)

[草图文件](https://drive.google.com/file/d/1H_c7IBZG5Ggvns1MmPPQn5Tqik5w3Cfb/view?usp=sharing)

![](img/dcebc75ed37351c9e771d9d7d40fd032.png)

[视觉可点击原型](https://projects.invisionapp.com/share/W2JK3G9AZXP#/299468172_My_Doppels)

[](https://projects.invisionapp.com/share/W2JK3G9AZXP#/screens/299468172_My_Doppels) [## 点击此处查看“二重身”项目

### 这个由 InVisionApp 带给你的原型

projects.invisionapp.com](https://projects.invisionapp.com/share/W2JK3G9AZXP#/screens/299468172_My_Doppels) 

因此，在本教程中我们会涉及到很多内容，我很高兴能与您分享这些内容。制作这个应用程序后，我最大的体会是，使用机器学习 API 创建大规模应用程序的能力变得非常简单。现在来看教程…

**我们将把教程分解如下:**

1.  设置 react 应用程序(我们将使用 create-react-app)
2.  配置我们的 Firebase 应用程序
3.  设置 Google 身份验证
4.  设置 Firebase 存储、Firestore 和添加映像
5.  设置 Firebase 函数，允许在我们上传新图像时触发无服务器函数
6.  在 Firebase 函数中设置 Google Vision API，从它的 API 中请求类似的图像
7.  用我们的应用程序测试我们的 Google Vision API
8.  部署到 Firebase 的一个实时工作的应用程序-哇！

# **1。设置 react 应用程序**

首先，确保您已经安装了[节点](https://nodejs.org/en/download/)(至少版本 6)

我们将使用脸书的 [create-react-app](https://github.com/facebook/create-react-app) 来构建 react 应用程序。

```
npm install -g yarn
npx create-react-app my-doppelganger-app
cd my-doppelganger-app
yarn start
```

运行“npm start”后，您应该会看到这个屏幕

![](img/10988ac5588df915e3cdfba09d45ab2e.png)

运行 npm start 后，您应该会看到此屏幕

接下来，在 src 文件夹中创建 3 个文件夹，如下所示:

```
mkdir src/features src/config src/helpers
```

然后让我们为登录、主页和相册设置一些虚拟组件

```
cd featurestouch Login.js Home.js
```

然后在 src/features/Login.js 中添加一个虚拟组件:

**src/features/Login.js**

```
import React, { Component } from 'react';export default class Login extends Component {
 render() {
  return (
   <div>this is my Login component</div>
  );
 }
}
```

对 Home 组件进行同样的操作:

**src/features/Home.js**

```
import React, { Component } from 'react';export default class Home extends Component {
 render() {
  return (
   <div>this is my Home component</div>
  );
 }
}
```

接下来，让我们为应用程序设置路线。首先我们将安装 [react-router](https://github.com/ReactTraining/react-router/tree/master/packages/react-router) 和 [react-router-dom](https://github.com/ReactTraining/react-router/tree/master/packages/react-router-dom) :

```
yarn add react-router react-router-dom
```

在 src/index.js 中，我们将添加它们并设置我们的路线:

**src/index.js**

```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import registerServiceWorker from './registerServiceWorker';import { Redirect, Route, Switch } from 'react-router';
import { Router } from 'react-router-dom';
import createBrowserHistory from 'history/createBrowserHistory';import Login from './features/Login';
import Home from './features/Home';const customHistory = createBrowserHistory();const Root = () => {
 return (
  <Router history={customHistory}>
   <Switch>
    <Route path="/login" component={Login} />
    <Route path="/app/home" component={Home} />
    <Redirect from="/" to="/login" />
   </Switch>
  </Router> 
 )
}ReactDOM.render(<Root />, document.getElementById('root'));
registerServiceWorker();
```

我们现在应该在浏览器中看到登录组件:

![](img/019a31e741d4ba3243df1d49b6815fdc.png)

接下来，我们可以通过手动转到主路由来测试我们的路由工作，以确保它们正常工作:

![](img/5f2894bfab695d1860ffecd42c5c8b28.png)

既然我们的路由已经工作了，我们可以开始设计我们的登录组件了:

我们将[使用这个 png](https://firebasestorage.googleapis.com/v0/b/doppelganger-app.appspot.com/o/google-icon-white.png?alt=media&token=ff891c5f-f2a4-441e-b457-d71b9b21762f) 作为按钮。

我们将把这个 png 文件上传到 Firebase 存储器。因此，首先，我们需要用谷歌帐户登录 firebase，然后创建一个新的 Firebase 应用程序

![](img/f41b6244c033e1b3c63d392da5ee6db9.png)

然后转到左侧导航栏上的“存储”,按“开始”。然后，您将看到您的存储桶的默认安全性规则—下面显示默认安全性是只要用户登录就允许访问任何文档。出于我们的目的，我们想让一些图像允许注销用户查看我们的谷歌登录按钮和背景图像的登录页面。

![](img/3a02bfdebddcb05d1d72de5b40f3a7d1.png)

现在让我们改变这些规则，允许任何人查看任何图像(我们稍后将锁定它)。

在“存储”的“规则”选项卡中，按如下方式更改规则:

```
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write;
    }
  }
}
```

接下来，将按钮图像上传到存储器

![](img/916ac439509697f93adc9055f2c5fc6d.png)

单击存储中的文件，然后按上传文件

接下来，在单击图像行后，通过单击文件位置下拉菜单，复制刚刚上传的图像的下载 URL

![](img/b90022cc739c7a003dddc5e375a237b6.png)

现在，我们可以将将要制作的 div 的背景图像设置为 Google 图像，如下所示:

**src/features/Login.js**

上面，我们已经包含了一个使用 firebaseAuth()的 loginWithGoogle helper 方法。firebase 为我们提供了 signInWithRedirect(googleProvider)方法，传递给该方法的 Google provider 位于 config.constants.js 中，它只是 firebase . auth . Google auth provider()的一个新实例。

当我们使用 localStorage 登录 Google 时，我们还展示了一个简单的加载屏幕。

**index.css**

```
html {
 margin: 0;
 width: 100%;
}body {
  margin: 0;
  padding: 0;
  font-family: sans-serif;
  max-width: 768px;
  margin: 0 auto;
  height: 100vh;
  width: 100%;
}.login-button {
 height: 100px;
 width: 100%;
 position: fixed;
 bottom: 0;
 background: black;
 max-width: 768px;
 margin: 0 auto;
}.login-container { 
 background: #FFC107;
 width: 100%;
 height: 100vh;
}.button-text {
 position: relative;
 font-size: 25px;
 color: white;
 line-height: 50px;
 left: 90px;
}.google-logo {
 height: 50px;
 float: left;
 background-size: contain;
 background-repeat: no-repeat; 
 margin: 25px 20px;
}#root {
 width: 100%;
 height: 100vh;
}
```

![](img/e5f6e391618d9e5a53d53a83ea0f91e2.png)

对于登录组件，我们现在应该看到类似这样的内容

接下来，我们将在归途中创建虚拟数据，以显示我们的 doppelgnger feed 将会是什么样子

首先，我们将把 react-bootstrap 用于我们的网格系统

```
yarn add react-bootstrap
```

然后将以下内容添加到 src/features/Home.js 中

**src/features/Home.js**

在上面的 Home.js 中，你会看到我们添加了 React Bootstrap，包括图像、网格、行和列组件。我们现在还添加了一组虚拟图像，并将其设置为“所有照片”的初始状态。

在我们最终的 return 语句之前，我们使用 build it JavaScript“map”方法遍历我们在初始状态下设置的所有照片。每次我们遍历一张照片时，我们都会创建一个 html 片段，其中动态地包含了这张照片的细节。

并将这个 css 添加到 index.css(这将是我们的应用程序需要的所有 css)

**index.css**

为了让图标和 react-bootstrap 正常工作，我们需要添加材质图标 cdn 和 react-bootstrap css 文件。

**index.html**

```
<link rel="manifest" href="%PUBLIC_URL%/manifest.json">
<link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico">...<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet"><link rel="stylesheet" href="[https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css](https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css)" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">...
```

我们现在应该看到我们的虚拟照片列表以及底部的导航栏，如下所示:

![](img/3077c542d467c3f234cd55cda79a8ca2.png)

添加我们的虚拟图像后，我们应该在/app/home route 中看到这个——不要担心——我们很快会用自己的照片去掉 Phil Murray 的脸

我们的 index.html 现在应该是这样的:

**public/index.html**

# 2.配置我们的 Firebase 应用程序

接下来，让我们在 React 应用程序中设置 Firebase 应用程序。

首先，转到 firebase 控制台，单击左上角导航栏中的“项目概述”。

![](img/cf152775f80dd4ba57ad008effc85d25.png)

您的 Firebase 控制台

然后单击“将 Firebase 添加到您的 web 应用程序”按钮，并复制在模式弹出窗口中弹出的代码。

![](img/b9de7ce5e4dc1358d955e13dc5bf75a7.png)

按“将 Firebase 添加到您的 web 应用程序”将您的 Firebase 详细信息复制并粘贴到 React 应用程序中

在 src/config/constants.js 中创建一个配置文件

**src/config/constants.js**

```
import firebase from 'firebase';const config = {
    apiKey: "AIzaSyBBaJIkRtF3jgzFF7BC83G6nI9joZKxezg",
    authDomain: "doppelganger-app.firebaseapp.com",
    databaseURL: "https://doppelganger-app.firebaseio.com",
    projectId: "doppelganger-app",
    storageBucket: "doppelganger-app.appspot.com",
    messagingSenderId: "115334021919"
};firebase.initializeApp(config);export const googleProvider = new firebase.auth.GoogleAuthProvider();
export const firebaseAuth = firebase.auth;
export const db = firebase.firestore().settings({ timestampsInSnapshots: true });
```

上面，我们使用了 firebase 应用程序的配置细节。我们还添加了来自 firebase 的 Google Auth provider，它将允许我们通过 Google 登录。最后，我们建立了 Firebase 数据库—在这种情况下，我们将使用 Firestore，这是一个新的实时文档存储库。我们在 firestore 设置中添加了一个设置，它将 create_at 时间戳保存为 firestore 时间戳，而不是常规的日期对象。

我们还需要安装 firebase 以及 firebase CLI (firebase-tools)，如下所示(我选择了 Firebase tools 的特定版本，以确保该应用程序为您工作，就像它为我工作一样，但如果您愿意，可以随意使用最新版本):

```
npm install -g firebase-tools@3.17.4
```

我们还将在我们的应用程序中本地安装 firebase:

```
yarn add firebase@5.0.2
```

接下来，让我们在终端中本地设置 firebase:

(注意:我在我的终端上使用了 [Oh-My-Zsh](https://github.com/robbyrussell/oh-my-zsh) ，这使得语法更好，我在 Mac 上的标准终端应用程序上使用了绿色主题，只需进入终端- >首选项- >选择草地，然后点击默认使用该主题)

```
firebase login
```

![](img/822cd2364db4387f3dedb1b66e3dc2b5.png)

然后你会在一个新的浏览器中看到一个谷歌登录弹出窗口——继续使用你注册 Firebase 时使用的 gmail 帐户进行验证。

一旦您通过终端登录，我们可以设置此应用程序为 Firebase 应用程序，如下所示:

```
firebase init
```

系统会提示您回答几个问题，请使用这些答案:

1.  你会看到 5 个选项，勾选我们想要的，按空格键和上下移动到我们想要的选项。按空格键为 Firestore，功能，主机和存储(我们将不需要数据库，因为这是旧的实时数据库，我们将使用 Firestore 数据库)

![](img/3cab9e027c580d89c09fbffe57bb0fe4.png)

在上面选择的选项上按空格键来选择它们。

2.假设我们已经登录，我们现在可以在下一个问题的列表中找到我们在 Firebase 中创建的项目:

![](img/cd5b6d8049d0ae65544a47333c850b94.png)

向下箭头指向 Firebase 中您想要使用的项目，然后按 Enter 键

3.按 Enter 键接受以下步骤的默认设置(下图看起来相似，但对于我们的 Firebase 应用程序来说是不同的设置问题:

![](img/d20dbe01c39b4dc2189776c91406cf22.png)

按 Enter 键接受此默认值

![](img/ec0eb9f9ff6630f16b310f9235e4f6f2.png)

按 Enter 键接受此默认值

![](img/c7df1811a16f0590963c65fb7fde8a37.png)![](img/40f3164f17727b397b91b625b5e54450.png)

我不会使用 ESLint，所以我会选择 n 并按回车键，但您也可以按 Y 并按回车键来使用它

![](img/f94ecc64fe0a06003038afd354bdefe9.png)

y 并回车安装软件包

![](img/0372c95a3bad8bec27b1096c68811630.png)

键入 build 将我们的应用程序发送到一个名为 build 的目录，firebase 将为我们部署这个目录

![](img/526f4d8b2657ab7a55f200a2da55a2cd.png)

按 Enter 键接受默认值

![](img/e739069752f0a053f308f0372daffff9.png)

按 Enter 键接受默认值

![](img/3ed3762ae6a749831b2c9f27f2991cd8.png)

我们被设置为 Firebase 应用程序！

最后，我们被设置为 Firebase 应用程序——哇哦！

你会注意到添加了许多新的文件和文件夹。这些文件的简要概述如下:

a.。firebaserc 文件——这允许 firebase 知道使用哪个应用程序，并将给出一个类似 default 的名称，该名称将指向您在 firebase 控制台中命名的 firebase 应用程序

b.firebase.json —这使 firebase 知道在哪里可以找到我们可以在应用程序中定义的规则文件，我们创建的数据库索引(firebase 预先索引我们的数据库以使过滤更快—我们将在本教程的后面介绍这一点)以及 firebase 将部署到 web 的构建文件夹的位置—使用 create-react-app，我们稍后将运行 npm run build 的一个命令，该命令将使用 webpack 生成一个构建文件夹。

c.firestore . indexes . JSON——如前所述，这是我们定义数据库索引的地方，这样可以更快地过滤各种预索引的集合

d.storage . rules——我们已经在线更改了 firebase 控制台中的存储规则，但是一旦我们部署了该应用程序，它们将被该文件中的存储规则覆盖，因此现在让我们更改该文件以反映我们在 firebase 控制台中所做的事情。只需确保您的 storage.rules 文件如下所示:

**存储规则**

```
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write;
    }
  }
}
```

e.您还会看到一个名为 functions 的新文件夹。这就是我们无服务器功能的发展方向——激动人心的时刻！我们将在后面的教程中讨论这些。

这是新的文件和文件夹，现在我们将设置 Firebase 存储，以便保存一些图像。

重要提示:在我们继续之前，如果您使用 Github 或 Bitbucket 等 git 源代码控件来保存您的工作，请确保您将/functions/node_modules 添加到您的。gitignore 文件如下:

**。gitignore**

```
...# dependencies
/node_modules
**/functions/node_modules**# testing
/coverage...
```

# 3.设置 Google 身份验证

接下来，我们将设置能够使用 Firebase 的内置[谷歌认证提供商](https://firebase.google.com/docs/auth/web/google-signin)登录和注销我们的应用程序。

首先，我们将设置我们的 auth.js 助手文件:

```
touch src/helpers/auth.js
```

**src/helpers/auth.js**

```
import { firebaseAuth, googleProvider } from '../config/constants';export function loginWithGoogle() {
 return firebaseAuth().signInWithRedirect(googleProvider);
}export function logout() {
 return firebaseAuth().signOut();
}
```

上面的 auth.js 文件将创建两个登录 Firebase 的 Google auth provider 和注销的方法。

接下来，我们将更新 Login.js 文件，如下所示:

**src/features/Login.js**

```
import React, { Component } from 'react';
import { loginWithGoogle } from '../helpers/auth';
import { firebaseAuth } from '../config/constants';const firebaseAuthKey = 'firebaseAuthInProgress';
const appTokenKey = 'appToken';export default class Login extends Component {constructor(props) {
        super(props);
        this.state = { splashScreen: false };
        this.handleGoogleLogin = this.handleGoogleLogin.bind(this);
    }handleGoogleLogin() {
     loginWithGoogle()
     .catch(err => {
      localStorage.removeItem(firebaseAuthKey)
     });// this will set the splashscreen until its overridden by the real firebaseAuthKey
     localStorage.setItem(firebaseAuthKey, '1');
    }componentWillMount() {// checks if we are logged in, if we are go to the home route
        if (localStorage.getItem(appTokenKey)) {
            this.props.history.push('/app/home');
            return;
        }firebaseAuth().onAuthStateChanged(user => {
         if (user) {
          localStorage.removeItem(firebaseAuthKey);
          localStorage.setItem(appTokenKey, user.uid);
          this.props.history.push('/app/home')
         }
        })
    }render() {if (localStorage.getItem(firebaseAuthKey) === '1') 
   return <Splashscreen />;
  return <LoginPage handleGoogleLogin={this.handleGoogleLogin} />;}
}// this is the URL we copied from firebase storage
const loginButtonUrl = '[https://firebasestorage.googleapis.com/v0/b/doppelganger-app.appspot.com/o/google-icon-white.png?alt=media&token=ff891c5f-f2a4-441e-b457-d71b9b21762f'](https://firebasestorage.googleapis.com/v0/b/doppelganger-app.appspot.com/o/google-icon-white.png?alt=media&token=ff891c5f-f2a4-441e-b457-d71b9b21762f');const styles = {
 backgroundImage: `url(${loginButtonUrl})`
}const LoginPage = ({ handleGoogleLogin }) => (<div className="login-container">
  <div onClick={handleGoogleLogin} className="login-button">
   <div style={styles} className="google-logo">
    <span className="button-text">Sign In With Google</span>
   </div>
  </div>
 </div>)const Splashscreen = () => (<p>Please Wait Loading...</p>);
```

我们在这里添加了很多代码，所以让我们一步步来。

*   我们导入了 loginWithGoogle helper 方法
*   在我们的构造函数中，我们最初隐藏了闪屏，我们还绑定了 handleGoogleLogin 函数，这样我们就可以在组件中使用它，而不需要从组件内部绑定它
*   handleGoogleLogin 函数调用我们导入的预构建的 loginWithGoogle 函数(我们对成功结果不感兴趣，因为无论如何它都会成功地转到主页，所以我们现在只是捕捉错误)
*   我们使用 componentDidMount React 生命周期方法来监听组件何时挂载，因为我们需要知道是否有登录的用户。我们还使用 Firebase 内置的 onAuthStateChanged 方法，该方法检查页面重新加载等情况何时发生，并返回当前登录的用户
*   我们使用 localStorage 来检查登录是否正在加载，在这种情况下显示闪屏组件，否则显示登录组件
*   我们为 LoginPage 使用了一个功能组件，并且我们还向该组件传递了一个参数，这是 handleGoogleLogin 方法，当我们单击 Login 按钮时，它可以使用该方法

在我们对此进行测试之前，我们需要转到 Firebase 控制台，打开 Google 作为应用程序的身份验证选项。

如果您已经在应用程序中尝试了 Google auth，但它卡在闪屏上，请进入浏览器的控制台，删除刚刚设置的 localStorage 项目:

**在你的浏览器控制台上运行这个，如果它卡在闪屏上:**

```
localStorage.removeItem("firebaseAuthInProgress")
```

在 Firebase 控制台中，转到“身份验证”,然后单击“设置登录方法”

Firebase 控制台>身份验证>登录方法> Google

![](img/35d1ee5f5c7fda3d357660a605688a34.png)

允许身份验证方法的 Firebase 身份验证区域

然后点击谷歌并将启用切换到开，然后**按保存**

![](img/a366f7e94685c426bf2938d8e8a89bf8.png)

在 Firebase 的身份验证区域启用 Google auth

你现在应该在按下你的谷歌登录按钮时得到一个谷歌认证屏幕——哇哦！

现在让我们也开始工作吧。

在 src/features/Home.js 中，在整个代码中添加以下内容:

**src/features/Home.js**

```
...
import { logout } from '../helpers/auth';
...// inside of the constructor
constructor() { ...
   this.handleLogout = this.handleLogout.bind(this);
   ...}handleLogout() {
        logout()
        .then(() => {
            localStorage.removeItem(appTokenKey);
            this.props.history.push("/login");
            console.log("user signed out from firebase");            
        }.bind(this));
} // then in the logout icon (the final Col in the Grid of icons) add a logout method like this:...
<Col onClick={this.handleLogout} xs={4} className="col-bottom">
          <i className="bottom-icon material-icons">assignment_return</i>
</Col>
...
```

所以 Home.js 应该是这样的:

```
import React, { Component } from 'react';
import { Image, Grid, Row, Col } from 'react-bootstrap';
import { Link } from 'react-router-dom';
import { logout } from '../helpers/auth';const appTokenKey = "appToken";
export default class Home extends Component {constructor(props) {
        super(props);const allPhotos = [
         {
          id: 'randomstringimadeup43454356546',
          url: '[http://fillmurray.com/200/200'](http://fillmurray.com/200/200')
         },
         {
          id: 'randomstringimadeup43523526534565',
          url: '[http://fillmurray.com/200/200'](http://fillmurray.com/200/200')
         },
         {
          id: 'randomstringimadeup433245234534',
          url: '[http://fillmurray.com/200/200'](http://fillmurray.com/200/200')
         }                  
        ]this.state = {
          allPhotos
        };this.handleLogout = this.handleLogout.bind(this);}handleLogout() {
        logout()
        .then(() => {
            localStorage.removeItem(appTokenKey);
            this.props.history.push("/login");
            console.log("user signed out from firebase");            
        });
    }render() {// our doppelganger images
        const allImages = this.state.allPhotos.map(photo => {return (
            <div key={photo.id}>
              <div style={{minHeight: '215px'}}>
                <i className="bottom-icon material-icons main-close">close</i>
                <Image style={{ width: '100%' }} src={photo.url} responsive />
              </div>
            </div>
          );
        })return (
   <div>
    {allImages}<Grid className="bottom-nav">
      <Row className="show-grid">
        <Col xs={4} className="col-bottom">
            <Link to="/app/album"><i className="bottom-icon material-icons">collections</i></Link>
        </Col>
        <Col xs={4} className="col-bottom">
            <i className="bottom-icon material-icons">camera_alt</i>
        </Col>
        <Col onClick={this.handleLogout} xs={4} className="col-bottom">
          <i className="bottom-icon material-icons">assignment_return</i>
        </Col>
      </Row>
    </Grid></div>
  );
 }
}
```

现在你应该可以使用 Google auth 登录和注销你的应用了——干得好——你知道吗…

![](img/76d8d15b030b1d954236a8d3f4070830.png)

菲尔·默里是这么说的:)

# 4.设置 Firebase 存储、Firestore 和添加映像

在这一节中，我们将底部导航栏中的中间按钮，当按下时，会提示我们上传图像，该图像将保存到 Firebase 存储中。我们走吧！

我们现在将使用一个名为[react-firebase-file-uploader](https://github.com/fris-fruitig/react-firebase-file-uploader)的文件上传器包，所以让我们先安装它:

```
yarn add react-firebase-file-uploader
```

一旦我们安装好了，把文件上传器 JSX 添加到 Home.js，你的中间导航栏按钮的位置如下:

**文件上传者 JSX 片段**

我们还将创建“handleUploadSuccess”函数(我们可以看到，当文件成功上传时，该函数在上面的 FileUploader 代码片段中执行。它将返回刚刚上传到 Firebase Storage 的图像的一系列详细信息，我们将把这个图像 url 保存到我们即将在 Firebase Firestore 数据库(也称为文档存储)中创建的名为“照片”的新集合中。

handleUploadSuccess 函数将如下所示:

**handleUploadSuccess.js**

经过上面的 2 处修改后，我们的 Home.js 文件现在应该是这样的:

**src/features/Home.js**

让我们解释一下“handleUploadSuccess”方法(或函数——无论你想怎么称呼它)以及文件上传器 JSX 片段。

1.  HandleUploadSuccess

*   handleUploadSuccess 方法是从 FileUploader jsx 代码片段上名为“onUploadSuccess”的 prop 中启动的
*   我们使用 ES7 async-await 语法，这是一种更好的承诺方式。现在，我们可以在函数前面加上“async”这个词，在我们通常想要添加的任何东西前面加上“await”这个词，而不是“then-ing”和“catch-ing”。然后“到。这对于清理我们的代码，而不是到处都是承诺是很棒的
*   有了 async-await 语法，我们现在可以使用 try-catch 块来检查错误。try 块包装了我们所有的 async-await 代码，如果任何一段 await 代码有错误，它将把这个错误和发生的错误一起扔进 catch 块，在这个例子中我称之为“err”
*   我还使用了来自 ES6 的[析构语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)，它允许我提取一个对象的特定部分，你可以看到我在上面的代码片段中使用它来获得**桶**和**完整路径**值。(存储桶和完整路径是照片上的元数据，描述照片在我们的存储桶中的位置，这对于了解从哪里访问我们的照片以在我们的应用程序中显示照片非常重要)
*   有了照片信息和用户信息，我们就可以创建一个“新照片”对象来保存到我们的 Firestore 集合中，这个集合叫做“照片”

**需要采取的行动:**

为了让我们的应用程序运行并将这张新照片不仅保存到 Firestore 存储中，还将其作为条目添加到我们数据库中的照片集合中，我们需要从 Firebase 控制台设置 Firestore，如下所示:

转到 Firebase 控制台>数据库

![](img/72590e8f87e226828d9b6c7618a63eed.png)

单击左边的云 Firestore Beta 入门(实时数据库是 Firebase 的旧文档存储，Firestore 在查询文档的速度和能力方面有了显著的改进)

将询问您想要什么样的数据库安全设置—选择“在测试模式下启动”,然后按“启用”。这将允许未经认证的用户进行读写操作——但鉴于这是一个测试应用程序，我们现在就让它保持原样。

![](img/c1e496facd2aff7cba50a73f48f97b99.png)

现在我们已经启用了数据库，让我们通过按下底部导航中的中间按钮来测试上传照片到应用程序。

如果成功，您应该会看到一些控制台日志，并且能够查看 Firebase 中的存储和数据库，看到如下内容:

浏览器控制台日志:

![](img/93fedb6debc02b8cf235ab215f047230.png)

如果您保留了上面示例片段中的控制台日志，您可能会看到这些控制台日志，显示我们成功地将照片添加到存储中，并在我们的照片数据库列表中使用了该照片的 url

存储:

![](img/debe53c1202f611e13a5a4202230b67a.png)

您应该可以在存储区的列表中看到您刚刚上传的照片的名称

Firestore 照片系列:

![](img/c8cefc438096b9b8f7eb291e80494a32.png)

您应该能够在 Firestore 中看到您添加到照片收藏中的新照片的详细信息

干得好！

本节的最后一部分是获取我们保存在数据库中的照片列表，并在屏幕上显示它们。我们目前正在使用一组模拟图像(感谢 Phil Murray——但你在我们的应用程序中的时间到了)。

现在，让我们使用 componentDidMount React 生命周期方法从我们的 Firebase Firestore 照片集合中获取图像。

为了获得一组初始照片，我们将使用。get 方法(稍后对于实时更新，我们也将包括 Firestore 的 onSnapshot 方法)

**componentDidMount 方法:**

现在，我们的应用程序应该能够显示我们像这样上传的图像(在重新加载后，当然这是我们接下来要修复的):

![](img/3eaf01c307b91b8d6441454d083792f2.png)

到目前为止，这是我们的应用程序，我们可以上传图像，但仍然需要刷新才能看到它们，接下来我们来解决这个问题

您会注意到，在我们的 componentDidMount 方法中，我们使用了另一个名为 getInitial 的方法，它将为我们获取所有的照片。getInitial 的好处在于它是一个倾听者，而不是一次性获取照片。因此，每当我们对照片集合进行更改时，它也会自动更新我们的前端照片列表。我们正在使用 Firestore 的内置 [**快照**](https://firebase.google.com/docs/firestore/query-data/listen?authuser=2) 方法来允许实时更新——哇！

您还会注意到，在 getInitial 方法中，我们确保在请求照片之前有一个登录的用户。我们使用内置的 Firebase auth 方法[**onAuthStateChanged**](https://firebase.google.com/docs/auth/web/manage-users)来确保有人登录。

尝试添加另一个——这一次，一旦它到达数据库，它应该会出现在应用程序中——很好！

干得好！我们就要有一个可以工作的二重身应用程序了。现在让我们让无服务器功能工作起来，然后将它们连接到 Google Vision API——这样我们就完成了。

您的 Home.js 文件现在应该如下所示:

**src/features/Home.js**

这就是我们的 src/features/Home.js 现在的样子

# 5.设置 Firebase 函数，允许在我们上传新图像时触发无服务器函数

我们现在将使用无服务器功能，允许我们编写服务器端代码，这些代码将连接到 Google Vision API，以检查谁看起来像我们。

如果你还没有了解无服务器运动，https://serverless.com/learn/[的一些优秀的初学者可以帮助你开始。](https://serverless.com/learn/)

简而言之，无服务器运动不仅仅是编写无服务器功能，它还允许开发人员更多地关注业务逻辑，而不是基础设施和设置。无服务器是一个有点误导的名字，因为仍然有服务器支持你的应用。只是作为一名开发人员，很多设置和扩展工作都要处理好(无论如何，在早期——无服务器有很多功能，但也有自己隐藏的缺陷，如监控和冷启动，这些功能每天都在改进)。

我喜欢把无服务器函数想象成我们可能编写的任何后端函数。只是这些功能都有自己的小服务器，等待运行。

当您运行这些函数时，它们会旋转自己的各种迷你容器，通常持续几毫秒到 6 秒，有时甚至更长，这取决于您的函数。一旦您完成了该函数的执行，微型容器就会关闭。您需要为使用的内存量和毫秒执行时间付费。你可能会听到人们谈论 Lambda 的无服务器功能，因为 AWS Lambda 是第一批让无服务器功能成为开发人员主流的产品之一(尽管不是第一个)。

Firebase 函数是另一个使用谷歌云平台的无服务器函数。

好了，关于无服务器函数已经说得够多了，本，让我们只写一些…

等等，为什么我们又要写一个无服务器函数？

因为我们希望与 Google Vision API 进行对话，以提供一个图像列表，该列表看起来像我们上传到照片集合的任何图像。

所以我们开始吧…

**消防基地功能**

如果你在主应用程序中还看不到函数文件夹，那么让我们从 firebase 函数文档[https://firebase.google.com/docs/functions/get-started](https://firebase.google.com/docs/functions/get-started)中为你设置一个如下的函数文件夹(如果你在本教程的早些时候就开始学习并在设置中要求函数，这将不会适用)

```
firebase init functions
```

接下来，我们应该会看到一个名为 **functions** 的文件夹，里面有一个名为 **index.js** 的文件

我们在这个 index.js 文件中还看不到太多东西，但是在这里我们可以编写所有将部署到 Google 云平台的无服务器函数。

我们的应用程序有一个 package.json。我们的函数文件夹也有自己的 package.json，它也是独立的，是我们将要编写的服务器端代码需要的节点模块。让我们设置无服务器功能所需的节点模块:

```
cd functions
```

接下来，添加必要的节点模块，包括[谷歌云视觉 API 节点模块](https://github.com/googleapis/nodejs-vision)

```
yarn add firebase-functions firebase-admin promise [@google](http://twitter.com/google)-cloud/vision
```

然后安装所有已有的模块和新模块:

```
yarn
```

我们将添加[**promise**](https://github.com/then/promise)**库，因为所有的函数都要求它们被 promise 化(那是一个我发誓的词？).**

**然后在 functions/index.js 中，我们将设置我们的无服务器函数文件:**

****functions/index.js****

**上面，我们看到我们已经导入了刚刚安装的模块，还启动了我们的 Firestore 数据库，我们将需要更新从 Google Vision API 中获取的图像。**

**接下来，让我们编写一个名为 **addSimilarImages** 的函数，它将把来自 Google Vision API 的相似图像添加到一个图像 URL 列表中，我们将根据刚刚保存在 Firestore 中的照片保存该列表。**

**Firestore 有一个很棒的特性，允许监听数据库事件。下面，您将通过**functions . firestore . document(' photos/{ document } ')
看到它的使用。onCreate** 方法如下:**

****functions/index.js** (带设置和功能的完整版)**

**您可以在上面看到以下内容:**

*   **Firestore 使用了一个 **onCreate** 监听器，它允许在向收藏中添加新照片这样的事件发生时调用一个函数**
*   **该语法允许我们通过添加。文档('照片/{文档}**
*   **快照返回添加的数据的快照，在本例中是添加到数据库的照片文档，上下文让我们知道使用了什么类型的方法，例如创建、读取、更新和销毁等元数据**
*   **我们通过重新创建一个名为 **photoUrl** 的 Google 存储风格的 url 来创建一个 Google Vision API 能够理解的完整的照片 url**
*   **然后，我们在使用谷歌视觉实例(visionClient)并使用 **webDetection** 方法后返回一个承诺(在这个 API 上有很多方法，比如 labelDetection、faceDetection，你可以在这里找到一些你可能喜欢使用的方法，如 https://cloud.google.com/vision/docs/all-samples**
*   **在本教程中，我们使用的是 webDetection 方法，该方法拾取与上传的图片具有相似属性的图片(我们称之为**图片链接**)**
*   **我们将这个图片 Url 通过 Vision API，它返回一个图片 URL 列表——我们对视觉上相似的图片感兴趣**
*   **我们创建这些视觉上相似的图像的数组，并用**相似图像**图像更新照片集合中的文档**

****注意:**在上面的功能为我们工作之前，我们现在需要在我们的帐户上启用 Google Cloud API**

# ****6。在 Firebase 函数中设置 Google Vision API，从其 API 中请求类似的图像****

**现在我们正在进入真正令人兴奋的部分，我们将使用机器学习服务来帮助我们找出世界上还有谁看起来和我们相似，反正据谷歌称。**

****第一步:在这里进入谷歌云控制台**[https://console.cloud.google.com/](https://console.cloud.google.com/)并登录(不要担心——你已经有一个帐户——你只需要使用与你的 Firebase 帐户相同的 gmail 帐户登录)**

****第二步:点击选择一个项目，然后点击新建项目****

**![](img/4a78ec527d0c873c5e71422ef9b0afa7.png)**

****第三步:命名你的项目(我刚刚选择了和我的 firebase 项目相同的名字，但是如果不同也没关系)****

**![](img/77ebe711569acc7467bf38af573dcb1c.png)**

****第四步:选择你刚刚做的项目(这里有个窍门)****

**一旦您创建了项目，我们将再次回到主控制台页面，但是在左上方，我们刚刚创建的项目没有被选中，因此我们需要选择它。**

**您可能认为这很简单，但是如果您已经有了以前的项目，单击选择一个项目可能不会在列表中显示您新创建的项目—您需要单击“最近的项目”选项卡旁边的**所有项目**选项卡来找到它。**

**![](img/cea60bc028fe7c0c14ae592e715d7330.png)**

**我们可能需要跳转到 All 选项卡来找到我们刚刚创建的项目**

****第五步:启用谷歌视觉应用编程接口****

**转到 API 和服务仪表板—从控制台转到 API 和服务>仪表板，如下所示**

**![](img/7c43d262034e0e229b00b598188a4f57.png)**

**然后单击启用 API 和服务，如下所示**

**![](img/e786dba689a9b4347c476a9c8891df4d.png)**

**然后在下面这个页面的搜索中输入“视觉”**

**![](img/5d2ac1dc1402bbdf1b957af20cb2aefb.png)**

**并单击 Vision API**

**![](img/002709389e619525cd4112b10a47ce23.png)**

**最后，点击启用，如下所示**

**![](img/308b3615529d3d0853f055e49ee79099.png)**

**哇——这话有点道理！我们现在准备开始实际使用一个预先训练好的机器学习 API——哇！**

# **7.用我们的应用程序测试我们的 Google Vision API，并将相似的图像添加到我们的应用程序中**

**这就是我们为之奋斗的时刻，无论你在公交车上、火车上、家里的沙发上还是在工作中读到这篇文章有多长时间了:)**

**为了测试这是否有效，我们需要上传一张图片，然后查看 Firebase 函数的日志，看看我们是否成功了。您会注意到，我有意在函数中留下了一堆控制台日志，以便我们可以看到我们正在获取哪些数据。**

**我们走吧！**

**首先，我们需要将我们的函数部署到 firebase，以便它能够识别我们刚刚编写的函数。我们还不会部署前端，只部署我们的服务器端代码。**

****部署 Firebase 功能:****

**从终端中应用程序的主目录:**

```
firebase deploy --only functions
```

**这将**只**部署功能文件夹。**

**![](img/b8bc30f0b3de07f47575c27ace15f328.png)**

**希望你能看到这样的确认:**

**![](img/42f75935a29dd2581006a55a3d84865d.png)**

**有时我会得到一个错误，就像这个图片的顶部，我能说的就是尝试，再试一次:)**

**如果您遇到任何错误，只需确保您使用 firebase 注销功能登录到 firebase，然后登录 firebase(然后使用正确的 gmail 帐户登录)并运行 firebase use default(在您的。firebaserc 文件——很可能您没有更改这个文件，所以您应该在那里看到一个默认的密钥。还要确保您已经在 functions 文件夹中运行了 yarn，因为该模块必须在部署到 Firebase 之前安装。如果你得到一个像上面图片顶部的错误，就再试一次。魔法。**

****允许云视觉工作需要添加计费****

**正如我在教程开始时提到的，如果你想看到你的函数工作，我们需要在你的 Firebase 控制台中包含一个有效的信用卡。不过不用担心，在谷歌开始向你收费之前，免费层已经有了巨大的月使用量。你可以点击查看 [Firebase 的定价，点击](https://firebase.google.com/pricing/)查看 [Cloud Vision 的定价。**简而言之，我们需要每天进行超过 12.5 万次函数调用，或者托管超过 5GB 的照片，或者每天对照片集进行超过 2 万次写操作，或者每月在谷歌云视觉上分析超过 1000 张图像，才能开始收费。**](https://cloud.google.com/vision/#cloud-vision-api-pricing)**

**在 Firebase 控制台的左下方，单击“升级”,然后选择“随用随付”计划。**

**![](img/1774cf0b7984117d533053d75fb4db07.png)****![](img/0cda9fa4d4497c54d3e190bba0db62b5.png)**

**最后，让我们进入 Firebase 控制台中的功能页面，检查一切是否正常…**

**![](img/a37900b664895ad02de12be72dca619b.png)**

**我们部署的第一个无服务器功能。不错！**

**你可以在上面看到，我们称为 addSimilarImages 的函数现在在 firebase 控制台的函数列表中。**

**太好了，接下来让我们通过上传图像来测试我们的功能，然后我们将检查这个无服务器功能的日志，看看发生了什么…**

**我刚上传了一张照片。现在，让我们在 Firebase 功能控制台中单击查看日志，如下所示:**

**![](img/022dbd024ae8a2e23c625a41147de9c1.png)**

**在右侧查看日志**

**如果我们的照片上传一切顺利，在您的功能日志中，您应该会看到如下内容:**

**![](img/e8b71c8f52d1791b1d3c5fee0fa15be5.png)**

**你可以在上面看到，该函数执行了 5553 毫秒，并从谷歌云视觉 API 获取了一系列类似的图像——如果你仔细查看上面的日志，谷歌会建议我看起来像 Stripe 团队中一个叫 Edwin 的人！**

**教程的最后一部分是在我们的应用程序的前端显示这些类似的图像。让我们看看根据谷歌我们看起来像谁…**

**首先，我们将添加一些加载微调器，让用户知道我们正在等待一些二重身图像从 Google Vision API 返回。我们将使用 react-spinners:**

**从应用程序的根目录，在您的终端中运行:**

```
yarn add react-spinners
```

**然后，我们将在 Home.js 的顶部导入它，并从 react-bootstrap 导入模型。当有人点击二重身的图像时，我们将使用模态，这将打开一个包含该图像的模态。**

```
...import { Image, Grid, Row, Col, Modal } from 'react-bootstrap';...import { ScaleLoader } from 'react-spinners';
```

**我们将会做一些小的改动，下面我会一一介绍。更改结束时，Home.js 文件应该是这样的:**

****src/features/Home.js****

**让我们浏览一下我们添加的内容:**

*   **我们已经添加了一行加载器，以显示我们何时刚刚上传了一张图像，并等待二重身图像从 Google Cloud Vision 返回。我们遍历一个“similarImages”数组，该数组附加到我们之前创建的 index.js 中 firebase 函数的 photo 文档中**
*   **我们现在还在每个有相似图像的图像下面添加了一个二重身列表，并且我们使用了一些 css 来为这些图像创建一个水平滚动条**
*   **我们还添加了一个模态，这样当一个二重身图像被点击时，我们可以看到图像的放大视图**
*   **我们在每张图片的左上方还有一个删除处理程序，允许我们删除每张图片**
*   **我们在图片列表中添加了一个 where 语句，这样我们只能看到上传的图片。在 firebase 中，我们可以在前端使用。集合调用中的 where('userId '，' == '，user.uid)。在下一节中，我们还将在后端添加 Firestore 规则，这将增加复制前端内容的安全性。**
*   **我们还添加了一个小助手功能来检查我们是否在移动设备上，因为我还没有解决从移动设备上传时的旋转问题——解决方法是将您的设备转向横向，这样当在桌面上查看图像时，图像可以正确显示**
*   **我们还添加了一个 imageRef 状态——这对以后在 Firebase 存储上设置安全规则很重要——它允许我们将用户上传的图像匹配到特定的文件位置**

**我们现在有了一个可用的应用程序——至少在本地。让我们把这个东西部署到网络上，完成这个教程！**

# ****8。部署到 Firebase 的一个实时工作的应用程序-哇！****

**最后一部分—部署！**

****第一步:在我们部署之前添加安全规则****

**在您的 firestore.rules 文件中，我们将添加以下内容，以确保只有您可以读取、写入和删除您的照片。**

****火焰杯****

****存储规则****

**每当我们上传一张图片时，我们保存路径以包含用户的 auth id(又名 userId)。**

**如果我们试图在未登录的情况下请求上传图像，或者另一个已登录的用户试图上传引用您的用户 Id 的照片，它会拒绝上传，如下所示:**

**![](img/82641e4f7855e01bc3a421360f0ce669.png)**

**上面的规则是实现基于属性的访问控制的一种很好的方式——简单地说，能够定义允许用户做什么类型的操作(创建、读取等),以及在什么集合中(例如照片集合)。这些规则也是功能性的，即您可以在 service cloud.firestore 块外部创建功能，也可以在内部引用它们。**

**出于我们的目的，我们在上面说过，只有照片集合被授予任何访问权限，对于照片集合中的任何照片，如果您的 firebase 身份验证的用户 Id 与我们保存的照片文档的用户 id 属性相同，我们将允许读取或删除。**

**类似地，对于写入对象，只有当您的用户 id 与 photos collection 文档有效负载上的 user id 属性相同时，您才能将文档保存到数据库。我觉得很不错。**

****第二步:创建 react 应用的构建版本****

**在您的终端中，运行:**

```
yarn build
```

**如果构建正确，您应该会看到如下消息:**

**![](img/580872bcf13e9158a4c4d50176602cf2.png)**

******重要******

**在应用程序根目录下的 firebase.json 文件中，我们将告诉 firebase 在哪个文件夹中找到 react 应用程序。您会注意到已经创建了一个构建文件夹来存放我们的 react 应用程序，因此我们需要将" **public** "更改为" **build** ",如下所示:**

****firebase.json****

```
{
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "hosting": {
    "**public**": "**build**",
    "rewrites": [ {
      "source": "**",
      "destination": "/index.html"
    }],    
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ]
  },
  "storage": {
    "rules": "storage.rules"
  }
}
```

**步骤 2:将完整的应用程序部署到 Firebase**

**在您的终端中，运行:**

```
firebase deploy
```

**这将部署您的前端应用程序以及 firebase functions 文件夹。**

**运气好的话，你的应用程序已经部署好了，而且你还有一个可以运行的二重身应用程序！呜！**

 **[## 重身幽灵应用——React、Firebase、GoogleCloudVision

### 重身幽灵应用程序——查看谷歌认为你长得像谁

doppelganger-app.firebaseapp.com](https://doppelganger-app.firebaseapp.com)** 

**干得好，走到了这一步！我们已经谈了很多。**

**总而言之，你刚刚学会了如何:**

*   **创建一个 react 应用程序并使用 react-router-dom**
*   **Firebase Firestore 数据库和 Firebase 存储的安全规则**
*   **react 和 firebase 应用程序的部署**
*   **通过谷歌云视觉与机器学习 API 进行交互**
*   **使用 Firebase 函数与 Google Cloud Vision API 对话的无服务器函数**
*   **最重要的是……**
*   **你已经知道谷歌认为你长得像谁了！**

**请留下任何意见，问题和任何你想在本教程中看到的改进。**

**如有任何问题，请随时[通过 LinkedIn 联系我](https://au.linkedin.com/in/benjamincochrane)**