# 用 React Native 在 5 分钟内启用 TypeScript

> 原文：<https://itnext.io/enabling-typescript-with-react-native-in-minimal-steps-839932b1a853?source=collection_archive---------5----------------------->

![](img/44926d9f606fd813e616c90525d80ac6.png)

图片来自 Unsplash.com

这篇教程很大程度上受到了 React Native 博客的启发。

这里的目标是有一个最小的设置，让我们在一个现有的项目上启动并运行 TypeScript，所以有些东西已经被排除，而其他的东西被包括在内，供 TS 运行。

**1。从安装依赖项开始:**

```
yarn add -dev typescript react-native-typescript-transformer [@types/react](http://twitter.com/types/react) [@types/react-native](http://twitter.com/types/react-native)
```

**2。创建一个类型脚本配置:**

```
yarn tsc --init --pretty --jsx react
```

这将在项目的根目录下创建一个`tsconfig.json`文件。

**3。在** `**tsconfig.json**` **:** 中进行修改

搜索`allowSyntheticDefaultImports`，将其设置为`true`启用。

现在，让我们把`outDir`改成`outputs`。这使得我们的 trans filed 文件在一个文件夹中，而不是让他们旁边的非 trans filed 的。

现在，为了快速进入无错误状态，我们想通过添加`”skipLibCheck”: true`来跳过对默认库声明文件的类型检查。

所以，我们的`tsconfig.json`应该包括

```
{
 “compilerOptions”: {
 // …
 “skipLibCheck”: true,
 “outDir”: “outputs”,
 “allowSyntheticDefaultImports”: true
 }
}
```

**4。配置 React 本机脚本转换器。**

让我们创建保存这些设置的文件，通过执行`touch rn-cli.config.js`，打开它并添加:

```
module.exports = {
 getTransformModulePath() {
 return require.resolve(‘react-native-typescript-transformer’);
 },
 getSourceExts() {
 return [‘ts’, ‘tsx’];
 },
};
```

5.就是这样！

在终端中运行`tsc -w`，一切都应该正常工作！

—

自由职业的移动和网络工程师🚀。我是 [*范围的播客主持人之一🎤，开发者和项目经理就不同的软件开发主题发表见解。*](http://scopecreeperspodcast.com) [*哥本哈根 React Native Meetup*](https://www.meetup.com/React-Native-CPH/)*协办单位👥一名志愿者帮助外国人融入丹麦。一个绝对的通过代码和同理心创造东西的粉丝。喜欢练拳击🥊。*

*在推特上关注我*[*@ Pedro foraday*](http://twitter.com/pedroforaday)*。*