# 如何在 5 分钟内设置好 Typescript，Eslint，Prettier 和 React

> 原文：<https://itnext.io/how-to-setup-typescript-eslint-prettier-and-react-in-5-minutes-44cfe8af5081?source=collection_archive---------1----------------------->

![](img/99ddff580d5ac83e3192664edbda9dd3.png)

任何人都能做它

将 Tslint 与 Typescript 一起使用的日子已经一去不复返了。😰

Typescript 的[路线图是在官方 TSLint 项目被](https://github.com/Microsoft/TypeScript/issues/29288)[否决](https://github.com/palantir/tslint/issues/4534)的同时转移到 Eslint。

这对我们来说是个好消息！最后，我们将在 ECMAScript 世界中获得一些一致性，作为 Eslint 标准用于 Typescript 和 Javascript！🤩

这里是你如何轻松地启动和运行 Eslint，Typescript，更漂亮和反应。我会先给你一个设置，然后解释它是如何一起工作的，最后，我会给你一个工作示例。

# 先决条件

如果您没有现有的 typescript/react 项目，可以将 create-react-app 与 typescript 模板一起使用:

```
npx create-react-app my-app --template typescript
cd my-app
```

# 我们开始吧

1.  在终端中，转到项目的根目录并运行:

```
npm i -D eslint prettier eslint-config-airbnb-typescript-prettier
```

2.在项目的根目录下，添加一个文件`.eslintrc.js`:

```
// .eslintrc.jsmodule.exports = {
  extends: ["airbnb-typescript-prettier"],
};
```

3.在您的项目的根目录中，添加一个文件:`.prettierrc.js`:

```
// .prettierrc.jsmodule.exports = {
  singleQuote: true,
  printWidth: 80,
};
```

4.要运行并自动修复所有的格式和 lint 问题，向`package.json`添加以下脚本。我假设您的源代码在一个`src`子目录中。如果不是，请改变，使其适合您的项目。

```
"scripts": {
  **"format": "prettier --write src/**/*.{js,jsx,ts,tsx}",
  "lint": "eslint --fix src/**/*.{js,jsx,ts,tsx}"**
}
```

搞定了。👏

# 引擎盖下发生了什么？🧐

首先，我们安装 Eslint 和 Prettier 作为开发依赖项。

多亏了预置的`eslint-config-airbnb-typescript-prettier`,很多样板文件的配置将会为我们完成。

在引擎盖下`eslint-config-airbnb-typescript-prettier`将使用以下依赖关系:

**@ typescript-eslint/parser**和**@ Typescript-Eslint/Eslint-plugin**，它们是让 Eslint 与 Typescript 一起工作所需要的

**eslint-config-airbnb-typescript**是 Airbnb 用于 Eslint 和 Typescript 的预置。它非常严格，但是有一些默认的规则，如果需要的话可以选择不使用。

**eslint-plugin-import，eslint-plugin-jsx-a11y，eslint-plugin-react** 和**eslint-plugin-react-hooks**是 Airbnb 预置需要的。

需要 eslint-config-appellister 来防止 eslint 和 appellister 之间的格式冲突。

# Airbnb 预置太严格了吗？😰

我喜欢 Airbnb 的预设，但有时快速迭代比完美的代码更重要，尤其是当你刚刚开始一个新项目的时候。

你可以排除一些你不喜欢的规则，(也许以后再加回来):

```
// eslintrc.jsmodule.exports = {
  extends: ['airbnb-typescript-prettier'],
  **rules: {
    'react/prop-types': 0,
    'react/destructuring-assignment': 0,
    'react/static-property-placement': 0,
    'jsx-a11y/alt-text': 0,
    'react/jsx-props-no-spreading': 0,
  },**
};
```

# 一个工作实例

我给你举了一个例子。(基于 create-react-app 构建)。如果你喜欢新项目，你可以把它作为一个基础。
[https://github . com/caki 0915/typescript-react-eslint-appetter](https://github.com/caki0915/typescript-react-eslint-prettier)

编码快乐！🥰