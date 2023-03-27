# 修复 Nightwatch.js 中的“expect”断言

> 原文：<https://itnext.io/fixing-expect-assertions-in-nightwatch-js-a84a244467b6?source=collection_archive---------2----------------------->

![](img/b6c8e9967270154142c49574540cb335.png)

我是 Nightwatch.js 进行功能测试的忠实粉丝，但是有一点一直让我很恼火，那就是 expect API。我更喜欢使用 expect 而不是 assert，但是因为 Nightwatch 在幕后使用 Chai.js，所以它继承了 Chai 在属性访问方面的蹩脚断言。如果你不知道为什么断言属性访问不好，[读一下这个](https://github.com/moll/js-must#asserting-on-property-access)。

使用以下代码检查登录页面元素，并填写表单:

```
login.expect.element('@username').to.be.visible;
login.setValue('@username', username);

login.expect.element('@password').to.be.visible;
login.setValue('@password', password);

login.expect.element('@submit').to.be.visible;
login.click('@submit');
```

毫无疑问，如果你的 IDE 中有任何形式的林挺代码，你会看到各种红色的曲线和警告。事实证明，这很容易解决，尽管使用一个名为 [dirty-chai](https://github.com/prodatakey/dirty-chai) 的包。也许你已经用它进行单元测试，但是不知道如何把它转移到 Nightwatch 中。

首先，您需要安装 dirty-chai。

```
npm install --save-dev dirty-chai
```

然后在你的 Nightwatch 配置文件中添加一个全局文件(如果你还没有的话)。

```
/* nightwatch.conf.js */module.exports = {
    /* ... */ globals: './globals.js' /* ... */
}
```

然后用 dirty-chai 扩展 Nightwatchs 自定义 Chai 库。在 node_modules 中已经有了 chai-nightwatch，因为它是 nightwatch 的一个依赖项。

```
/* globals.js */module.exports = {

   before: **function** (done) {
      **var** chai = require('chai-nightwatch');
      chai.use(require('dirty-chai'));
      done();
   }

};
```

现在，所有 expect API 断言都将是函数，而不是属性访问器！最初的测试片段现在看起来像这样:

```
login.expect.element('@username').to.be.visible();
login.setValue('@username', username);

login.expect.element('@password').to.be.visible();
login.setValue('@password', password);

login.expect.element('@submit').to.be.visible();
login.click('@submit');
```

轻松搞定！