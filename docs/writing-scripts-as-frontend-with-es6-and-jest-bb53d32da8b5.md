# 用 Es6 和 Jest 编写脚本作为前端

> 原文：<https://itnext.io/writing-scripts-as-frontend-with-es6-and-jest-bb53d32da8b5?source=collection_archive---------1----------------------->

![](img/2d70b1fa43c9d01a1b0a134673d5f725.png)

# 语境

作为前端开发人员，当我想从一个网站提取信息，然后注入到一个文档(通常是 markdown 文档)中时，我会做一些`web scraping`来了解如何使用 [CSS 选择器](https://www.w3schools.com/cssref/css_selectors.asp)(例如`classNames`)从网页中获取我需要的元素的数据，然后我在控制台中执行一个简单的脚本，如下所示:

```
copy(
  Array.from(document.querySelectorAll('.row-title'))
    .map(e => e.textContent)
    .join(';;')
);
```

然后在 [Sublime](https://www.sublimetext.com/) 或 [VS Code](https://code.visualstudio.com/) 中，我使用那些连接字符粘贴它们；;'和多光标功能的编辑器来编辑它们，因为我想给它我需要的降价格式。

这一次，我想提取网页的技术，并把它们的链接保存在一个文档中。但是 wappalyzer 提供了[浏览器扩展](https://www.wappalyzer.com/download)，我们不能从控制台访问浏览器扩展的弹出内容，所以不可能像我以前那样从控制台脚本中提取信息。

# 代表团

我发现了这个 [npm 节点模块](https://www.npmjs.com/package/wappalyzer)，它开启了我的思维，让我知道如何以最终用户的身份使用它来获取网站上使用的技术并将其粘贴到剪贴板。

在我的职业生涯中，我创建了许多`.sh`来自动化开发过程、部署、可重复的任务，有时甚至使用 [AppleScript](https://en.wikipedia.org/wiki/AppleScript) ，但令我惊讶的是，我从未想过使用我的 web 开发工具集(这里是 javascript 和 npm)来编写“桌面脚本”。所以`npm scripts`和这个`wappalyzer node module`是一个很好的开始来构建要用 [npx](https://www.npmjs.com/package/npx) 执行的东西。

> ***我想写一个脚本来提取任何带有 npm 的页面的技术，通过 npx 来执行，并从 wappalyzer 中获取输出到 markdown 中的剪贴板。***

所以这个“练习”的目的不仅仅是解决它，而是在一些限制下学习一些东西。

*   我想要单元测试。
*   我想使用模块和最新的 es6 语法，而不仅仅是(它是在 node 内部执行的，所以我知道这将是一个难点)。
*   将其注册到 npm register，这样任何人都可以在将来使用它或由任何其他人使用。
*   当然，在 Github 中发布它，也许将来有人可以使用它或对它进行修改。

# 工具集和规划

我找到了这篇文章[“用 npm 构建一个简单的命令行工具”](https://blog.npmjs.org/post/118810260230/building-a-simple-command-line-tool-with-npm)，但是没有给我 es6 模块、babel、eslint 和 jest。我学会了如何通过`package.json`中的`bin`属性公开 npm 脚本。我们链接的那些脚本需要有一个作为节点脚本执行的`[Shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))`。

然后我就知道用`last es6 syntax`、`node modules imports`、`prettier`，...我需要一个工具来编译`js`，我的第一选择是`[webpack](https://webpack.js.org/)`，就像我在我的 web 项目中经常做的那样。但是`webpack`是为了构建 web 输出而创建的，我不得不将它修改为不要期望 HTML、CSS，并且在`.js`中接受带有`Shebang`的脚本，所以我最终决定使用简单的`babel-cli`。

`Jest`对于单元测试来说是一个非常明确的决定，没有任何额外的配置，因为它对测试功能非常有效，我们不需要任何特别的东西，比如[酶](https://airbnb.io/enzyme/)或[反应测试库](https://testing-library.com/react)。

最后一个难题是如何复制机器剪贴板上的内容，因为我知道在浏览器控制台中有这个`[copy](https://css-tricks.com/can-copy-console/)`，但在`node`中没有。但是又一个准备好使用的节点模块在那里做这项工作(`[clipboardy](https://github.com/sindresorhus/clipboardy)`)

# 履行

所以一旦我清楚了如何构建这个项目，我只需要开始。

# 项目初始化

只是创建了一个简单的空的新文件夹`wappalyzer-to-md`，剩下的由 npm `npm init`完成。

```
mkdir wappalyzer-to-md; cd wappalyzer-to-md; npm init
```

# 配置`.babelrc`和`eslint`

现在让我们添加`.babelrc`和`eslint`配置:

首先，我们安装配置所需的东西:

`npm install --save-dev @babel/core @babel/cli @babel/preset-env @babel/node`

而`.babelrc`文件看起来是这样的:

```
{
  "presets": ["@babel/preset-env"]
}
```

然后是 eslint 的，因为现在没有 eslint+pretty 我无法工作。同样，对于开发依赖关系:

```
npm install --save-dev babel-eslint eslint eslint-config-node eslint-config-prettier eslint-plugin-prettier babel-loader babel-polyfill
```

配置文件(`.eslintrc`)也是这样，在对这个项目做了一些特定的修改之后:

```
{
  "extends": ["node", "prettier"],
  "parser": "babel-eslint",
  "parserOptions": {
    "ecmaVersion": 8,
    "ecmaFeatures": {
      "experimentalObjectRestSpread": true,
      "impliedStrict": true,
      "classes": true
    }
  },
  "env": {
    "es6": true,
    "browser": false,
    "jasmine": true,
    "node": true,
    "commonjs": false,
    "jest": true
  },
  "rules": {
    "no-unused-vars": [
      1,
      {
        "ignoreRestSiblings": true,
        "argsIgnorePattern": "^ignore",
        "varsIgnorePattern": "^ignore",
        "caughtErrorsIgnorePattern": "^ignore"
      }
    ],
    "arrow-body-style": [2, "as-needed"],
    "no-param-reassign": [
      2,
      {
        "props": false
      }
    ],
    "no-console": 0,
    "import": 0,
    "func-names": 0,
    "space-before-function-paren": 0,
    "max-len": 0,
    "no_underscore-dangle": 0,
    "consistent-return": 0,
    "comma-dangle": 0,
    "import/prefer-default-export": 0,
    "array-bracket-spacing": 0,
    "space-in-parens": 0,
    "prefer-arrow-callback": 0,
    "no-plusplus": 0,
    "no-use-before-define": 0,
    "global-require": 1,
    "import/no-commonjs": 0,
    "import/no-extraneous-dependencies": [
      "error",
      {
        "devDependencies": true
      }
    ],
    "no-process-exit": 0
  },
  "plugins": ["prettier"]
}
```

# 要使用的库和包设置`npm build`

然后我们需要能够使用`babel`转换过程来构建。/src，并在`bin`脚本中连接。

```
{
  // ...
  "scripts": {
    "build": "babel src --out-dir dist",
    "test": "jest --watchAll"
  },
  // ...
  "bin": {
    "wappalyzer-to-md": "./dist/cli.js"
  },
  // ...
}
```

使用前面的配置，我们能够执行`babel`编译的源代码。我们可以在每次构建项目时更新它们。

因此，让我们也创建我们的第一个文件，这将是我们的脚本`/src/cli.js`的起点:

```
console.log('-- Starting Execution --');
const [, , url] = process.argv;(async () => {
  try {
    // 1.- We need to validate params
    // 2.- Make the call to the sever side to check
    // 3.- Transform it to markdown
    // 4.- Copy it to the clipboard console.log(' -- 📋 Markdown Copied 📋 -- ');
  } catch (e) {
    console.error(' --💥 Something when wrong 💥-- ');
    console.error(e);
  } finally {
    process.exit(0);
  }
})();
```

我们不仅创建了文件，而且定义了“脚本工作流”,并对其进行了分割，以便能够以正确的方式处理测试💁‍.

# 使用`npm link`执行项目

`[npm link](https://docs.npmjs.com/cli/link)`在注册之前测试和使用`npm modules`是一个很好的方法。

要执行前面的示例，我们只需在控制台的项目文件夹中执行。

```
npm link
```

然后，我们可以使用以下命令来执行它:

```
npx wappalyzer-to-md
```

并在控制台中获得控制台输出。

```
-- Starting Execution --
-- 📋 Markdown Copied 📋 --
```

# 笑话设置`npm test`

让我们添加 uni 测试配置，以便能够实现前面单元测试中描述的工作流的每个部分。为此，让我们安装 jest 配置

```
npm install --save-dev jest babel-jest
```

在`test`脚本中拥有`jest --watchAll`让我能够执行`npm test`

我得到了这些问题:`ReferenceError: regeneratorRuntime is not defined`在这里描述了，我通过在`.babelrc`中指定巴别塔的目标得到了解决。

```
{
  "presets": [
    [
      "@babel/preset-env", {
        "targets": {
          "node": "current"
        }
      }
    ]
  ]
}
```

然后我就可以开始用任何一个`js`项目像往常一样运行`npm test`来编码了。

# 实施计划

对于实现，我们只需要 3 个库两个已经描述过的库`wappalyzer`和`clipboardy`以及不可或缺的库`lodash`。

```
npm install lodash clipboardy wappalyzer
```

我的 [Github](https://github.com/robertovg/wappalyzer-to-md) 账户里有完整的项目，你可以检查、使用、修改或改进它。

基本上，我为`cli.js`主干中描述的每个步骤创建了以下文件夹结构:

*   **验证**:进行参数验证，因为在 cli 中输入来自调用参数。
*   **数据**:包装使用`Wappalyzer` npm 模块的逻辑并测试。
*   **逻辑**:对我来说，这个程序的逻辑是将 Json 输出从`Wappalyzer`转移到我试图创建的“标准”markdown。以优雅的方式创作，非常直，非常令人兴奋🤓
*   **输出**:在脚本中，我们没有查看，但是我们有输出逻辑，所以我再次决定将`clipboardy`逻辑分离出来，放在逻辑文件夹中。

这些是小脚本项目的主要部分，我认为了解每个部分如何工作的最好方法，正如我认为我们应该一直做的那样，只是阅读每个文件夹上的`specs`测试，然后理解编排脚本片段的简单的`cli.js`文件。

此外，要在注册脚本项目之前执行，我们应该`npm link`该项目并使用有效参数再次执行它，如:

```
npx wappalyzer-to-md [https://robertovg.com](https://robertovg.com/)
```

我们将在剪贴板上有一个我们想要提取🥳.的减价文件

# 在 npmjs.com 注册将被广泛使用🤔。

所以最后一步是注册到 npm，这样我们就可以在没有源代码的情况下使用它，这是一个非常有趣的方法，老实说，这是我第一次想到在 npm 上注册一个不是作为`node module`而是作为`npm script`使用的东西。

很简单，这是由`npm publish`一如既往地完成的，首先，你需要在[npmjs.com](https://www.npmjs.com/)建立一个账户，并通过`npm login`调用登录 npm。

那么任何打电话的人都可以得到这个包

```
npx wappalyzer-to-md <url>
```

并在 npm [寄存器](https://www.npmjs.com/package/wappalyzer-to-md)中公开列出。

# 结论

这里没有什么疯狂的，但对我来说，用我的 web 工具集制作一个脚本很有趣。如果我需要再次解决脚本挑战，当我看到未来的机会时，我会尝试使用这种方法。

你曾经这样使用过`npm script`吗？你认为我使用它的方式有所改进吗？或者也许你有任何疑问？我总是很高兴从任何反馈中学习，希望这不仅对我有用，而且对任何面临问题的人都有用，这些问题可以在`npm scripts`之前解决。

我希望你喜欢阅读🤗。

*最初发表于*[T5【robertovg.com】](https://robertovg.com/blog/writing-scripts-as-frontend-with-es6-and-jest)*。*