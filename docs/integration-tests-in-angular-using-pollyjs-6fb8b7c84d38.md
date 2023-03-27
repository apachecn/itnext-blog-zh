# 使用 PollyJS 在 Angular 中进行集成测试

> 原文：<https://itnext.io/integration-tests-in-angular-using-pollyjs-6fb8b7c84d38?source=collection_archive---------8----------------------->

![](img/87ccc284e046afbb31aa0a960cb90f8c.png)

https://github.com/JonJam/angular_pollyjs_playground 的[有代码](https://github.com/JonJam/angular_pollyjs_playground)

# **背景故事**

在 2022 年期间，我的团队一直试图通过加快我们工作的整体建筑的建造时间(平均 30 分钟)来提高开发人员的生产率。通过将前端分离出来，我们取得了一些巨大的成功，前端是一个大型的 Angular 应用程序，它有自己的 git 存储库。

Angular 应用程序构建需要大约 25 分钟，主要原因是端到端测试设置；这平均需要运行 12 分钟，占总构建时间的 47%。

其他使用 React 的团队有一个集成测试框架，该框架利用了以下几个方面:

> *“帮助将大量的 E2E 测试转化为更快速和确定性的集成测试”。*

出于这些原因，我开始为 Angular 做同样的事情，但在网上看时，我没有遇到太多的指导。这就是我写这篇文章的动机，来分享我是如何做到这一点的，希望它能帮助别人。

# 目标

正如背景故事部分所提到的，目标是在 Angular 中构建一个集成测试框架，它将:

