# 使用 Rush monorepo 和 React 构建可扩展的前端—第 2 部分

> 原文：<https://itnext.io/build-a-scalable-front-end-with-rush-monorepo-and-react-part-2-d7f1c19c1797?source=collection_archive---------2----------------------->

# Webpack + Jest

![](img/89a248ffe0ec5a346b3fb966902b87be.png)

这是博客系列“用 Rush monorepo 和 React 构建可伸缩前端”的第二部分

*   [第 1 部分](https://medium.com/@alexandrubereghici/build-a-scalable-front-end-with-rush-monorepo-and-react-part-1-dd50ae38ad3e) : Monorepo 设置，导入保留 git 历史的项目，
    添加更漂亮的
*   第 2 部分:用 Webpack 和 react-scripts 创建构建工具包
*   [第 3 部分](https://medium.com/@alexandrubereghici/build-a-scalable-front-end-with-rush-monorepo-and-react-part-3-b90430f15af7):添加共享 ESLint 配置，并与 lint-staged 一起使用
*   [第 4 部分](https://medium.com/@alexandrubereghici/build-a-scalable-front-end-with-rush-monorepo-and-react-part-4-d0939bfb8b8a):用 Github 动作和 Netlify 设置部署工作流。
*   [第 5 部分](https://medium.com/@alexandrubereghici/build-a-scalable-front-end-with-rush-monorepo-and-react-part-5-355f5391fd27):增加 VSCode 配置，获得更好的开发体验。

# TL；速度三角形定位法(dead reckoning)

如果你有兴趣只是看看代码，你可以在这里找到它:[https://github.com/abereghici/rush-monorepo-boilerplate](https://github.com/abereghici/rush-monorepo-boilerplate)

如果你想看看 Rush 在一个真实的大型项目中的应用，你可以看看 Bentley Systems 开发的开源项目 [ITwin.js](https://github.com/imodeljs/imodeljs) 。

[create-react-app](https://create-react-app.dev/) 是一个伟大的工具，它允许你开发 react 应用程序，而不必手动配置构建工具。当您开发小型项目时，管理您自己的构建工具可能有些多余，CRA 已经足够了，但是当您开始构建大型项目时，您可能会发现缺少 CRA 配置。CRA 是一个固执己见的预配置工具，所以总会有一些你不满意的设置，你不能改变它们。

在这篇文章中，我们将基于 [create-react-app](https://create-react-app.dev/) 创建我们自己的构建工具。这将使我们能够完全控制构建过程。

## 创建反应脚本包

让我们为`react-scripts`包创建一个新文件夹。

```
mkdir -p packages/react-scripts
```

在`react-scripts`文件夹中创建一个`package.json`文件，并添加以下配置:

```
{
  "name": "@monorepo/react-scripts",
  "version": "1.0.0",
  "files": ["bin", "config", "lib", "scripts", "utils"],
  "bin": {
    "react-scripts": "./bin/react-scripts.js"
  },
  "types": "./lib/react-app.d.ts",
  "dependencies": {
    "@babel/core": "7.12.3",
    "@pmmmwh/react-refresh-webpack-plugin": "0.4.3",
    "@svgr/webpack": "5.5.0",
    "@testing-library/jest-dom": "^5.14.1",
    "@testing-library/react": "^11.2.7",
    "@testing-library/user-event": "^12.8.3",
    "@types/jest": "^26.0.24",
    "@types/node": "^12.20.19",
    "@types/react": "^17.0.18",
    "@types/react-dom": "^17.0.9",
    "@typescript-eslint/eslint-plugin": "^4.5.0",
    "@typescript-eslint/parser": "^4.5.0",
    "babel-eslint": "^10.1.0",
    "babel-jest": "^26.6.0",
    "babel-loader": "8.1.0",
    "babel-plugin-named-asset-import": "^0.3.7",
    "babel-preset-react-app": "^10.0.0",
    "bfj": "^7.0.2",
    "camelcase": "^6.1.0",
    "case-sensitive-paths-webpack-plugin": "2.3.0",
    "css-loader": "4.3.0",
    "dotenv": "8.2.0",
    "dotenv-expand": "5.1.0",
    "file-loader": "6.1.1",
    "fs-extra": "^9.0.1",
    "html-webpack-plugin": "4.5.0",
    "identity-obj-proxy": "3.0.0",
    "jest": "26.6.0",
    "jest-circus": "26.6.0",
    "jest-resolve": "26.6.0",
    "jest-watch-typeahead": "0.6.1",
    "mini-css-extract-plugin": "0.11.3",
    "optimize-css-assets-webpack-plugin": "5.0.4",
    "pnp-webpack-plugin": "1.6.4",
    "postcss-flexbugs-fixes": "4.2.1",
    "postcss-loader": "3.0.0",
    "postcss-normalize": "8.0.1",
    "postcss-preset-env": "6.7.0",
    "postcss-safe-parser": "5.0.2",
    "prompts": "2.4.0",
    "react-app-polyfill": "^2.0.0",
    "react-dev-utils": "^11.0.3",
    "react-refresh": "^0.8.3",
    "resolve": "1.18.1",
    "resolve-url-loader": "^3.1.2",
    "sass-loader": "^10.0.5",
    "semver": "7.3.2",
    "style-loader": "1.3.0",
    "terser-webpack-plugin": "4.2.3",
    "ts-pnp": "1.2.0",
    "url-loader": "4.1.1",
    "web-vitals": "^1.1.2",
    "webpack": "4.44.2",
    "webpack-dev-server": "3.11.1",
    "webpack-manifest-plugin": "2.2.0",
    "workbox-webpack-plugin": "5.1.4"
  },
  "devDependencies": {
    "react": "^17.0.1",
    "react-dom": "^17.0.1"
  }
}
```

正如您已经注意到的，我们将所有构建依赖项从`react-app`项目转移到了`react-scripts`包中。

将`config`和`scripts`文件夹从`react-app`移到`react-scripts`，让我们调整配置。

从`packages/react-scripts/config/webpack.config.js`中移除`ESLint`插件及其依赖项。稍后，我们将在构建过程中使用它作为一个单独的操作。

打开`packages/react-scripts/config/paths.js`，将文件内容替换为:

```
const path = require('path');
const fs = require('fs');
const getPublicUrlOrPath = require('react-dev-utils/getPublicUrlOrPath');

*// Make sure any symlinks in the project folder are resolved:*
*// https://github.com/facebook/create-react-app/issues/637*
const appDirectory = fs.realpathSync(process.cwd());
const resolveApp = relativePath => path.resolve(appDirectory, relativePath);

*// We use `PUBLIC_URL` environment variable or "homepage" field to infer*
*// "public path" at which the app is served.*
*// webpack needs to know it to put the right <script> hrefs into HTML even in*
*// single-page apps that may serve index.html for nested URLs like /todos/42.*
*// We can't use a relative path in HTML because we don't want to load something*
*// like /todos/42/static/js/bundle.7289d.js. We have to know the root.*
const publicUrlOrPath = getPublicUrlOrPath(
  process.env.NODE_ENV === 'development',
  require(resolveApp('package.json')).homepage,
  process.env.PUBLIC_URL
);

const buildPath = process.env.BUILD_PATH || 'build';

const moduleFileExtensions = [
  'web.mjs',
  'mjs',
  'web.js',
  'js',
  'web.ts',
  'ts',
  'web.tsx',
  'tsx',
  'json',
  'web.jsx',
  'jsx',
];

*// Resolve file paths in the same order as webpack*
const resolveModule = (resolveFn, filePath) => {
  const extension = moduleFileExtensions.find(extension =>
    fs.existsSync(resolveFn(`${filePath}.${extension}`))
  );

  if (extension) {
    return resolveFn(`${filePath}.${extension}`);
  }

  return resolveFn(`${filePath}.js`);
};

const resolveOwn = relativePath => path.resolve(__dirname, '..', relativePath);

module.exports = {
  dotenv: resolveApp('.env'),
  appPath: resolveApp('.'),
  appBuild: resolveApp(buildPath),
  appPublic: resolveApp('public'),
  appHtml: resolveApp('public/index.html'),
  appIndexJs: resolveModule(resolveApp, 'src/index'),
  appPackageJson: resolveApp('package.json'),
  appSrc: resolveApp('src'),
  appTsConfig: resolveApp('tsconfig.json'),
  appJsConfig: resolveApp('jsconfig.json'),
  yarnLockFile: resolveApp('yarn.lock'),
  testsSetup: resolveModule(resolveApp, 'src/setupTests'),
  proxySetup: resolveApp('src/setupProxy.js'),
  appNodeModules: resolveApp('node_modules'),
  appWebpackCache: resolveApp('node_modules/.cache'),
  appTsBuildInfoFile: resolveApp('node_modules/.cache/tsconfig.tsbuildinfo'),
  swSrc: resolveModule(resolveApp, 'src/service-worker'),
  publicUrlOrPath,
  *// These properties only exist before ejecting:*
  ownPath: resolveOwn('.'),
  ownNodeModules: resolveOwn('node_modules'), *// This is empty on npm 3*
  appTypeDeclarations: resolveApp('src/react-app-env.d.ts'),
  ownTypeDeclarations: resolveOwn('lib/react-app.d.ts'),
};

module.exports.moduleFileExtensions = moduleFileExtensions;
```

这个文件中最重要的变化是`resolveApp`函数调用。我们不应该再使用相对路径，因为我们正在将配置文件移动到`react-scripts`包中。我们将使用`path.resolve`和`process.cwd()`的组合来获取项目文件路径。

创建一个`bin`文件夹，并在`react-scripts.js`文件中添加以下代码:

```
process.on('unhandledRejection', err => {
  throw err;
});

const spawn = require('react-dev-utils/crossSpawn');
const args = process.argv.slice(2);

const scriptIndex = args.findIndex(
  x => x === 'build' || x === 'start' || x === 'test'
);
const script = scriptIndex === -1 ? args[0] : args[scriptIndex];
const nodeArgs = scriptIndex > 0 ? args.slice(0, scriptIndex) : [];

if (['build', 'start', 'test'].includes(script)) {
  const result = spawn.sync(
    process.execPath,
    nodeArgs
      .concat(require.resolve('../scripts/' + script))
      .concat(args.slice(scriptIndex + 1)),
    { stdio: 'inherit' }
  );
  if (result.signal) {
    if (result.signal === 'SIGKILL') {
      console.log(
        'The build failed because the process exited too early. ' +
          'This probably means the system ran out of memory or someone called ' +
          '`kill -9` on the process.'
      );
    } else if (result.signal === 'SIGTERM') {
      console.log(
        'The build failed because the process exited too early. ' +
          'Someone might have called `kill` or `killall`, or the system could ' +
          'be shutting down.'
      );
    }
    process.exit(1);
  }
  process.exit(result.status);
} else {
  console.log('Unknown script "' + script + '".');
}
```

这将是我们 CLI 的入口点。每次当你想添加一个新的命令时，你必须调整这个文件。

创建另一个名为`lib`的文件夹，并在`react-app.d.ts`文件中添加以下代码:

```
*/// <reference types="node" />*
*/// <reference types="react" />*
*/// <reference types="react-dom" />*

declare namespace NodeJS {
  interface ProcessEnv {
    readonly NODE_ENV: 'development' | 'production' | 'test';
    readonly PUBLIC_URL: string;
  }
}

declare module '*.avif' {
  const src: string;
  export default src;
}

declare module '*.bmp' {
  const src: string;
  export default src;
}

declare module '*.gif' {
  const src: string;
  export default src;
}

declare module '*.jpg' {
  const src: string;
  export default src;
}

declare module '*.jpeg' {
  const src: string;
  export default src;
}

declare module '*.png' {
  const src: string;
  export default src;
}

declare module '*.webp' {
  const src: string;
  export default src;
}

declare module '*.svg' {
  import * as React from 'react';

  export const ReactComponent: React.FunctionComponent<
    React.SVGProps<SVGSVGElement> & { title?: string }
  >;

  const src: string;
  export default src;
}

declare module '*.module.css' {
  const classes: { readonly [key: string]: string };
  export default classes;
}

declare module '*.module.scss' {
  const classes: { readonly [key: string]: string };
  export default classes;
}

declare module '*.module.sass' {
  const classes: { readonly [key: string]: string };
  export default classes;
}
```

这里我们将存储 CSS、SASS、SCSS 和图像文件的所有类型定义。我们将在 monorepo 内部的所有 react 应用程序中重用它们。您可以通过在 react 项目中创建一个名为`react-app-env.d.ts`的文件来处理这些类型定义，该文件包含以下内容:

```
*/// <reference types="@monorepo/react-scripts" />*
```

现在让我们在 rush 配置文件中注册`@monorepo/react-scripts`包。打开`rush.json`文件，在项目清单下添加一个条目，如下所示:

```
. . .
 "projects": [
    {
      "packageName": "@monorepo/react-scripts",
      "projectFolder": "packages/react-scripts",
      "reviewCategory": "tools"
    }
  ]
. . .
```

现在，让我们通过删除所有与构建工具相关的依赖项来清理`apps/react-app/package-json`文件。我们还将删除`eslint`、`babel`或`jest`的配置，因为我们会将它们存储在一个单独的文件中。在我们的依赖项中，我们将列出`@monorepo/react-scripts`包。

编辑`apps/react-app/package.json`文件并添加以下配置:

```
{
  "name": "@monorepo/react-app",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^5.14.1",
    "@testing-library/react": "^11.2.7",
    "@testing-library/user-event": "^12.8.3",
    "@types/jest": "^26.0.24",
    "@types/react": "^17.0.18",
    "@types/react-dom": "^17.0.9",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "typescript": "^4.3.5",
    "web-vitals": "^1.1.2",
    "@monorepo/react-scripts": "1.0.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test"
  }
}
```

运行`rush update`来安装所有的依赖项，让我们试着用刚刚创建的`react-script`包来构建我们的应用程序。

```
cd apps/react-app rushx build
```

## 设置浏览器列表

默认情况下，`react-scripts`期望在每个项目中找到一个`browserslist`配置，否则，它会退回到默认的`browserslist`配置。如果您想对所有项目使用相同的配置，您可以修改`packages/react-scripts/build.js`以使用来自`package.json`和`@monorepo/react-scripts`的`browserslist`。

从`@monorepo/react-scripts`打开`package.json`，添加以下配置:

```
"browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
```

用以下命令修改来自`packages/react-scripts/build.js`和`packages/react-scripts/start.js`的`checkBrowsers`函数调用:

```
checkBrowsers(paths.ownPath, isInteractive)
```

## 设置笑话

我们当前的`test`脚本期望在每个项目中找到一个`jest`配置。我们将修改它以使用共享配置。

在`packages/react-scripts/scripts`中创建一个`utils`文件夹，并在`createJestConfig.js`文件中添加以下内容:

```
const fs = require('fs');
const chalk = require('react-dev-utils/chalk');
const paths = require('../../config/paths');
const modules = require('../../config/modules');

module.exports = (resolve, rootDir, isEjecting) => {
  *// Use this instead of `paths.testsSetup` to avoid putting*
  *// an absolute filename into configuration after ejecting.*
  const setupTestsMatches = paths.testsSetup.match(/src[/\\]setupTests\.(.+)/);
  const setupTestsFileExtension =
    (setupTestsMatches && setupTestsMatches[1]) || 'js';
  const setupTestsFile = fs.existsSync(paths.testsSetup)
    ? `<rootDir>/src/setupTests.${setupTestsFileExtension}`
    : undefined;

  const config = {
    roots: ['<rootDir>/src'],

    collectCoverageFrom: ['src/**/*.{js,jsx,ts,tsx}', '!src/**/*.d.ts'],

    setupFiles: [
      isEjecting
        ? 'react-app-polyfill/jsdom'
        : require.resolve('react-app-polyfill/jsdom'),
    ],

    setupFilesAfterEnv: setupTestsFile ? [setupTestsFile] : [],
    testMatch: [
      '<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}',
      '<rootDir>/src/**/*.{spec,test}.{js,jsx,ts,tsx}',
    ],
    testEnvironment: 'jsdom',
    testRunner: require.resolve('jest-circus/runner'),
    transform: {
      '^.+\\.(js|jsx|mjs|cjs|ts|tsx)$': resolve(
        'config/jest/babelTransform.js'
      ),
      '^.+\\.css$': resolve('config/jest/cssTransform.js'),
      '^(?!.*\\.(js|jsx|mjs|cjs|ts|tsx|css|json)$)': resolve(
        'config/jest/fileTransform.js'
      ),
    },
    transformIgnorePatterns: [
      '[/\\\\]node_modules[/\\\\].+\\.(js|jsx|mjs|cjs|ts|tsx)$',
      '^.+\\.module\\.(css|sass|scss)$',
    ],
    modulePaths: modules.additionalModulePaths || [],
    moduleNameMapper: {
      '^react-native$': 'react-native-web',
      '^.+\\.module\\.(css|sass|scss)$': 'identity-obj-proxy',
      ...(modules.jestAliases || {}),
    },
    moduleFileExtensions: [...paths.moduleFileExtensions, 'node'].filter(
      ext => !ext.includes('mjs')
    ),
    watchPlugins: [
      'jest-watch-typeahead/filename',
      'jest-watch-typeahead/testname',
    ],
    resetMocks: true,
  };
  if (rootDir) {
    config.rootDir = rootDir;
  }
  const overrides = Object.assign({}, require(paths.appPackageJson).jest);
  const supportedKeys = [
    'clearMocks',
    'collectCoverageFrom',
    'coveragePathIgnorePatterns',
    'coverageReporters',
    'coverageThreshold',
    'displayName',
    'extraGlobals',
    'globalSetup',
    'globalTeardown',
    'moduleNameMapper',
    'resetMocks',
    'resetModules',
    'restoreMocks',
    'snapshotSerializers',
    'testMatch',
    'transform',
    'transformIgnorePatterns',
    'watchPathIgnorePatterns',
  ];
  if (overrides) {
    supportedKeys.forEach(key => {
      if (Object.prototype.hasOwnProperty.call(overrides, key)) {
        if (Array.isArray(config[key]) || typeof config[key] !== 'object') {
          *// for arrays or primitive types, directly override the config key*
          config[key] = overrides[key];
        } else {
          *// for object types, extend gracefully*
          config[key] = Object.assign({}, config[key], overrides[key]);
        }

        delete overrides[key];
      }
    });
    const unsupportedKeys = Object.keys(overrides);
    if (unsupportedKeys.length) {
      const isOverridingSetupFile =
        unsupportedKeys.indexOf('setupFilesAfterEnv') > -1;

      if (isOverridingSetupFile) {
        console.error(
          chalk.red(
            'We detected ' +
              chalk.bold('setupFilesAfterEnv') +
              ' in your package.json.\n\n' +
              'Remove it from Jest configuration, and put the initialization code in ' +
              chalk.bold('src/setupTests.js') +
              '.\nThis file will be loaded automatically.\n'
          )
        );
      } else {
        console.error(
          chalk.red(
            '\nOut of the box, Create React App only supports overriding ' +
              'these Jest options:\n\n' +
              supportedKeys
                .map(key => chalk.bold('  \u2022 ' + key))
                .join('\n') +
              '.\n\n' +
              'These options in your package.json Jest configuration ' +
              'are not currently supported by Create React App:\n\n' +
              unsupportedKeys
                .map(key => chalk.bold('  \u2022 ' + key))
                .join('\n') +
              '\n\nIf you wish to override other Jest options, you need to ' +
              'eject from the default setup. You can do so by running ' +
              chalk.bold('npm run eject') +
              ' but remember that this is a one-way operation. ' +
              'You may also file an issue with Create React App to discuss ' +
              'supporting more options out of the box.\n'
          )
        );
      }

      process.exit(1);
    }
  }
  return config;
};
```

现在让我们使用这个配置。修改`packages/react-scripts/scripts/test.js`中的`test`脚本，如下所示:

```
*// Do this as the first thing so that any code reading it knows the right env.*
process.env.BABEL_ENV = 'test';
process.env.NODE_ENV = 'test';
process.env.PUBLIC_URL = '';

process.on('unhandledRejection', err => {
  throw err;
});

*// Ensure environment variables are read.*
require('../config/env');

const jest = require('jest');
const execSync = require('child_process').execSync;
let argv = process.argv.slice(2);

function isInGitRepository() {
  try {
    execSync('git rev-parse --is-inside-work-tree', { stdio: 'ignore' });
    return true;
  } catch (e) {
    return false;
  }
}

function isInMercurialRepository() {
  try {
    execSync('hg --cwd . root', { stdio: 'ignore' });
    return true;
  } catch (e) {
    return false;
  }
}

*// Watch unless on CI or explicitly running all tests*
if (
  !process.env.CI &&
  argv.indexOf('--watchAll') === -1 &&
  argv.indexOf('--watchAll=false') === -1
) {
  *// https://github.com/facebook/create-react-app/issues/5210*
  const hasSourceControl = isInGitRepository() || isInMercurialRepository();
  argv.push(hasSourceControl ? '--watch' : '--watchAll');
}

const createJestConfig = require('./utils/createJestConfig');
const path = require('path');
const paths = require('../config/paths');
argv.push(
  '--config',
  JSON.stringify(
    createJestConfig(
      relativePath => path.resolve(__dirname, '..', relativePath),
      path.resolve(paths.appSrc, '..'),
      false
    )
  )
);

*// This is a very dirty workaround for https://github.com/facebook/jest/issues/5913.*
*// We're trying to resolve the environment ourselves because Jest does it incorrectly.*
*// TODO: remove this as soon as it's fixed in Jest.*
const resolve = require('resolve');
function resolveJestDefaultEnvironment(name) {
  const jestDir = path.dirname(
    resolve.sync('jest', {
      basedir: __dirname,
    })
  );
  const jestCLIDir = path.dirname(
    resolve.sync('jest-cli', {
      basedir: jestDir,
    })
  );
  const jestConfigDir = path.dirname(
    resolve.sync('jest-config', {
      basedir: jestCLIDir,
    })
  );
  return resolve.sync(name, {
    basedir: jestConfigDir,
  });
}
let cleanArgv = [];
let env = 'jsdom';
let next;
do {
  next = argv.shift();
  if (next === '--env') {
    env = argv.shift();
  } else if (next.indexOf('--env=') === 0) {
    env = next.substring('--env='.length);
  } else {
    cleanArgv.push(next);
  }
} while (argv.length > 0);
argv = cleanArgv;
let resolvedEnv;
try {
  resolvedEnv = resolveJestDefaultEnvironment(`jest-environment-${env}`);
} catch (e) {
  *// ignore*
}
if (!resolvedEnv) {
  try {
    resolvedEnv = resolveJestDefaultEnvironment(env);
  } catch (e) {
    *// ignore*
  }
}
const testEnvironment = resolvedEnv || env;
argv.push('--env', testEnvironment);

jest.run(argv);
```

现在我们可以在`react-app`项目中运行`rushx test`并测试共享配置。

如果你在这个过程中遇到了任何问题，你可以在[这个提交](https://github.com/abereghici/rush-monorepo-boilerplate/commit/de30f0a1bc1f3958e34bcccf7089d3439d153e2c)中检查所有的修改。

在下一部分的[中，我们将添加一个共享的 ESLint 配置，并将它与](https://medium.com/@alexandrubereghici/build-a-scalable-front-end-with-rush-monorepo-and-react-part-3-b90430f15af7) [lint-staged](https://github.com/okonet/lint-staged) 集成。

*原发布于*[*https://bereghici . dev*](https://bereghici.dev/blog/build-a-scalable-front-end-with-rush-monorepo-and-react--webpack+jest)*。*