# 使用 auth.nuxtjs.org 自定义社交授权流

> 原文：<https://itnext.io/custom-social-auth-flow-with-auth-nuxtjs-org-daa836676587?source=collection_archive---------0----------------------->

![](img/52f8a96ac4cfe66aebe351bf41aab439.png)

[来源](https://entwickler.de/wp-content/uploads/2018/01/nuxt-900x450.png)

nuxt-auth 模块可以很好地处理本地和社交媒体登录，但是当你试图用本地 API 令牌交换社交登录令牌时，事情就变得糟糕了。nuxt 验证模块不支持社交登录的自定义验证流。实际上，nuxt-auth 做的是，它让你登录社交网络，但不允许你用自己的令牌交换社交令牌。因此，我创建了这个教程，使事情变得简单。

所以下面是保持清洁的方法。首先，从我的 github 克隆基础应用。这是我之前关于 nuxt-auth 模块的教程。[查看我之前关于 nuxt-auth 的文章，获得下载项目的详细介绍。](https://medium.com/@rama41222/basic-authentication-using-auth-nuxt-js-e140859ab4c3)

我使用 epic-spinners 来加载动画。您可以使用以下命令安装 epic spinners。

```
npm install --save epic-spinners
```

然后在 plugins 文件夹中创建一个名为 spinners.js 的插件，将 epic spinners 包含到您的项目中。

```
import Vue from 'vue'
import { SemipolarSpinner } from 'epic-spinners'Vue.component('semipolar-spinner', SemipolarSpinner)
```

然后将插件包含到 nuxt.config.js 中

```
plugins: ['~plugins/spinners.js'],
```

## 让我们从授权流程开始

这是我们将要做的。首先，我们让 nuxt-auth 插件登录一个社交网络。然后，我们将从 facebook、google 或任何其他平台获取回调收到的令牌。然后，我们会将该令牌发送到我们自己的服务器，并用本地令牌交换社交登录令牌。然后我们将在 nuxt-auth 模块中强制设置本地 authtoken。这样做之后，我们必须将策略更改为 local，最后您可以从自己的 API 获取本地用户。为了简洁起见，我们将为 axios 创建一个插件，并在 axios 插件中包含所有的标题。然后我们将把插件包含到项目中。

## 第一

您应该更改 nuxt.auth 配置以适应您的授权流。我是这样做的。

```
auth: {
  strategies: {
    local: {
      endpoints: {
        login: {url: '/user/login', method: 'post', propertyName:      'token' },
        logout: **false**,
        user: {url: '/user/user', method: 'get', propertyName: 'data'},
      },
      tokenRequired: **true**,
      tokenType: 'Bearer'
    },
    facebook: {
      client_id: '',
      userinfo_endpoint: **false**,
      scope: ['public_profile', 'email'],
      redirect_uri:'http://localhost:3000/callback'
    },
    google: {
      client_id: '',
      user:**false**,
      redirect_uri:'http://localhost:3000/callback'

    },
  },
  redirect: {
    login: '/?login=1',
    logout: '/',
  }
},
```

然后添加 API 端点作为基础

```
axios: {
     baseURL:'[https://aaa.com/api/v1'](https://aaa.com/api/v1')
  },
```

我有目的地删除了配置的其余部分。完整配置请参考 github。

## 第二

让我们设置 axios。在这里，我将创建一个插件来将所有的标题保存在一个地方。首先，在插件中创建一个名为 axios.js 的文件，并将其包含在 nuxt.config.js 中。

```
plugins: [‘~plugins/axios.js’,’~plugins/spinners.js’]
```

然后将以下代码放入插件文件夹中的 axios.js。

```
export default function ({$axios, redirect}) {
    $axios.onRequest(config => {
        config.headers['Content-Type'] = 'application/json';
        config.headers['Access-Control-Allow-Origin'] = "*";
    })
}
```

如果你需要更多的标题，只需将它们包含在 axios 插件中，axios 会在发送请求时自动插入标题。

现在我们已经设置了 axios。让我们创建自定义身份验证插件。在 plugins 文件夹中创建一个名为 auth.js 的新文件，并键入以下代码。

```
export default async function ({ app }) {
    if (!app.$auth.loggedIn) {
        return
    }const auth = app.$auth;const authStrategy = auth.strategy.name;
    if(authStrategy === 'facebook' || authStrategy === 'google'){
        const token = auth.getToken(authStrategy).substr(7)
        const authStrategyConverted = authStrategy === 'facebook' ? 'fb' : 'google';
        const url = `/user/signup/${authStrategyConverted}?token=${token}`;try {
            const {data} = await app.$axios.$post(url, null);
            auth.setToken('local', "Bearer "+ data.access_token);
            setTimeout( async () => {
                auth.setStrategy('local');
                setTimeout( async () => {
                    await auth.fetchUser();
                })
            });
        } catch (e) {
            console.log(e);
        }
    }
}
```

如您所知，nuxt-auth 模块将登录状态保存在 auth.loggedIn 变量中。因此，首先我们必须检查用户是否已经登录。如果没有登录，这个方法应该退出。记住我之前告诉你的，我们应该在介入之前让 nuxt-auth 模块登录。然后，我们将把记录的状态复制到一个变量中，这样我们就不必在代码中重复$nuxt.auth。

然后我进入策略。这很重要，因为我们正在定制社交媒体认证流程。您可以通过 auth.strategy.name (app)从 nuxt-auth 访问当前记录的策略。$auth.strategy.name)。我们这样做，只是为了允许社交登录。然后通过 auth.getToken(authStrategy)从 nuxt-auth 获取保存的社交登录访问令牌。substr(7)。这里 auth.getToken('策略名')是内置于 nuxt-auth 的默认方法。有关更多信息，请参考文档。

```
const authStrategyConverted = authStrategy === 'facebook' ? 'fb' : 'google';
const url = `/user/signup/${authStrategyConverted}?token=${token}`;
```

使用上面的代码，我正在创建基于策略的 url。然后向您的端点发送 post 调用以检索自定义令牌，并通过 auth.setToken('strategy '，token)方法将本地令牌设置为 Vuex。您的 API 必须获取社交令牌，检索相关用户信息，并发布访问令牌。这里的策略应该是本地化的。然后使用 auth.setStrategy(' strategy ')方法将策略更改为 local。

最后将这个插件全局包含到 nuxtjs 中。

```
plugins: ['~plugins/auth.js','~plugins/axios.js','~plugins/spinners.js']
```

这里是回购的 github 链接。本教程到此结束，❤

[GITHUB 回购](https://github.com/rama41222/NuxtMedium.git)

```
git clone [https://github.com/rama41222/NuxtMedium.git](https://github.com/rama41222/NuxtMedium.git)
```