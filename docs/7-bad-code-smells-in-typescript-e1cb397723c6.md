# 7 Typescript 中的不良代码味道

> 原文：<https://itnext.io/7-bad-code-smells-in-typescript-e1cb397723c6?source=collection_archive---------0----------------------->

## 让你的代码更加可重用、可读和可重构

![](img/329ca28dfb242c8161bdefd43ba610cd.png)

肖恩·托马斯在 [Unsplash](https://unsplash.com/photos/44RX_wJlmE8?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

作为一名软件工程师，你可能听说过或经历过代码味道的概念。代码气味是程序源代码的任何特征，可以指出更深层次的问题。

简单地说，代码气味通常不是 bug，它们不是技术上不正确的。但是它们指出了设计中的弱点，这些弱点可能会减慢开发速度或增加错误的风险。我们的目标是编写可读、可重用和可重构的软件。

在本文中，我将讨论 Typescript 中的 7 种糟糕的代码味道。

## 1.使用可搜索的名称

我们读的代码总是比写的多。重要的是，我们编写的代码是可读和可搜索的。

不良做法:

```
// What is 36000000 for?
setTimeout(restart, 36000000);
```

请改为这样做:

```
// Declare them as capitalized named constants.
const MILLISECONDS_PER_HOUR = 60 * 60 * 1000; // 36000000

setTimeout(restart, MILLISECONDS_PER_HOUR);
```

## 2.不要添加不必要的上下文

如果你的类/类型/对象名告诉你一些事情，不要在你的变量名中重复。

不良做法:

```
type User = {
  userName: string;
  userLastName: string;
  userAge: number;
}

function print(user: User): void {
  console.log(`${user.userName} ${user.userLastName} (${user.userAge})`);
}
```

请改为这样做:

```
type User = {
  name: string;
  lastName: string;
  age: number;
}

function print(user: User): void {
  console.log(`${user.name} ${user.lastName} (${user.age})`);
}
```

## 3.函数参数(理想情况下为 2 个或更少)

如果我们的函数有两个以上的参数，那么它们做得太多了。一个或两个论点是理想的，容易测试，如果可能的话，应该避免三个。

不良做法:

```
type UserStatus = 'online' | 'offline';
function createUser(name: string, lastName: string, age: number, status: UserStatus) {
  // ...
}

createUser('Gapur', 'Kassym', 29, 'online');
```

请改为这样做:

```
type UserStatus = 'online' | 'offline';
type User = { name: string, lastName: string, age: number, status: UserStatus };

function createUser(user: User) {
  // ...
}

createUser({
  name: 'Gapur',
  lastName: 'Kassym',
  age: 29,
  status: 'online'
});
```

## 4.！某物和功能()

这是(ab)使用短路评估。但是很难调试，并且在许多情况下很难找到实际上是什么在起作用:

```
myVariable && myFunction();
```

这相当于:

```
if (myVariable) {
    myFunction()
}
```

不良做法:

```
const userAge = 19;
const ageLimit = 18;

userAge >= ageLimit && voteForPresident();
```

请改为这样做:

```
const userAge = 19;
const ageLimit = 18;

if (userAge >= ageLimit) {
  voteForPresident();
}
```

## 5.使用 Object.assign 或 spread 运算符设置默认对象

不良做法:

```
type User = {
  name: string | undefined,
  lastName: string | undefined,
  age: number | undefined,
};

function createUser(user: User) {
  user.name = user.name || 'User';
  user.lastName = user.lastName || '';
  user.age = user.age || 18;
  // ...
}

createUser({ name: 'Gapur' });
```

请改为这样做:

```
type User = {
  name: string | undefined,
  lastName: string | undefined,
  age: number | undefined,
};

function createUser(user: User) {
  const userWithDefaultValues = {
    name: 'User',
    lastName: '',
    age: 18,
    ...user,
  };
  // ...
}

createUser({ name: 'Gapur' });
```

## 6.避免副作用

重点是避免常见的错误，比如在没有任何结构的对象之间共享状态，使用任何人都可以编写的可变数据类型。

不良做法:

```
// Global variable referenced by following function.
let name = 'Robert C. Martin';

function toBase64() {
  name = btoa(name);
}

toBase64();
// If we had another function that used this name, now it'd be a Base64 value

console.log(name); // expected to print 'Robert C. Martin' but instead 'Um9iZXJ0IEMuIE1hcnRpbg=='
```

请改为这样做:

```
const name = 'Robert C. Martin';

function toBase64(text: string): string {
  return btoa(text);
}

const encodedName = toBase64(name);
console.log(name);
```

## 7.避免消极条件句

不良做法:

```
type UserStatus = 'online' | 'offline';

function isUserNotOnline(status: UserStatus): boolean {
  // ...
}

if (isUserNotOnline(status)) {
  // ...
}
```

请改为这样做:

```
type UserStatus = 'online' | 'offline';

function isUserOnline(status: UserStatus): boolean {
  // ...
}

if (!isUserOnline(status)) {
  // ...
}
```

# 结论

感谢阅读，希望这篇文章对你有用。编码快乐！

# 资源

[](https://startup-cto.net/10-bad-typescript-habits-to-break-this-year/) [## 今年要改掉的 10 个打字坏习惯

### TypeScript 和 JavaScript 在过去几年中稳步发展，我们在过去几年中养成的一些习惯…

startup-cto.net](https://startup-cto.net/10-bad-typescript-habits-to-break-this-year/) [](https://javascript.plainenglish.io/clean-code-in-typescript-a183d43f3bf0) [## 清除类型脚本中的代码

### 介绍

javascript.plainenglish.io](https://javascript.plainenglish.io/clean-code-in-typescript-a183d43f3bf0) [](https://github.com/labs42io/clean-code-typescript) [## GitHub-labs 42 io/Clean-Code-typescript:适用于 TypeScript 的干净代码概念

### 适用于 TypeScript 的干净代码概念。灵感来自干净代码 javascript。软件工程原理，来自…

github.com](https://github.com/labs42io/clean-code-typescript)