# 棱角分明的开发者的 JSX

> 原文：<https://itnext.io/jsx-for-angular-developers-23f9d1f21259?source=collection_archive---------2----------------------->

## 在 Stencil 或 React 中为 Angular 开发人员简要介绍 JSX

![](img/eb4a12807c111f2b588ccddd8edeb58e.png)

由[在](https://unsplash.com/@maelrenau?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) [Unsplash](https://unsplash.com/s/photos/free?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的雷诺

我每天分享[一个窍门](https://medium.com/@david.dalbusco/one-trick-a-day-d-34-469a0336a07e)直到 2020 年 4 月 19 日新冠肺炎隔离期结束。离希望中的好日子还有 16 天。

起初，当我在用[模板](https://stenciljs.com)开发我的第一个 Web 组件时发现了 [JSX](https://www.typescriptlang.org/docs/handbook/jsx.html) 语法时，我并不那么喜欢它。我错过了[有角度的](https://angular.io) HTML 模板。

时下？将来我可能会再次改变我的想法，但是在开发了像 [DeckDeckGo](https://deckdeckgo.com) 这样的生态系统，甚至学会了 [React](https://reactjs.org) 之后，我可以肯定地说，我实际上感觉完全相反，我爱 JSX ❤️.甚至可能更多，因为我每周都在开发 Angular 客户的项目。

这就是为什么我有这个想法写一个非常简短的，我希望初学者友好的介绍 JSX，因为在模板或反应角度的开发人员使用。

# JSX 与 HTML 模板

如果你写一个 Angular 应用程序，通常是将你的组件分层，甚至可能是三个独立的文件:代码(类型脚本)，样式(CSS)和模板(HTML，GUI)。

```
import {Component} from '@angular/core';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.component.html',
  styleUrls: ['./my-component.component.scss']
})
export class MyComponentComponent {

}
```

和相关模板:

```
<div>Hello, World!</div>
```

使用 JSX，不管是 Stencil 还是 React，你也有这种分离的问题，但是你不会把你的模板和代码分离到两个独立的文件中。所有的东西通常都打包在一个文件中，甚至在同一个`class`或`function`中。

关注点的分离发生在代码端。如果你有一个`class`，你将不得不公开一个方法`render()`，它返回应该呈现的内容。简而言之:“一个呈现你的 HTML 代码的方法”。

```
import {Component, h} from '@stencil/core';

@Component({
  tag: 'my-component',
  styleUrl: 'my-component.css'
})
export class MyComponent {

  render() {
    return <div>Hello, World!</div>;
  }

}
```

如果你有一个`function`，那么你将有一个遵循相同行为的`return`方法，而不是`render`。

```
import React from 'react';

const MyComponent: React.FC = () => {

    return (
        <div>Hello, World!</div>
    );
};

export default MyComponent;
```

Stencil 和 React 都支持`class`或`function`。我认为，由于使用和引入了`Hooks`，最后一种类型在 React 中变得非常流行，我不打算在本文中介绍。如果你有兴趣在一个单独的职位，平我！我还有很多帖子要写，以完成我的挑战😆。

还要注意，在本文的其余部分，我将使用`class`显示模板示例，使用`functions`显示 React 示例。

# 根元素

一个重要的区别是根元素的概念。在 Angular 中，你并不真正关心是否。如果您的模板包含一个或多个根元素，它在任何情况下都可以编译。

```
<div>Hello, World!</div>

<div>
  <p>Salut</p>
  <p>Hallo</p>
</div>
```

相反，在 JSX，这很重要。应该开发您的组件来处理这种情况。

因此，我们的第一个解决方案可能是将我们的孩子分组到一个 HTML 节点下。

```
import {Component, h} from '@stencil/core';

@Component({
  tag: 'my-component',
  styleUrl: 'my-component.css'
})
export class MyComponent {

  render() {
    return <div>
      <div>Hello, World!</div>

      <div>
        <p>Salut</p>
        <p>Hallo</p>
      </div>
    </div>;
  }

}
```

这是可行的，但这会导致在我们的 DOM 中添加一个不需要的`div`标签，即父标签。这就是为什么 Stencil 和 React 对这个问题都有各自相似的解决方案。

在 Stencil 中，你可以使用一个`Host`元素。

```
import {Component, h, Host} from '@stencil/core';

@Component({
  tag: 'my-component',
  styleUrl: 'my-component.css'
})
export class MyComponent {

  render() {
    return <Host>
      <div>Hello, World!</div>

      <div>
        <p>Salut</p>
        <p>Hallo</p>
      </div>
    </Host>;
  }

}
```

在 React 中，你可以使用所谓的[片段](https://reactjs.org/docs/fragments.html)。

```
import React from 'react';

const MyComponent: React.FC = () => {

    return (
        <>
            <div>Hello, World!</div>

            <div>
                <p>Salut</p>
                <p>Hallo</p>
            </div>
        </>
    );
};

export default MyComponent;
```

最后，在 Stencil 中，如果你不想使用这样的容器，你可以返回一个`array`元素。但是我觉得，主要是因为样式的原因，到目前为止我更经常使用上面的解决方案。

```
import {Component, h} from '@stencil/core';

@Component({
  tag: 'my-component',
  styleUrl: 'my-component.css'
})
export class MyComponent {

  render() {
    return [
      <div>Hello, World!</div>,
      <div>
        <p>Salut</p>
        <p>Hallo</p>
      </div>
    ];
  }

}
```

# 状态和属性

在 Angular `public`中，变量是模板中使用的变量，任何变化都会触发新的渲染(“变化应用于 GUI”)。

制造的变量`private`是组件内部使用的变量，不需要新的渲染。

此外，还有[输入](https://angular.io/api/core/Input)装饰器，用于将变量作为组件的属性公开。

```
import {Component, Input} from '@angular/core';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.component.html',
  styleUrls: ['./my-component.component.scss']
})
export class MyComponentComponent {

  @Input()
  count = 0;

  odd = false;

  private even = false;

  inc() {
    // Render again
    this.count++;
    this.odd = this.count % 2 === 1;

    // Do not trigger a new render
    this.even = this.count % 2 === 0;

}
```

和相应的模板:

```
<div>Hello, World!</div>

<div>{{odd}} {{count}}</div>
```

在 JSX，你会发现相同的方法，但分为两类，`state`和`properties`，对于这两类，任何改变都会触发组件的新渲染。另一方面，如果你有一个变量不是这两者之一，那么就不会再次触发渲染。

`properties`是与`@Input()`字段相对应的概念，这些是组件的公开属性。

`states`是一种没有被标记为输入的角度`public`变量。

具体来说，在模版中你使用`decorator`来达到这个目的。

```
import {Component, h, Host, Prop, State} from '@stencil/core';

@Component({
  tag: 'my-component',
  styleUrl: 'my-component.css'
})
export class MyComponent {

  @Prop()
  count = 0;

  @State()
  private odd = false;

  even = false;

  inc() {
    // Render again
    this.count++;
    this.odd = this.count % 2 === 1;

    // Do not trigger a new render
    this.even = this.count % 2 === 0;
  }

  render() {
    return <Host>
        <div>{this.odd} {this.count}</div>
      </Host>
    ;
  }

}
```

在 React 函数中，你将使用`hooks`来处理状态，使用`interfaces`来声明你的属性。

```
import React, {useEffect, useState} from 'react';

interface MyProps {
    count: number;
}

const MyComponent: React.FC<MyProps> = (props: MyProps) => {

    const [odd, setOdd] = useState<boolean>(false);
    let even = false;

    useEffect(() => {
        // Render again
        props.count++;
        setOdd(props.count % 2 === 1);

        // Do not trigger a new render
        even = props.count % 2 === 0;
    }, [props.count]);

    return (
        <>
            <div>{odd} {props.count}</div>
        </>
    );
};

export default MyComponent;
```

现在，我说过我不会在本文中讨论钩子，因此让我们把它们总结为异步函数，它观察或应用一个变量的变化，在钩子专用于状态的情况下，`useState`，如果一个变化被应用到观察到的变量，触发一个新的渲染。

# 条件渲染

Angular exposes 是自己的标签，必须在模板中使用，以执行任何逻辑操作，特别是用于条件渲染的`*ngIf`。

```
<div>Hello, World!</div>

<div ***ngIf="odd">{{count}}</div>
```

JSX 的一个优点是你不用模板开发，所以你可以像写代码一样使用语句。

简而言之，一个`if`就是一个`if`😉。

关于条件渲染，需要记住的唯一重要的事情是:总是返回一些东西！这就是为什么，如果你不想渲染任何东西，我建议返回`undefined`,这样对 DOM 没有任何影响。

带模板:

```
render() {
  return <Host>
    {
      this.odd ? <div>{this.odd} {this.count}</div> : undefined
    }
  </Host>;
}
```

或者用 React:

```
return (
    <>
        {
            odd ? <div>{odd} {props.count}</div> : undefined
        }
    </>
);
```

此外，您可以像上面一样内联您的条件，或者在拆分呈现方法中明智地使用它。

如该模板示例所示:

```
render() {
  return <Host>
    {this.renderLabel()}
  </Host>;
}

private renderLabel() {
  return this.odd ? <div>{this.odd} {this.count}</div> : undefined;
}
```

或者在这个反应中:

```
return (
    <>
        {renderLabel()}
    </>
);

function renderLabel() {
    return odd ? <div>{odd} {props.count}</div> : undefined;
}
```

# 摘要

还有很多要说的和要描述的，但不幸的是，我不得不在一个有用的，特别是在这些特殊的日子里，我为一个客户开发的移动应用程序上向前迈进。

如果这道开胃菜让你渴望从一个角度更多地了解 JSX，请告诉我。我真的很乐意在几篇博文中进一步发展它。就像我说的，我还需要更多来完成我的挑战😃。

呆在家里，注意安全！

大卫