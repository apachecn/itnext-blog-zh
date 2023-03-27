# 使用 Angular 中的 CSS 变量动态设置应用程序主题

> 原文：<https://itnext.io/css-vars-in-use-while-dynamically-setting-app-theme-angular-485ff17570f8?source=collection_archive---------1----------------------->

*   声明:css 变量很棒，但我在这篇文章中使用它们的方式不是最好的“角度方式”，因为我使用 elementRef.nativeElement 而不是 rendere2，目前(2019 年 2 月)似乎是**渲染器**。setStyle()不支持 css 变量..

```
**this**.**renderer**.setStyle(**this**.**elementRef**.**nativeElement**, key, theme[key]);
```

**/

最近，我开始在我的小项目(一个类似 firebase + firestore 上的可拖动任务板的“trello”)上使用 css 自定义属性(css 变量)。

真的是又好看又轻松！我实现了一个应用程序“主题化”功能，只需点击一个切换按钮，就可以动态地为应用程序提供不同的外观和感觉(目前只有 2 个主题可用，硬编码——接下来是让用户可以选择设置任何他想要的颜色！使用变量的力量！).

CSS 变量/自定义属性包括:

> 以`--`为前缀的属性名，如`--example-name`，代表*自定义属性*，其中包含一个值，该值可以在使用`[var()](https://developer.mozilla.org/en-US/docs/Web/CSS/var)`函数的其他声明中使用

示例:

```
Declare: :root {
  --first-color: #488cff;
  --second-color: #ffff8c;
}Use:.some-class {
  background-color: var(--first-color);
  color: var(--second-color);
}
```

角度单位:

我做了一个 [stackblitz repo](https://stackblitz.com/edit/angular-css-vars-for-theme) 展示了如何在 Angular:

我创建了一个主题模块，它导出了 appTheme 指令(这里有点夸张)，appTheme 用于动态地更改应用程序主题，只要它通知这样做(添加一个主题反应服务)。

单击“更改主题”链接将通知指令它应该更改活动主题。

主题更新是通过为自定义 css 属性(也称为 css 变量)设置一个新值来完成的

仅此而已:elem nt . style . set property('—background '，' Alice blue ')；