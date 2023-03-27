# 使用 Rush monorepo 和 React 构建可扩展的前端—第 3 部分

> 原文：<https://itnext.io/build-a-scalable-front-end-with-rush-monorepo-and-react-part-3-b90430f15af7?source=collection_archive---------3----------------------->

# ESLint + Lint 暂存

![](img/aa52b2bdc5d89dac2f8ec7d4444cfeae.png)

这是博客系列“用 Rush monorepo 和 React 构建可伸缩前端”的第 3 部分

*   [第 1 部分](https://medium.com/@alexandrubereghici/build-a-scalable-front-end-with-rush-monorepo-and-react-part-1-dd50ae38ad3e) : Monorepo 设置，导入项目并保留 git 历史，
    添加漂亮的
*   第 2 部分:用 Webpack 和 react-scripts 创建构建工具包
*   [第 3 部分](https://medium.com/@alexandrubereghici/build-a-scalable-front-end-with-rush-monorepo-and-react-part-3-b90430f15af7):添加共享 ESLint 配置，并与 lint-staged 一起使用
*   [第 4 部分](https://medium.com/@alexandrubereghici/build-a-scalable-front-end-with-rush-monorepo-and-react-part-4-d0939bfb8b8a):用 Github 动作和 Netlify 设置部署工作流。
*   [第 5 部分](https://medium.com/@alexandrubereghici/build-a-scalable-front-end-with-rush-monorepo-and-react-part-5-355f5391fd27):增加 VSCode 配置，获得更好的开发体验。

## TL；速度三角形定位法(dead reckoning)

如果你有兴趣只是看看代码，你可以在这里找到它:[https://github.com/abereghici/rush-monorepo-boilerplate](https://github.com/abereghici/rush-monorepo-boilerplate)

如果你想看看 Rush 在一个真实的大型项目中的应用，你可以看看由 Bentley Systems 开发的开源项目 [ITwin.js](https://github.com/imodeljs/imodeljs) 。

ESLint 是林挺打字稿和 JavaScript 代码的主要工具。我们将使用它和 Lint Staged 来实现我们在[第 1 部分](https://medium.com/@alexandrubereghici/build-a-scalable-front-end-with-rush-monorepo-and-react-part-1-dd50ae38ad3e)中定义的目标“*代码质量强制规则*”。

ESLint 使用您定义的一组规则。如果您已经有了您喜欢的 ESLint 配置，您可以将其添加到我们的下一个设置中。我们将使用 AirBnB 的 ESLint 配置，这是 JavaScript 项目最常见的规则列表。截至 2021 年年中，它每周从 NPM 获得超过 270 万次下载。

## 构建 eslint-config 包

让我们首先在`packages`中创建一个名为`eslint-config`的文件夹，并创建`package.json`文件。

```
mkdir packages/eslint-config touch packages/eslint-config/package.json
```

将以下内容粘贴到`packages/eslint-config/package.json`:

```
{
  "name": "@monorepo/eslint-config",
  "version": "1.0.0",
  "description": "Shared eslint rules",
  "main": "index.js",
  "scripts": {
    "build": ""
  },
  "dependencies": {
    "@babel/eslint-parser": "~7.14.4",
    "@babel/eslint-plugin": "~7.13.16",
    "@babel/preset-react": "~7.13.13",
    "@typescript-eslint/eslint-plugin": "^4.26.1",
    "@typescript-eslint/parser": "^4.26.1",
    "babel-eslint": "~10.1.0",
    "eslint-config-airbnb": "^18.2.1",
    "eslint-config-prettier": "^7.1.0",
    "eslint-config-react-app": "~6.0.0",
    "eslint-plugin-import": "^2.23.4",
    "eslint-plugin-flowtype": "^5.2.1",
    "eslint-plugin-jest": "^24.1.5",
    "eslint-plugin-jsx-a11y": "^6.4.1",
    "eslint-plugin-prettier": "^3.3.1",
    "eslint-plugin-react": "^7.24.0",
    "eslint-plugin-react-hooks": "^4.2.0",
    "eslint-plugin-testing-library": "^3.9.2"
  },
  "devDependencies": {
    "read-pkg-up": "7.0.1",
    "semver": "~7.3.5"
  },
  "peerDependencies": {
    "eslint": "^7.28.0",
    "typescript": "^4.3.5"
  },
  "peerDependenciesMeta": {
    "typescript": {
      "optional": true
    }
  }
}
```

在这里，我们添加了 ESLint 配置所需的所有依赖项。

现在，让我们创建一个`config.js`文件，我们将在其中定义与规则无关的 ESLint 配置。

```
const fs = require('fs');
const path = require('path');

const tsConfig = fs.existsSync('tsconfig.json')
  ? path.resolve('tsconfig.json')
  : undefined;

module.exports = {
  parser: '@babel/eslint-parser',
  parserOptions: {
    babelOptions: {
      presets: ['@babel/preset-react'],
    },
    requireConfigFile: false,
    ecmaVersion: 2021,
    sourceType: 'module',
    ecmaFeatures: {
      jsx: true,
    },
  },
  env: {
    es6: true,
    jest: true,
    browser: true,
  },
  globals: {
    globals: true,
    shallow: true,
    render: true,
    mount: true,
  },
  overrides: [
    {
      files: ['**/*.ts?(x)'],
      parser: '@typescript-eslint/parser',
      parserOptions: {
        ecmaVersion: 2021,
        sourceType: 'module',
        project: tsConfig,
        ecmaFeatures: {
          jsx: true,
        },
        warnOnUnsupportedTypeScriptVersion: true,
      },
    },
  ],
};
```

我们将把 ESLint 规则分成多个文件。在`base.js`文件中，我们将定义适用于所有包的主要规则。在`react.js`中将会有 React 特有的规则。我们可能有不使用 React 的包，所以我们将只使用`base`规则。

创建一个`base.js`文件并添加:

```
module.exports = {
  extends: ['airbnb', 'prettier'],
  plugins: ['prettier'],
  rules: {
    camelcase: 'error',
    semi: ['error', 'always'],
    quotes: [
      'error',
      'single',
      {
        allowTemplateLiterals: true,
        avoidEscape: true,
      },
    ],
  },
  overrides: [
    {
      files: ['**/*.ts?(x)'],
      extends: [
        'prettier/@typescript-eslint',
        'plugin:@typescript-eslint/recommended',
      ],
      rules: {},
    },
  ],
};
```

这里我们扩展了`airbnb`和`prettier`配置。在这里，您可以包括您想要使用的其他基本规则。

在`react.js`中添加以下内容:

```
const readPkgUp = require('read-pkg-up');
const semver = require('semver');

let oldestSupportedReactVersion = '17.0.1';

*// Get react version from package.json and used it in lint configuration*
try {
  const pkg = readPkgUp.sync({ normalize: true });
  const allDeps = Object.assign(
    { react: '17.0.1' },
    pkg.peerDependencies,
    pkg.devDependencies,
    pkg.dependencies
  );

  oldestSupportedReactVersion = semver
    .validRange(allDeps.react)
    .replace(/[>=<|]/g, ' ')
    .split(' ')
    .filter(Boolean)
    .sort(semver.compare)[0];
} catch (error) {
  *// ignore error*
}

module.exports = {
  extends: [
    'react-app',
    'react-app/jest',
    'prettier/react',
    'plugin:testing-library/recommended',
    'plugin:testing-library/react',
  ],
  plugins: ['react', 'react-hooks', 'testing-library', 'prettier'],
  settings: {
    react: {
      version: oldestSupportedReactVersion,
    },
  },
  rules: {
    'react/jsx-fragments': ['error', 'element'],
    'react-hooks/rules-of-hooks': 'error',
  },
  overrides: [
    {
      files: ['**/*.ts?(x)'],
      rules: {
        'react/jsx-filename-extension': [
          1,
          {
            extensions: ['.js', '.jsx', '.ts', '.tsx'],
          },
        ],
      },
    },
  ],
};
```

我们必须为`react-app`配置提供一个`react`版本。我们将使用`read-pkg-up`从`package.json`获取版本，而不是硬编码。`semver`用于帮助我们挑选合适的版本。

最后一步是定义我们配置的入口点。创建一个`index.js`文件并添加:

```
module.exports = {
  extends: ['./config.js', './base.js'],
};
```

## 向 react-scripts 添加 lint 命令

ESLint 可以以多种方式使用。您可以将它安装在每个包中，或者创建一个为您运行 ESLint bin 的`lint`脚本。我觉得第二种方法更合适。我们可以在一个地方控制 ESLint 版本，这使得升级过程更加容易。

对于`lint`脚本，我们需要一些实用函数，所以在`packages/react-scripts/scripts/utils`中创建一个`index.js`文件，并添加以下内容:

```
const fs = require('fs');
const path = require('path');
const which = require('which');
const readPkgUp = require('read-pkg-up');

const { path: pkgPath } = readPkgUp.sync({
  cwd: fs.realpathSync(process.cwd()),
});

const appDirectory = path.dirname(pkgPath);

const fromRoot = (...p) => path.join(appDirectory, ...p);

function resolveBin(
  modName,
  { executable = modName, cwd = process.cwd() } = {}
) {
  let pathFromWhich;
  try {
    pathFromWhich = fs.realpathSync(which.sync(executable));
    if (pathFromWhich && pathFromWhich.includes('.CMD')) return pathFromWhich;
  } catch (_error) {
    *// ignore _error*
  }
  try {
    const modPkgPath = require.resolve(`${modName}/package.json`);
    const modPkgDir = path.dirname(modPkgPath);
    const { bin } = require(modPkgPath);
    const binPath = typeof bin === 'string' ? bin : bin[executable];
    const fullPathToBin = path.join(modPkgDir, binPath);
    if (fullPathToBin === pathFromWhich) {
      return executable;
    }
    return fullPathToBin.replace(cwd, '.');
  } catch (error) {
    if (pathFromWhich) {
      return executable;
    }
    throw error;
  }
}

module.exports = {
  resolveBin,
  fromRoot,
  appDirectory,
};
```

这里最重要的函数是`resolveBin`，它将尝试解析给定模块的二进制代码。

在`packages/react-scripts/scripts`中创建`lint.js`文件，并添加以下内容:

```
const spawn = require('react-dev-utils/crossSpawn');
const yargsParser = require('yargs-parser');
const { resolveBin, fromRoot, appDirectory } = require('./utils');

let args = process.argv.slice(2);
const parsedArgs = yargsParser(args);

const cache = args.includes('--no-cache')
  ? []
  : [
      '--cache',
      '--cache-location',
      fromRoot('node_modules/.cache/.eslintcache'),
    ];

const files = parsedArgs._;

const relativeEslintNodeModules = 'node_modules/@monorepo/eslint-config';
const pluginsDirectory = `${appDirectory}/${relativeEslintNodeModules}`;

const resolvePluginsRelativeTo = [
  '--resolve-plugins-relative-to',
  pluginsDirectory,
];

const result = spawn.sync(
  resolveBin('eslint'),
  [
    ...cache,
    ...files,
    ...resolvePluginsRelativeTo,
    '--no-error-on-unmatched-pattern',
  ],
  { stdio: 'inherit' }
);

process.exit(result.status);
```

在`packages/react-scripts/bin/react-scripts.js`中注册`lint`命令:

```
. . . const scriptIndex = args.findIndex(
  x => x === 'build' || x === 'start' || x === 'lint' || x === 'test'
);. . . if (['build', 'start', 'lint', 'test'].includes(script)) {. . .
```

现在，在`packages/react-scripts/package.json`中添加我们的新依赖项:

```
. . . "which": "~2.0.2",
    "read-pkg-up": "7.0.1",
    "yargs-parser": "~20.2.7",
    "eslint": "^7.28.0". . .
```

## 运行中的 Lint 脚本

我们的`lint`脚本已经准备好了，现在让我们在`react-app`项目中运行它。创建一个名为`.eslintrc.js`的新文件，并添加以下内容:

```
module.exports = {
  extends: ['@monorepo/eslint-config', '@monorepo/eslint-config/react'],
};
```

在`package.json`内添加`eslint-config`作为依赖:

```
 "@monorepo/eslint-config": "1.0.0" 
```

在`scripts`部分添加`lint`命令:

```
"lint": "react-scripts lint src"
```

由`rushx lint`跟随运行`rush update`。此时，您应该会看到一堆 ESLint 错误。作为练习，您可以尝试通过启用/禁用`eslint-config`中的一些规则来修复它们，或者修改`react-app`项目以使其通过林挺。

## 向 react 脚本添加 lint-staged 命令

我们将遵循与使用`lint`脚本相同的方法。在`packages/react-scripts/scripts`中创建`lint-staged.js`文件，并添加以下内容:

```
const spawn = require('react-dev-utils/crossSpawn');
const { resolveBin } = require('./utils');

const args = process.argv.slice(2);

result = spawn.sync(resolveBin('lint-staged'), [...args], {
  stdio: 'inherit',
});

process.exit(result.status);
```

在`package.json`中添加`lint-staged`作为依赖:

```
...
 "lint-staged": "~11.0.0"
...
```

打开`packages/react-scripts/bin/react-scripts.js`并记录`lint-staged`命令。

下一步是在`common/config/command-line.json`中注册一个`lint-staged` rush 命令，就像我们在[第 1 部分](https://bereghici.dev/blog/build-a-scalable-front-end-with-rush-monorepo-and-react--repo-setup+import-projects+prettier)中注册`prettier`命令一样。

```
{
  "name": "lint-staged",
  "commandKind": "bulk",
  "summary": "Run lint-staged on each package",
  "description": "Iterates through each package in the monorepo and runs the 'lint-staged' script",
  "enableParallelism": false,
  "ignoreMissingScript": true,
  "ignoreDependencyOrder": true,
  "allowWarningsInSuccessfulBuild": true
},
```

现在，让我们在 git `pre-commit`钩子上运行`lint-staged`命令。打开`common/git-hooks/pre-commit`并将追加添加到文件末尾:

```
node common/scripts/install-run-rush.js lint-staged || exit $?
```

## 林特登台演出

让我们定义我们希望`lint-staged`为`react-app`项目运行什么任务。打开`react-app`中的`package.json`，添加`lint-staged`的配置；

```
"lint-staged": {
    "src/**/*.{ts,tsx}": [
      "react-scripts lint --fix --",
      "react-scripts test --findRelatedTests --watchAll=false --silent"
    ],
  },
```

同样在`package.json`中添加新的`lint-staged`脚本:

```
"lint-staged": "react-scripts lint-staged"
```

现在，在每次提交时，`lint-staged`将 lint 我们的文件，并将运行相关文件的测试。

运行`rush install`来注册我们的命令，然后运行`rush update`来提交我们的更改，以查看所有操作。

如果你在这个过程中遇到任何问题，你可以在这里看到这篇文章[的相关代码。](https://github.com/abereghici/rush-monorepo-boilerplate/tree/049506a98529d4931b2a2ecb629957e0d93e8a8d)

让我们转到[下一部分](https://medium.com/@alexandrubereghici/build-a-scalable-front-end-with-rush-monorepo-and-react-part-4-d0939bfb8b8a)，在这里我们将看到如何使用 Github Actions 和 Netlify 来自动化我们的部署工作流。

*原载于*[*https://bereghici . dev*](https://bereghici.dev/blog/build-a-scalable-front-end-with-rush-monorepo-and-react--eslint+lint-staged)*。*