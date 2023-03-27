# 棱角分明的定制装饰

> 原文：<https://itnext.io/custom-decorators-in-angular-c54da873b3b3?source=collection_archive---------0----------------------->

![](img/d7618681dcf5f5555a2394537dbafa64.png)

释放原力！

你多久遇到一次类似的情况？

*   通过`ngOnchanges`跟踪组件属性变化。
*   调用`unsubscribe`避免内存泄漏。
*   角度区域之外的复杂计算。
*   处理和记录错误。
*   使用*存储 API。*

例行的动作，不时地重复，导致代码的重复，并随着项目的增长留下*臭味*的苦涩余味。

在这篇文章中，我将通过我们团队在 Angular 中使用 decorators 的成果为例来强调上述问题的解决方案。让我们更详细地看看它们。

# 跟踪更改

我敢肯定，在开发哑组件的过程中，当您需要对输入属性的变化做出反应时，您会反复遇到这种情况。所以这个看似普通的动作会导致过多的代码重复，同时污染和膨胀`ngOnChanges`钩子。

```
@Component({
  selector: 'app-test',
  templateUrl: './test.component.html'
})
export class TestComponent implements OnChanges {
    @Input() value1: string;
    @Input() value2: string; ngOnChanges(changes: SimpleChanges): void {
        if(changes['value1'] && changes['value1'].currentValue) {
            this.makeChangesVal1(changes['value1'].currentValue);
        }
        if(changes['value2'] && changes['value2'].currentValue) {
           this.makeChangesVal2(changes['value2'].currentValue); 
        } 
    }
}
```

为了避免这种情况，将使用我们的自定义装饰器标记`ngOnChanges` hook，并向其传递必要的参数。

```
@Component({
  selector: 'app-test',
  templateUrl: './test.component.html'
})
export class TestComponent implements OnChanges {
    @Input() value1: string;
    @Input() value2: string; @TrackChanges<string>('value1', 'makeChangesVal1')
    @TrackChanges<string>('value2', 'makeChangesVal2', ChangesStrategy.*First*)
    ngOnChanges(changes: SimpleChanges): void {}
}
```

这个想法是将条件放入装饰器中，使代码更容易阅读。同时，我们可以通过指定`strategy`参数来增加确定我们应该如何准确跟踪变更的能力。

让我们更详细地考虑一下。有以下输入参数:

*   *key: string* —跟踪属性的名称
*   *methodName: string* —将对指定属性的更改做出反应的方法的名称
*   *策略:变化策略* —确定我们是否对所有变化做出反应(可能的值为每个、第一、非第一)

```
export function TrackChanges<Type>(key: string, methodName: string, strategy: ChangesStrategy = ChangesStrategy.*Each*): Function {
  return function(targetClass, functionName: string, descriptor): Function {
    const source = descriptor.value; descriptor.value = function(changes: SimpleChanges): Function { if (changes && changes[key] && changes[key].currentValue !== undefined) {
        const isFirstChange = changes[key].firstChange; if (strategy === ChangesStrategy.*Each* ||
           (strategy === ChangesStrategy.*First* && isFirstChange) ||
           (strategy === ChangesStrategy.*NonFirst* && !isFirstChange)) {
          targetClass[methodName].call(this, changes[key].currentValue as Type);
        }
      } return source.call(this, changes);
    }; return descriptor;
  };
}
```

