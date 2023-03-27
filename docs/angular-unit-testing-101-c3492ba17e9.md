# 角度单位测试 101

> 原文：<https://itnext.io/angular-unit-testing-101-c3492ba17e9?source=collection_archive---------5----------------------->

![](img/e206e35525411bb805837bda70238e51.png)

是关于角 2+单元测试的。

```
Angular: v7.2.3
Karma: v4.0.1
Jasmine-core: v3.1.0
```

**添加组件**

```
beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [
        AppComponent
      ],
    }).compileComponents();
  }));
// compileComponents may not needed if using WebPack
```

**创建一个组件**

```
const fixture = TestBed.createComponent(AppComponent);
const app = fixture.debugElement.componentInstance;
```

**覆盖** **入口组件**

```
TestBed.overrideModule(BrowserDynamicTestingModule, {
      set: {
           entryComponents: [SomeComponent]
      }
});
```

**创建编译后的 HTML 组件**

```
const fixture = TestBed.createComponent(AppComponent);
fixture.detectChanges();
const compiled = fixture.debugElement.nativeElement;
compiled.querySelector('h1').textContent
```

**向组件注入服务**

```
const quoteService = fixture.debugElement.injector.get(QuoteService);fixture.detectChanges();
```

**模拟服务**

```
const data: Sample = new Sample('test');
class MockSampleService {
   // Override test function from SampleService
   test(): Observable<Sample>{
      return of(data);
   }
}providers: [{provide: SampleService, useValue: {}]TestBed.overrideProvider(SampleService, {useValue: new MockSampleService()});
```

模仿 HttpClient 服务的示例

我通常使用这个假的 HttpClient 调用来避免“**无法从假的异步测试**中生成 xhr”的错误。

```
const httpClientSpy: jasmine.SpyObj<HttpClient> = jasmine.createSpyObj('httpClient', ['get']);httpClientSpy.get.and.returnValue(of('your-data'}));...providers: [{provide: HttpClient, useValue: {}]TestBed.overrideProvider(HttpClient, { useValue: httpClientSpy });
```

**窥探服务功能**

```
spyOn(SomeService.prototype, 'someFunc').and.returnValue('test');
// commands to be run
expect(SomeService.prototype.someFunc).toHaveBeenCalled()
```

刺探函数时不必添加 **returnValue** 函数。

**模拟组件**

```
[@Component](http://twitter.com/Component)({
  template: `<app-user [user]="user"></app-user>`
})
class MockUser {
  user = {
    name: 'foobar'
  };
}
```

**模拟异步(fakeAsync)**

如果我有一个来自 constructor 或 ngOnInit 的异步调用，我将使用 fakeAsync 来等待所有调用完成。

```
beforeEach(fakeAsync(() => {
  fixture = TestBed.createComponent(AppComponent);
  component = fixture.debugElement.componentInstance; component.ngOnInit();
  tick();
  ...
}));
```

**故障排除**

*   **错误:1 个定时器仍在队列中。**

如果您正在使用 ***fakeAsync*** ，请添加 ***tick()*** 功能，并尝试增加 tick 功能中的定时器，以测试其是否工作(如 tick(15000) ~ 15s)

*   **类型错误:无法读取空值**的属性“getComponentFromError”

请在每个功能前添加这些**中的重置测试环境**

```
TestBed.resetTestEnvironment();
TestBed.initTestEnvironment(BrowserDynamicTestingModule, platformBrowserDynamicTesting());
```

*   如果要运行单个测试，应该导入区域文件。然后在你运行时删除它们来运行每一个测试用例，否则，你会得到一些错误。

```
import 'zone.js/dist/zone';
import 'zone.js/dist/long-stack-trace-zone';
import 'zone.js/dist/proxy';
import 'zone.js/dist/sync-test';
import 'zone.js/dist/jasmine-patch';
import 'zone.js/dist/async-test';
import 'zone.js/dist/fake-async-test';
```

这是仅用于运行一个测试的命令。

```
ng test --main path-to-spec.file.js
```

*   添加 ***浏览器:['ChromeHeadless']*** 到***karma . conf . js***以便在无头模式下运行单元测试