# 解释的 TypeScript 实用工具类型

> 原文：<https://itnext.io/typescript-utility-types-explained-6584b0306bd1?source=collection_archive---------4----------------------->

## 以打字打的文件

## 部分、必需、只读、挑选、省略、记录、非空、提取、排除类型

![](img/e17c5b355c27def6e49bcb31ab973e10.png)

[Ash Edmonds](https://unsplash.com/@badashproducts?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

在 TypeScript 中，有多种内置实用工具(默认)类型。知道它们的存在是件好事，因为它们在某些情况下很方便。有些比其他的更容易理解。我将用简单明了的方式来解释它们，这样每个人都能理解。

## 1-部分

第一个是`Partial`，这可以使一个类型中的所有属性都是可选的，而这些属性原本应该是必需的。

```
type Worker = {
  name: string;
  profession: string;
}// Not defining 'profession' is allowed
const worker: Partial<Worker> = {
  name: 'Jeroen' 
}
```

## 2 —必需

Partial 的反义词是`Required`，它不是让属性可选，而是让属性必选。

```
type Worker = {
  name?: string;
  profession?: string;
}// You should defining 'name' and 'profession'
const worker: Required<Worker> = {
  name: 'Jeroen',
  profession: 'Developer',
}
```

## 3-只读

使用`Readonly`，你可以防止属性被重新分配。

```
type Worker = {
  name: string;
  profession: string;
}const worker: Readonly<Worker> = {
  name: 'Jeroen',
  profession: 'Developer',
}worker.name = 'Bob'; // Not allowed
```

## 4 —选择

一个更高级的实用程序类型是`Pick`，它可以用来包含一些属性。

```
type Worker = {
  name: string;
  profession: string;
  isWorking: boolean;
}// Only 'name' and 'isWorking' are included
const worker: Pick<Worker, 'name' | 'isWorking'> = {
  name: 'Jeroen',
  isWorking: true,
}
```

## 5 —省略

与选择类型相反的方式叫做`Omit`，你排除你不想使用的属性。

```
type Worker = {
  name: string;
  profession: string;
  isWorking: boolean;
}// 'profession' is now excluded
const worker: Omit<Worker, 'profession'> = {
  name: 'Jeroen',
  isWorking: true,
}
```

## 6 —记录

`Record`类型允许您定义一个对象，其中键的类型与其属性值不同。

```
type Worker = 'developer' | 'architect'
type Person = { name: string }// a worker object has now 'developer' or 'architect' as keys
const worker: Record<Worker, Person> = {
  'developer': { name: 'Jeroen' },
  'architect': { name: 'Bob' },
}
```

## 7-不可空

从您的值中排除`null`和`undefined`的一个简单方法是使用`NonNullable`类型。

```
// Returns only 'Jeroen'
const worker: NonNullable<'Jeroen', undefined, null>;
```

## 8 —摘录

在这种情况下，要提取`keyof`类型的值，您可以使用`Extract`来只包含存在于两种给定类型中的键。

```
type Worker = {
  name: string;
  age: string;
  isWorking: boolean
}type Person = {
  name: string;
  age: string;
}// 'name' or 'age' is both in type Person and Worker therefore only allowed 
const worker: Extract<keyof Worker, keyof Person> = 'name' || 'age';
```

## 9-排除

在这种情况下，要再次排除`keyof`类型的值，您可以使用`Exclude`来排除在两个给定类型中重复的键。

```
type Worker = {
  name: string;
  age: string;
  isWorking: boolean
}type Person = {
  name: string;
  age: string;
}// 'isWorking' is not in type Person and therefor only allowed 
const worker: Exclude<keyof Worker, keyof Person> = 'isWorking';
```