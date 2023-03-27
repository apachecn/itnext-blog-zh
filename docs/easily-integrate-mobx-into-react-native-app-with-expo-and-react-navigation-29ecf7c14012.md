# 通过 Expo 和 React 导航轻松将 Mobx 集成到 React 本机应用程序中

> 原文：<https://itnext.io/easily-integrate-mobx-into-react-native-app-with-expo-and-react-navigation-29ecf7c14012?source=collection_archive---------0----------------------->

![](img/f7c94f533fcd823ad39aaf4584f3bd32.png)

照片由 [Adi Goldstein](https://unsplash.com/@adigold1?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

让我们假设您的项目运行的是最新的 expo 版本。

## 属国

让我们为 mobx 及其装饰器添加依赖项😍

```
npm i mobx mobx-react --save
npm i babel-plugin-transform-decorators-legacy --save-dev
```

## 巴比伦式的城市

打开巴别塔配置(。根文件夹中的 babelrc)并将其编辑成这样

```
{
   "presets": ["babel-preset-expo"],
   "plugins": [
      [
         "@babel/plugin-proposal-decorators",
         {
            "legacy": true
         }
      ]
   ]
}
```

## Mobx 商店

是时候开店了。用`your-store.js`或`your-store.ts`文件创建一个 **mobx** 文件夹。我在我的项目中使用 TypeScript，所以它是`store.ts`。还有谁爱 TypeScript？🙋

## 商店编码

向您的存储添加一些代码。这里我们有一个非常简单的属性和更新它的动作。酷！

## 连接商店

是时候将商店连接到应用程序了。导航到你的`App.tsx`，用`mobx`中的`<Provider>`组件包裹`<AppContainer />`。

如果你没有`<AppContainer/>`只是稍微重构一下你的代码，看这里的。

## 快到了！订阅商店。

让我们订阅我们创建的可观察商店。

## 更新商店

更新属性的时间到了。

# 搞定了。

小提示:我喜欢它们提供的 TypeScript 和 intellisense 接口，所以我通常创建接口 *IObservableStoreProps* 并在组件 Props 中扩展它。

享受你的编码！