1.  使用与单元测试相同的包:[jest](https://github.com/just-jeb/angular-builders/tree/master/packages/jest)with[ng neat/spectator](https://ngneat.github.io/spectator/)。

2.自动记录任何网络请求/响应，并能够为 CI/CD 重放它们。

3.允许使用[黑盒](https://en.wikipedia.org/wiki/Black-box_testing)风格以与端到端测试相同的方式编写测试。

# 包装

## 棱角分明的建设者——笑话建设者

[Angular Builders——Jest Builder](https://github.com/just-jeb/angular-builders/tree/master/packages/jest)允许用 Jest 而不是 Karma&Jasmine(Angular 中的默认设置)运行`ng test`(即单元测试)。

与 Aim 1 相关，这被重用来为集成测试创建一个新的[构建目标](https://angular.io/guide/workspace-config#configuring-builder-targets)。

## ngneat/旁观者

ngneat/spectator 用于使单元测试更容易编写，并允许查询类似于[测试库](https://testing-library.com/)的 DOM。

ngneat/spectator 有一个特殊的工厂，`createRoutingFactory`，可以用来测试路由(甚至提到了[集成测试](https://ngneat.github.io/spectator/docs/testing-with-routing#integration-testing-with-routertestingmodule))。

与 Aim 1 和 3 相关，这用于创建和查询测试中的 Angular 应用程序/与之交互。

## PollyJS

PollyJ 是由网飞创建的 JavaScript 库，正如他们的文档所述:

> 波利。JS 是一个独立的、与框架无关的 JavaScript 库，支持 HTTP 交互的记录、重放和存根。

这个方案使我们能够实现目标 2。

# 集成测试框架

利用[Angular Tutorial——Tour of Heroes](https://angular.io/tutorial/toh-pt6)作为一个应用程序来测试和组合上一节中提到的包，产生了我们将演练的这个集成测试框架。

完整的例子可以在 GitHub 上找到:[JonJam/angular _ Polly js _ playground](https://github.com/JonJam/angular_pollyjs_playground)。

## 集成测试目录

一个新的目录`integration-tests` 存在于`src` 文件夹之外，它包含了测试，以及支持测试所需的大部分配置。

## 集成测试的类型脚本配置

[ts config . integration . JSON](https://github.com/JonJam/angular_pollyjs_playground/blob/main/tsconfig.integration.json)扩展单元测试配置以包含`integration-tests`目录中的代码。

```
{
  "extends": "./tsconfig.spec.json",
  "include": [
    "integration-tests/**/*.ts",
  ],
  "compilerOptions": {
    "types": [
      "integration-tests/index.d.ts"
    ]
  }
}
```

## **角度整合测试建立目标**

一个新的构建目标`test-integration`已经被添加到 [angular.json](https://github.com/JonJam/angular_pollyjs_playground/blob/main/angular.json#L90) 中，它使用带有集成测试特定配置文件的 Jest Builder。

```
"test-integration": {
  "builder": "@angular-builders/jest:run",
  "options": {
    "configPath": "integration-tests/jest-integration.config.js",
    "tsConfig": "tsconfig.integration.json"
  }
}
```

## 集成测试 npm 脚本

在 [package.json](https://github.com/JonJam/angular_pollyjs_playground/blob/main/package.json) 中有两个新的 npm 脚本来运行集成测试。

```
"test:integration": "ng run angular.io-example:test-integration",
"test:integration:record": "POLLY_MODE=record ng run angular.io-example:test-integration"
```

`test:integration:record`将在`record`模式下运行集成测试，这意味着测试将发出实际的网络请求，然后在完成后，将记录保存为`.har`文件。

`test:integration`将在`replay`模式下运行集成测试，这意味着将使用记录而不是发出实际请求。这是您将在 CI/CD 管道中使用的内容。

## Jest 自定义环境

为了用 jest 设置 PollyJS，我们需要使用一个定制环境[jest-integration . environment . js](https://github.com/JonJam/angular_pollyjs_playground/blob/main/integration-tests/jest-integration.environment.js)，按照这个[指南](https://netflix.github.io/pollyjs/#/test-frameworks/jest-jasmine)。

```
// custom-environment
const SetupPollyJestEnvironment = require('setup-polly-jest/jest-environment-jsdom');

class JestIntegrationEnvironment extends SetupPollyJestEnvironment {
  constructor(config, context) {
    super(config, context);

    // Setting up testPath global for pollyContext.ts
    // Followed https://stackoverflow.com/questions/62995762/how-do-you-find-the-filename-and-path-of-a-running-test-in-jest
    this.global.testPath = context.testPath;
  }
}

module.exports = JestIntegrationEnvironment;
```

这扩展了由[gribnoysup/setup-Polly-jest](https://github.com/gribnoysup/setup-polly-jest)提供的设置用于 PollyJS 配置的`testPath`全局。

## PollyJS 设置

PollyJS 在 jest 配置中引用的 [pollyContext.ts](https://github.com/JonJam/angular_pollyjs_playground/blob/main/integration-tests/jest/setupFilesAfterEnv/pollyContext.ts) 中配置。

```
import { setupPolly } from 'setup-polly-jest';
import ***path*** from 'path';
import {PollyConfig} from '@pollyjs/core';
import {MODES} from '@pollyjs/utils';

function getPollyMode() : "record" | "replay" {
  let mode: PollyConfig['mode'] = MODES.*REPLAY*;

  switch (***process***.env['POLLY_MODE']) {
    case 'record':
      mode = MODES.*RECORD*;
      break;
    case 'replay':
      mode = MODES.*REPLAY*;
      break;
    case 'offline':
      mode = MODES.*REPLAY*;
      break;
  }

  return mode;
}

function getDefaultRecordingDir() {
  // This is setup using a custom jest environment
  const testPath: string = (***global*** as any).testPath;

  return path.relative(
    ***process***.cwd(),
    `${path.dirname(testPath)}/__recordings__`,
  );
}

const pollyContext = setupPolly({
    recordIfMissing: false,
    mode: getPollyMode(),
    // Having to use require imports due to https://github.com/gribnoysup/setup-polly-jest/issues/23
    adapters: [require('@pollyjs/adapter-xhr')],
    persister: require('@pollyjs/persister-fs'),
    persisterOptions: {
      fs: {
        recordingsDir: getDefaultRecordingDir(),
      },
      // Improve diff readability for git. See https://netflix.github.io/pollyjs/#/configuration?id=disablesortingharentries
      disableSortingHarEntries: true,
    },
    flushRequestsOnStop: true,
    recordFailedRequests: true,
    // Default level is warn. Useful to see PollyJs Recorded or Replayed logs
    logLevel: 'info',
    expiryStrategy: 'error',
    expiresIn: '14d',
    // Insulate the tests from differences in session data.
    //
    // See https://netflix.github.io/pollyjs/#/configuration?id=matchrequestsby for options.
    matchRequestsBy: {
      headers: false,
      body: false,
    },
  });

beforeEach(() => {
  // Grouping common requests so that they share a single recording.
  //
  // See https://netflix.github.io/pollyjs/#/server/route-handler?id=recordingname
  //
  // *TODO Add common requests.* // pollyContext.polly.server.any('/api/example').recordingName('Example');
});

***global***.pollyContext = pollyContext;
```

这是使用 PollyJS 中的[类型脚本示例](https://github.com/Netflix/pollyjs/blob/master/examples/typescript-jest-node-fetch/src/utils/auto-setup-polly.ts)和[Spotify/Polly-jest-preset](https://github.com/spotify/polly-jest-presets/blob/master/src/pollyContext.ts)放在一起的。

我鼓励你阅读 [PollyJS 配置文档](https://netflix.github.io/pollyjs/#/configuration)来理解所有的选项，但是总结一下这是在做什么:

*   它允许根据环境变量在`record`和`replay`模式之间切换。
*   它将网络请求和响应的记录保存到带有`__tests__`目录的`__recordings__`文件夹中(类似于 jest 快照)。
*   它每 14 天使记录过期一次，并在过期时导致构建出错。
*   它定义了请求匹配标准来忽略头部和主体内容。
*   它将指定的公共请求分组以共享单个记录。
*   它创建了一个可以在测试中引用的全局`pollyContext`。

## Jest 配置

[jest-integration . config . js](https://github.com/JonJam/angular_pollyjs_playground/blob/main/integration-tests/jest-integration.config.js)参考前面提到的一些文件，为集成测试配置 jest 环境。

```
// This is merged with default config as per https://github.com/just-jeb/angular-builders/tree/master/packages/jest#builder-options

// Test environment copied from: https://netflix.github.io/pollyjs/#/test-frameworks/jest-jasmine?id=supported-test-runners
module.exports = {
  testEnvironment: "<rootDir>/integration-tests/jest-integration.environment.js",
  testMatch: ["<rootDir>/integration-tests/__tests__/**/*.spec.ts"],
  setupFilesAfterEnv: [
    "<rootDir>/integration-tests/jest/setupFilesAfterEnv/pollyContext.ts",
  ]
};
```

## utils/testUtils.ts

[testUtil.ts](https://github.com/JonJam/angular_pollyjs_playground/blob/main/integration-tests/utils/testUtils.ts) 包含助手功能，例如启动应用程序和导航到初始页面，这些功能在集成测试中被重用。

```
import {createRoutingFactory, SpectatorRouting} from '@ngneat/spectator/jest';
import {***TestBed***} from '@angular/core/testing';
import {NgZone} from '@angular/core';

import {AppComponent} from '../../src/app/app.component';
import {AppModule} from '../../src/app/app.module';

const createComponent = createRoutingFactory({
  component: AppComponent,
  imports: [AppModule],
  // Setting to false as per https://ngneat.github.io/spectator/docs/testing-with-routing#integration-testing-with-routertestingmodule
  // to perform an integration test.
  stubsEnabled: false,
  detectChanges: false
});

function startApp() : SpectatorRouting<AppComponent> {
  // Create app
  return createComponent();
}

async function navigateToInitialPage(spectator: SpectatorRouting<AppComponent>, url: string) : Promise<void> {
  // Navigate to page under test
  const ngZone = ***TestBed***.inject(NgZone);
  await ngZone.run(async () => {
    await spectator.router.navigateByUrl(url);
  });

  // Wait for any HTTP and UI updates
  await ***global***.pollyContext.polly.flush();
  spectator.detectChanges();
}

export { startApp, navigateToInitialPage };
```

# 把所有的放在一起

让我们用一个示例测试来实践这一点: [heroes.spec.ts](https://github.com/JonJam/angular_pollyjs_playground/blob/main/integration-tests/__tests__/heroes.spec.ts) 。

```
import {***byRole***} from '@ngneat/spectator/jest';

import {navigateToInitialPage, startApp} from '../utils/testUtils';

describe('Heroes', () => {
  it('renders a list of heroes', async () => {
    // Start app and navigate to heroes page
    const spectator = startApp();
    const url = `heroes`;
    await navigateToInitialPage(spectator, url);

    // Update view with heroes data
    await ***global***.pollyContext.polly.flush();
    spectator.detectChanges();

    // ASSERT
    const heroes : HTMLLIElement[] = spectator.queryAll(***byRole***('listitem'));

    const linkContents = heroes.map(li => {
      const link = li.querySelector('a');

      if (link === null || link.textContent === null) {
        return "";
      }

      return link.textContent.trim();

    });

    const expected =  [
      '1 Leanne Graham',
      '2 Ervin Howell',
      '3 Clementine Bauch',
      '4 Patricia Lebsack',
      '5 Chelsey Dietrich',
      '6 Mrs. Dennis Schulist',
      '7 Kurtis Weissnat',
      '8 Nicholas Runolfsdottir V',
      '9 Glenna Reichert',
      '10 Clementina DuBuque'
    ];

    expect(linkContents).toEqual(expect.arrayContaining(expected));
  });
});
```

这项测试:

*   启动应用程序。
*   导航到`/heroes`。
*   等待网络请求和 UI 更新。
*   断言显示预期的列表。

首先，我们使用`test:integration:record`创建记录来运行这个测试。

```
➜  angular_pollyjs_playground git:(main) yarn test:integration:record
yarn run v1.22.15
$ POLLY_MODE=record ng run angular.io-example:test-integration
Warning: 'no-cache' option has been declared with a 'no' prefix in the schema.Please file an issue with the author of this package.
Warning: 'noStackTrace' option has been declared with a 'no' prefix in the schema.Please file an issue with the author of this package.
Determining test suites to run...
ngcc-jest-processor: running ngcc
Request: GET [https://jsonplaceholder.typicode.com/users](https://jsonplaceholder.typicode.com/users)
Response: Recorded ➞ GET [https://jsonplaceholder.typicode.com/users](https://jsonplaceholder.typicode.com/users) 200 • 447ms
 PASS  integration-tests/__tests__/heroes.spec.ts (8.964 s)
  Heroes
    ✓ renders a list of heroes (502 ms)Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        15.446 s
Ran all test suites.
✨  Done in 32.81s.
```

PollyJS 日志有助于显示记录了哪些请求和响应。现在我们已经有了一些记录，我们可以在`replay`模式下运行`test:integration`:

```
➜  angular_pollyjs_playground git:(main) ✗ yarn test:integration       
yarn run v1.22.15
$ ng run angular.io-example:test-integration
Warning: 'no-cache' option has been declared with a 'no' prefix in the schema.Please file an issue with the author of this package.
Warning: 'noStackTrace' option has been declared with a 'no' prefix in the schema.Please file an issue with the author of this package.
Determining test suites to run...
ngcc-jest-processor: running ngcc
Request: GET [https://jsonplaceholder.typicode.com/users](https://jsonplaceholder.typicode.com/users)
Response: Replayed ➞ GET [https://jsonplaceholder.typicode.com/users](https://jsonplaceholder.typicode.com/users) 200 • 2ms
 PASS  integration-tests/__tests__/heroes.spec.ts
  Heroes
    ✓ renders a list of heroes (53 ms)Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        2.087 s, estimated 9 s
Ran all test suites.
✨  Done in 4.55s.
```

那都是乡亲:)

# 逮到你了

当我真正这么做的时候，我遇到了一些可能有用的事情。

## 具有自签名证书的服务器

如果你的测试运行在一个使用自签名证书的后端，你可能会遇到测试失败的错误，比如`Error: self signed certificate`。

为了解决这个问题，我发现了这个[玩笑问题](https://github.com/facebook/jest/issues/8449#issuecomment-781661413)，并采取了以下措施:

*   从 jest 复制`JSDOMEnvironment`(确保复制正确的版本)到应用程序中。
*   修改了类构造函数中的`JSDOM` 构造，以在创建`ResourceLoader.`时指定`strictSSL:false`
*   更改`jest-integration.environment.js`以使用该自定义`JSDOMEnvironment.`

## ng neat/观众和活动路线

通常为了获得 Angular 中的活动路径，我们使用[ActivatedRoute](https://angular.io/api/router/ActivatedRoute)；但是当使用 ngneat/spectator 的`createRoutingFactory`(即使有`stubsEnabled: false)`值也是存根`ActivatedRouteStub`；你可以在这里看到代码。

这给测试中的应用程序带来了问题，因为某些功能没有实现(访问`snapshot`和`firstchild`属性)。

一个解决方法是使用`router.routerstate.root`而不是`ActivatedRoute`，因为这是 Angular 在内部使用的(参见 [router_module](https://github.com/angular/angular/blob/14.2.3/packages/router/src/router_module.ts#L50) 和 [provide_router](https://github.com/angular/angular/blob/14.2.3/packages/router/src/provide_router.ts) )。

## 相对请求的基本 URL

测试中的应用程序使用了相对请求，即`/api/example`，因为它希望在与 API 相同的域中运行。

当作为集成测试运行时，我发现正在向`https://github.com/just-jeb/angular-builders/api/example`发出请求。

为了解决这个问题，我必须在`jest-integration.config.js`中添加以下内容来设置基本 url:

```
testEnvironmentOptions: {
  url: "https://localhost",
},
```