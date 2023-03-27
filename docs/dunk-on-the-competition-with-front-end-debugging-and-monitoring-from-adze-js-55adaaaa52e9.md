# 通过 Adze JS 的前端调试和监控，在竞争中脱颖而出

> 原文：<https://itnext.io/dunk-on-the-competition-with-front-end-debugging-and-monitoring-from-adze-js-55adaaaa52e9?source=collection_archive---------2----------------------->

## 它的发音是“广告”…

![](img/5c7173ee893d08daab138889f916ef07.png)

照片由[哈里·坎宁安](https://unsplash.com/@harrycunningham?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

最近，我有机会开始一个新的绿地项目，我们可以选择闪亮的新工具和框架。首先，我不想低估这种感觉有多糟糕；很遗憾我们不能经常这样做。无论如何，对于前端日志，我记得听一位前同事提到过这个开源项目。

[](https://adzejs.com/) [## Adze —更好的 JavaScript 日志记录

### 打开开发控制台进行快速启动演示→将日志写成方法链。Adze 的方方面面都在你的掌控之中…

adzejs.com](https://adzejs.com/) 

# 它是做什么的？

乍看之下，Adze JS 是一个浏览器控制台 API 的包装器，它拥有你熟悉和喜爱的所有相同的命令。永恒的热门歌曲，如`console.log`或苦乐参半的`console.error`，甚至是邪教经典`console.dir`。你知道这些，所以我不会说太多。然而，有一些更高层次的 Adze 概念真正有助于防止您的代码变成一个巨大的面条怪物。以下是对这些概念的简要描述。

## **棚子**

一个集中的地方(在您的代码中)来缓存您的日志，提供全局配置，添加任何过滤(即忽略调试日志级别)或覆盖(即在生产中记录未处理的异常时不包括 fire 表情):

```
import { adze, createShed } from 'adze';

const shed = createShed();

shed.addListener([1, 2, 3], ({ definition, timestamp, args }, render, printed) => {
  if (printed) {
    const log = {
      level: definition.level,
      name: definition.levelName,
      timestamp: timestamp.utc,
      args,
    };
    // You could now write it to a file, send it to a rest API, etc.
  }
});
```

## **工厂**

所有的记录都是从 Adze 的工厂开始的。我还没有机会全部使用它们，但主要的一个被简单地称为`adze`:

```
const cfg = { useEmoji: true };

adze(cfg).log('A log with emoji enabled.');
```

## **主题化**

Adze 公开了一个配置，您可以将它传递给默认的 adze 工厂。它允许您使用类似于 CSS 的属性来更改日志样式:

```
const config = {
  logLevel: 8,
  useEmoji: true,
  logLevels: {
    log: {
      emoji: '🔥',
      style: 'padding-right: 5px; background: red; color: white; border-color: black;',
      terminal: ['bgRed', 'white'],    
    },
  }
};

adze(config).log("I'm red");
```

## **修改器**

顾名思义，您可以修改将要记录的任何内容。像`timestamp`、`count`和`namespace`这样的函数可以提供更多关于正在记录的内容的上下文。

```
adze().label('my-label').timestamp.log('my message');// Output: Log(1) 2022-10-25T16:33:43-4:00 [my-label] my message
```

## 链接

正如所承诺的，有一些更高层次的概念使 Adze 与众不同，这是我认为的主要概念。一旦你有了一个工厂或修改器，它就变成了一个可链接的 API，这意味着你可以这样做:

```
adze().label('my-label').namespace('my-ns').timestamp.count.info("my info");adze().label('my-label').namespace('my-ns').timestamp.count.info("my info");// Output: 
// Info(1) 2022-10-25T16:33:43-4:00 #my-ns [my-label] (Count: 1) my info
// Info(1) 2022-10-25T16:33:44-4:00 #my-ns [my-label] (Count: 2) my info
```

# 开发人员如何在典型的应用程序中使用它？

我将简要介绍我是如何在 React Native + Typescript 应用程序中使用 Adze JS 的。首先，我将它作为一个依赖项安装:

```
# NPM
$ npm install --save adze# Yarn
$ yarn add adze
```

在我的例子中，我将创建一个处理所有日志逻辑并可扩展的类。

```
import { adze, createShed, Constraints } from 'adze';
import { Log } from 'adze/dist/log';
import { Shed } from 'adze/dist/shed';

export default class LoggerClass {
  shed: Shed;
  logger: Log<Constraints>;

  constructor() {
    this.shed = createShed();

    this.shed.addListener('*', (data, render, printed) => {
      if (printed) {
        // TODO: Send to AWS CloudWatch here
      }
    });

    const sealed = adze({
      logLevel: 8,
      useEmoji: true,
    }).seal();

    this.logger = sealed().namespace(new.target.name);
  }

  error(message: string): Error {
    this.logger.error(message);
    throw new Error(message);
  }

  logGroup(messages: string[]) {
    if (messages.length < 1) return;
    if (messages.length === 1) return this.logger.group.log(messages[0]);
    const [first, ...rest] = messages;
    this.logger.groupCollapsed.log(first);
    rest.forEach(this.logger.log);
    this.logger.groupEnd.log();
  }

  logTime(message: string) {
    this.logger.timeNow.log(message);
  }

  logAssert(assertion: boolean, message: string) {
    this.logger.assert(assertion).log(message);
  }

  logTable(tabularData: [{ [key: string]: any }]) {
    this.logger.table.log(tabularData);
  }
}
```

然后，我可以扩展这个类并进行一些日志记录，而不必每次都进行大量的设置:

```
import Deck from "./Deck";
import Square from "../types/Square";
import LoggerClass from "./LoggerClass";
import { CardArray, CardName } from "../types/CardName";

const DECK_SIZE = 54;
const NUM_ROWS = 4;
const NUM_COLUMNS = 4;

export default class Board extends LoggerClass {
  private squares: Square[];

  constructor() {
    super();

    if (CardArray.length !== DECK_SIZE) this.error('Not Enough Cards');
    this.squares = [];
    const arr: CardName[] = Deck.sortRandomly<CardName>([...CardArray]);
    for (let row = 1; row <= NUM_ROWS; row++) {
      for (let column = 1; column <= NUM_COLUMNS; column++) {
        this.squares.push({
          row,
          column,
          cardType: arr.pop(),
        });
      }
    }
  } ...}
```

# 这太棒了，有办法为开源项目做贡献吗？

如果你一直梦想在简历中加入开源项目的贡献，现在是你的机会了。

[](https://github.com/AJStacy/adze) [## GitHub - AJStacy/adze:一个用于塑造 JavaScript 日志的库。

### 请访问我们在 adzejs.com 的官方文件。Adze -一种切割工具...主要用于塑造木材…

github.com](https://github.com/AJStacy/adze) 

这里有一个未决问题的列表:[https://github.com/AJStacy/adze/issues](https://github.com/AJStacy/adze/issues)。请务必遵循投稿指南。创建一个分支，并在准备就绪时打开一个合并请求。