[*查看斯塔克布里茨*](https://stackblitz.com/edit/angular-ivy-changes-decorator?embed=1&file=src/app/app.component.ts)

# 外部计算

如你所知，有几个事件触发了变化检测机制:

*   所有浏览器事件(点击、鼠标悬停、击键等。)
*   `setTimeout()`和`setInterval()`
*   Ajax 请求

> 每次事件发生时，NgZone 都会通知 Angular，从而触发新的变化检测周期

例如，您想通过使用 *scrollTo* 滚动到指定坐标来突出显示一个块。您可以使用`runOutsideAngular`来触发角度区域之外的事件，从而优化变化检测。

```
@Component({
  selector: 'app-test',
  templateUrl: './test.component.html'
})
export class TestComponent { constructor(private ngZone: NgZone) {
        this.drawChart();
    } highlightBlock(top = 100): void {
        this.zone.runOutsideAngular(() => {
            ***window***.scrollTo({
                behavior: 'smooth',
                top
            });
        });
    } drawChart(): void {
        this.zone.runOutsideAngular(() => {
            // drawing logic
        });
    }
}
```

另一种方法是使用装饰器，使代码更干净，更具声明性。

```
@Component({
  selector: 'app-test',
  templateUrl: './test.component.html'
})
export class TestComponent { constructor(private ngZone: NgZone) {
        this.drawChart();
    } @OutsideZone
    highlightBlock(top = 100): void {
        ***window***.scrollTo({
            behavior: 'smooth',
            top
        });
    } @OutsideZone
    drawChart(): void {
        // drawing logic
    }
}
```

在我们的项目中，我们正在处理足够数量的各种图形和地图，试图将所有不需要更新 UI 的计算放在角度区域之外，从而提高性能。

```
export function OutsideZone(targetClass, functionName: string, descriptor) { const source = descriptor.value; descriptor.value = function(...data): Function { if (!this.ngZone) {
            throw new Error("Class with 'OutsideZone' decorator should have 'ngZone' class property with 'NgZone' class.");
        } return this.ngZone.runOutsideAngular(() => source.call(this,  ...data)); }; return descriptor;}
```

值得注意的是，该实现包括检查 *ngZone* 的存在。换句话说，要使用这个装饰器，需要注入`NgZone`。

> 我们不能将实例变量作为参数传递给 decorator，因为 decorator 是在声明类时调用的，而此时我们没有可以传递的实例。

[*堆栈视图*](https://stackblitz.com/edit/angular-ivy-outside-zone-decorator?embed=1&file=src/app/app.component.ts)

# 访问存储 API

有时我们必须处理 *Web 存储 API* 。这可以用不同的方式来组织:有人直接从他们需要获取值的同一个地方访问*存储 API* ，有人使用各种抽象然后调用它们，等等。无论如何，这需要太多额外的代码。让我们看看装饰方法。

```
@Component({
  selector: 'app-test',
  templateUrl: './test.component.html'
})
export class TestComponent { @Storage<number>('pagination_size', StorageType.*Local*, ***20***)
    currentItemsPerPage: number;
}
```

这是一个属性装饰器。这里我们处理的是`[defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)`方法和修改`decriptor`的能力。通过定义我们的访问器属性，我们获得了更多的灵活性和对真实属性值的控制，然后我们引入了获取/设置值的所有必要逻辑。

```
export function Storage<Type>(key: string, storageType: StorageType = StorageType.*Local*, defaultValue: Type = null): Function {
  return (target: object, propName: string) => {
    let _val: Type = target[propName]; ***Object***.defineProperty(
      target,
      propName, {
        get(): Type | unknown {
          if (!isNull(_val)) {
            return _val;
          } let item = StorageService.*getItem*(key);
          if (isNull(item)) {
            item = defaultValue;
            _val = defaultValue;
            StorageService.*setItem*(key, item, storageType === StorageType.*Local*);
          } return item;
        },
        set(item: Type): void {
          _val = item;
          StorageService.*setItem*(key, item, storageType === StorageType.*Local*);
        }
      }
    );
  };
}
```

我想特别关注一下`StorageService`。在当前的实现中，我们的装饰者完全不知道这个服务是什么，它是如何安排的，以及它在哪里写数据。它只执行纯粹的“装饰”/“装饰者”任务。这是本质，是一种抽象，可以根据你的需要随时替换。

[*堆栈视图*](https://stackblitz.com/edit/angular-storage-decorator?embed=1&file=src/app/app.component.ts)

# 错误处理

> 无法读取未定义的属性“X”

我们中有多少人发现了这一点，处理包含大量嵌套或复杂数据结构或者两者兼有的“繁重”后端 API 响应？

为了避免这种情况，通常使用 *if…else* 链或 *try…catch* 结构。当它增长时，这变成了一个逻辑和大量检查的大杂烩，方法变得臃肿，在支持和理解上变得不方便。应用装饰器:

```
@Component({
  selector: 'app-test',
  templateUrl: './test.component.html'
})
export class TestComponent { @Safe()
    getData(): object { ... } @Safe({ returnValue: [] })
    getData(): string[] { ... }
}
```

这里我们将原始方法包装在一个 *try…catch* 块中，同时允许我们添加对各种错误的跟踪和处理。首先，调用本机方法。如果出现错误，我们会按照您需要的方式进行处理，并返回默认值，该值可以作为参数传递给装饰器。

```
export function Safe<T>(params: SafeDecoratorParams<T> = {}): Function {
  return function(target: object, propertyKey: string, descriptor: TypedPropertyDescriptor<Function>): TypedPropertyDescriptor<Function> {
    const originalMethod = descriptor.value;
    const logLevel = params.logLevel || SafeDecoratorLogLevel.*Default*; descriptor.value = function SafeWrapper(): SafeDecoratorParams<T> | false {
      try {
        return originalMethod.apply(this, arguments);
      } catch (error) {
        if (logLevel === SafeDecoratorLogLevel.*Console*) { ***console***.error(error); } if (logLevel === SafeDecoratorLogLevel.*Sentry*) {
          if (!this.errorHandler) {
            throw new ***Error***(
              "Class with 'Safe' decorator and logLevel 2 should have 'errorHandler' class property with 'ErrorHandler' class."
            );
          } else {
            this.errorHandler.handleError(error);
          }
        } return params.returnValue !== undefined ? params.returnValue : false;
      }
    }; return descriptor;
  };
}
```

结果，我们得到了一个干净简单的结构，避免了许多检查和方法的增加，以及跟踪和响应各种错误的能力。

[*堆栈视图*](https://stackblitz.com/edit/angular-ivy-save-decorator?embed=1&file=src/app/app.component.ts)

# 取消订阅

几乎总有你需要退订的流。令人痛苦的熟悉情况，不是吗？

订阅一个流，一个`Subscription`被创建并继续存在，直到它*完成*或者直到我们*手动取消订阅*。但是如果有两个，三个，五个流呢…？当我们使用多重订阅时，这可能会变得相当混乱。

```
@Component({
  selector: 'app-test',
  templateUrl: './test.component.html'
})
export class TestComponent implements OnDestroy { subscription1: Subscription;
    subscription2: Subscription; constructor() {
        this.subscription1 = this.getData1().subscribe();
        this.subscription2 = this.getData2().subscribe();
    } ngOnDestroy(): void {
        this.subscription1.unsubscribe();
        this.subscription2.unsubscribe();
    } getData1(): Observable<unknown[]> {...}
    getData2(): Observable<unknown[]> {...}
}
```

让我们使用我们的装饰器来自动化这个过程:

1.  用`@TakeUntilDestroy`标记组件
2.  实施`ngOnDestroy`挂钩
3.  在我们装饰类的主体中创建`componentDestroy`属性
4.  此外，当您需要取消订阅时，可以结合使用`takeUntil`操作符和`componentDestroy`调用

```
@Component({
  selector: 'app-test',
  templateUrl: './test.component.html'
})
@TakeUntilDestroy
export class TestComponent implements OnDestroy { private componentDestroy: () => Observable<unknown>; constructor() {
        this.getData1()
            .pipe(takeUntil(this.componentDestroy()))
            .subscribe();
        this.getData2()
            .pipe(takeUntil(this.componentDestroy()))
            .subscribe();
    } ngOnDestroy(): void {} getData1(): Observable<unknown[]> {...}
    getData2(): Observable<unknown[]> {...}
}
```

就实现而言，一切看起来都很简单。装饰者创建了一个 Subject — `_takeUntilDestroy$`，它将继续管理我们的退订。下一点是覆盖`ngOnDestroy`钩子，其中我们最初将控制转移到原始方法，然后在我们的主题上调用`next()`和`complete()`。一旦这些方法奏效，`takeUntil`就会触发并取消订阅。

```
export function TakeUntilDestroy(constructor: Function): void {
  const originalDestroy = constructor.prototype.ngOnDestroy; if (typeof originalDestroy !== 'function') {
    ***console***.warn(`${constructor.name} is using @TakeUntilDestroy but does not implement OnDestroy`);
  } constructor.prototype.componentDestroy = function(): object {
    this._takeUntilDestroy$ = this._takeUntilDestroy$ || new Subject(); return this._takeUntilDestroy$.asObservable();
  }; constructor.prototype.ngOnDestroy = function(...args): void {
    if (typeof originalDestroy === 'function') {
      originalDestroy.apply(this, args);
    } if (this._takeUntilDestroy$) {
      this._takeUntilDestroy$.next();
      this._takeUntilDestroy$.complete();
    }
  };
}
```

[*查看 StackBlitz*](https://stackblitz.com/edit/angular-take-until-destroy-decorator?embed=1&file=src/app/hello.component.ts)

# 结论

装饰器是真正强大的工具，可以给你的代码带来美丽、优雅和纯洁。

我们研究了一些对我们任何人都有用的最常见和最有趣的场景。当然，你不应该仅仅局限于这些。在更多的情况下，这种力量是有用的:

*   贮藏
*   排除故障
*   扩展第三方库功能。
*   与服务器交互的服务。
*   用户设置的操作。
*   用户权限的请求和验证。
*   暂时隐藏未来的功能。

祝您在使用和编写您的 decorators 时好运！

[](https://stackblitz.com/edit/angular-custom-decorators?embed=1&file=src/app/app.component.ts&view=editor) [## 角度-定制-装饰者 StackBlitz

### 导出到 Angular CLI 的 Angular 应用程序的启动项目

stackblitz.com](https://stackblitz.com/edit/angular-custom-decorators?embed=1&file=src/app/app.component.ts&view=editor)