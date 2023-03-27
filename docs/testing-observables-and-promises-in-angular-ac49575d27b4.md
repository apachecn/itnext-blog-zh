# 测试角度的可观测性和承诺；

> 原文：<https://itnext.io/testing-observables-and-promises-in-angular-ac49575d27b4?source=collection_archive---------0----------------------->

## 从 angular 组件测试服务调用的快速指南。

![](img/762469821344810ea7d7231e344eb640.png)

# 我们正在测试的方法:

```
*/**
 * Sends the screenshot to the backend api and shows loading   indicators
 ** ***@param*** *$event
 */* async submitFeedback() {
  this.sendingFeedback = true;

  // Take a screenshot
  const screenshot = await this.takeScreenshot();

  this.feedbackService.sendImage(screenshot).subscribe((res) => {
    this.sendingFeedback = false;
    this.displaySuccessModal();
  }, (error => {
    this.sendingFeedback = false;
    this.displayErrorModal();
  }))
}
```

# 我们想要测试的是:

让我们在观察完成后测试我们的组件是否正确执行。

*   当服务成功响应时，它应该显示一个成功模式。
*   当服务响应一个错误时，它应该显示一个错误模式。

# 测试可观察到的成功/失败的代码示例:

```
import { ***NO_ERRORS_SCHEMA*** } from '@angular/core';
import { ComponentFixture, fakeAsync, flushMicrotasks, ***TestBed*** } from '@angular/core/testing';
import { FeedbackButtonComponent } from './feedback-button.component';
import { of, throwError } from 'rxjs';
import { HttpClientModule } from '@angular/common/http';
import { FeedbackService } from '../feedback.service';

describe('ButtonComponent', () => {
  let component: FeedbackButtonComponent;
  let fixture: ComponentFixture<FeedbackButtonComponent>;
  const feedbackServiceSpy = jasmine.createSpyObj('FeedbackService', ['']);

  beforeEach(async () => {
    await ***TestBed***.configureTestingModule({
      schemas: [***NO_ERRORS_SCHEMA***],
      declarations: [ FeedbackButtonComponent ],
      providers: [{
        provide: FeedbackService, useValue: feedbackServiceSpy
      }],
      imports: [ HttpClientModule ]
    })
    .compileComponents();
  });

  beforeEach(() => {
    fixture = ***TestBed***.createComponent(FeedbackButtonComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  describe('sendImageToServerObservable', () => {

    beforeEach(() => {
      const fakeScreenshot = { fakeScreenshot: true };
      spyOn(component, 'displaySuccessModal');
      spyOn(component, 'takeScreenshot').and.callFake(() => new ***Promise***((resolve => resolve(fakeScreenshot))));
    })

    it('should display the error message modal if an error is returned', fakeAsync(() => {
      component['feedbackService'].sendImage = () => throwError({ error: true });
      component.submitFeedback();
      flushMicrotasks();
      expect(component.displayErrorModal).toHaveBeenCalled();
    }))

    it('should display the success message modal if an error is returned', fakeAsync(() => {
      component['feedbackService'].sendImage = () => of({ success: true });
      component.submitFeedback();
      flushMicrotasks();
      expect(component.displaySuccessModal).toHaveBeenCalled();
    }))
  })
});
```

## 测试设置需要注意的重要事项:

*   在尝试测试一个方法是否被调用之前，您必须先探查该方法。例如，因为我们想测试是否已经调用了`component.takeScreenshot`,所以我们需要在调用 submitFeedback 方法之前调用它。
*   用`fakeAsync`包装 it 函数来模拟异步操作，然后在运行测试断言之前用`flushMicrotasks`跟进。
*   `fakeAsync`为您提供了一种在断言之前运行异步代码*的简单方法。它将确保没有异步工作剩余，你可以简单地调用使用你的`expect`语句。我不会试图在这里详细解释一切，但如果你想了解更多，我强烈建议你阅读这篇文章:[https://blog . nrwl . io/controlling-time-with-zone-js-and-fake async-f 0002 DFB f48 c](https://blog.nrwl.io/controlling-time-with-zone-js-and-fakeasync-f0002dfbf48c)*

## 使用 jasmine.createSpyObj 和提供程序测试设置:

如上面的代码示例所示，您将看到我如何使用`jasmine.createSpyObj`为服务创建一个 spy 来设置您的单元测试。注意，您必须将这个 spy 添加到 providers 数组中，以便您的组件知道您正在使用这个 spy，而不是实际的服务。

# 结论:

# 感谢阅读。如果你学到了什么，点击跟随按钮！