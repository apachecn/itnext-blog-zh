# 角度教程-使用 NgRx 创建负载指示器

> 原文：<https://itnext.io/angular-tutorial-create-loading-indicator-using-ngrx-687f8a66be0d?source=collection_archive---------0----------------------->

![](img/188f925ed64a2b8cf8fb6b7faff64913.png)

图片来源:[https://playtech . ro/2017/Cele-mai-mari-turbine-EO liene-iti-alimente aza-casa-dintr-o-singura-rotatie/](https://playtech.ro/2017/cele-mai-mari-turbine-eoliene-iti-alimenteaza-casa-dintr-o-singura-rotatie/)

如今，大多数应用程序执行异步请求来加载数据，所以就用户界面体验而言，向用户提供反馈是很重要的。
在本文中，我将向您展示一个使用 NgRx 和 Angular 的解决方案，基于分派给商店的操作来切换装载指示器的显示。

使用 redux 显示负载指示器有许多不同的方法，一种常见的方法是在 redux 上使用一个简单的布尔值，但是当应用程序增长时，就很难管理它了。我们可以利用@ngrx/effects 模块来创建一个更通用的解决方案。
那么，我们开始吧！

# 动作装饰师

在为我们的加载指示器创建动作、减少器和效果之前，让我们看一下用于 NgRx 动作的两个装饰器，它们应该显示一个加载指示器。

decorator 将向 action 类添加一个新的布尔属性，以便知道这个动作应该显示一个加载微调器。

```
**export function** *ShowLoader*() {
    **return function** (Class: ***Function***) {
        ***Object***.defineProperty(Class.**prototype**,**'showLoader'**, {
            **value**: **true** });
    }
}
```

`HideLoader` decorator 将向名为`triggerAction`的动作类添加一个新的字符串属性，该属性将用于加载指示器缩减器，以解决当两个相同的动作被触发但响应只针对最后一个动作时出现的问题。为了避免无限加载微调器，我们将使用此属性进行一些检查。我们稍后将讨论这个问题。

```
**export function** *HideLoader*(triggerAction: **string**) {
    **return function** (Class: ***Function***) {
        ***Object***.defineProperty(Class.**prototype**, **'triggerAction'**, {
            **value**: triggerAction
        });
    }
}
```

这些装饰器可以像下面的例子一样使用:

```
import { Action } from '@ngrx/store';
import { ShowLoader, HideLoader } from '../../shared/decorators';export const GET_ARTICLES = '[Article] Get Articles';
export const GET_ARTICLES_SUCCESS = '[Article] Get Articles success';
export const GET_ARTICLES_FAILURE = '[Article] Get Articles 
failure';@ShowLoader()
export class GetArticles implements Action {
  readonly type = GET_ARTICLES;
}@HideLoader(GET_ARTICLES)
export class GetArticlesSuccess implements Action {
  readonly type = GET_ARTICLES_SUCCESS;
  constructor(public payload?:any) {}
}@HideLoader(GET_ARTICLES)
export class GetArticlesFailure implements Action {
  readonly type = GET_ARTICLES_FAILURE;
  constructor(public payload?:any) {}
}export type ArticleAction = GetArticles | GetArticlesSuccess | GetArticlesFailure;
```

# 装载指示器—动作

让我们创建两个动作来显示和隐藏我们的装载指示器:

```
import { Action } from "@ngrx/store";

export const SHOW_SPINNER = '[UI] Show loading spinner';
export const HIDE_SPINNER = '[UI] Hide loading spinner';export class ShowSpinner implements Action {
  readonly type = SHOW_SPINNER; constructor(payload?: any) {}}export class HideSpinner implements Action {
  readonly type = HIDE_SPINNER; constructor(payload?: any) {}}

export type SpinnerAction = ShowSpinner | HideSpinner;
```

# **减速器**

接下来，我们将定义一个`reducer`来改变加载指示器的状态。

```
import * as loadingSpinner from '../actions/loading-spinner'; export interface State {  
  active: number;  
  actionsInProgress: any[];
}const initialState: State = {  
  active: 0,  
  actionsInProgress: []
}export function reducer(
  state = initialState, 
  action: any): State { switch(action.type) {case loadingSpinner.SHOW_SPINNER: {

        const isActionAlreadyInProgress = state.actionsInProgress
             .filter((currentAction: any) => 
                currentAction === action.payload.type)
             .length; **// If the action in already in progress and is registered
         // we don't modify the state** if(isActionAlreadyInProgress) {
          return state;
        } **// Adding the action type in our actionsInProgress array** const newActionsInProgress = [
              ...state.actionsInProgress, 
              action.payload.type
        ]; return Object.assign(state, {     
            active: newActionsInProgress.length, 
            actionsInProgress: newActionsInProgress                 
        });
     }case loadingSpinner.HIDE_SPINNER: { **// We remove trigger action from *actionsInProgress* array** const *newActionsInProgress* = action.payload.triggerAction  ?
              *state*.*actionsInProgress* .filter((currentAction: any) => 
                   currentAction !== action.payload.triggerAction) :
              *state*.*actionsInProgress*;

        return Object.assign(state, {
           actionsInProgress: newActionsInProgress,
           active: state.active > 0 ? 
                   newActionsInProgress.length : 0
         });
     } default:      
       return state;
   }
}export const isLoadingSpinnerActive = 
                (state: State) => state.active;
```

让我们回顾一下我们的减速器，以了解这里发生了什么:

*   我们的状态包含两个属性:
    `active` —这是正在进行的动作的数量。`active > 0`表示我们可以显示我们的装载指示器。
    `actionsInProgress` —这是保存所有应该显示加载指示器的动作的数组。

**ShowSpinner** 和 **HideSpinner** 动作的有效负载将是另一个类似`ShowArticles, ShowArticlesSuccess, ShowArticlesFailure.` 的动作，因此，我们可以访问由我们的 decorator(`triggerAction`)添加的属性来检查我们是否应该改变状态。

# 选择器

要从缩减器中访问`active`值，我们需要为此创建一个选择器。

```
import { createSelector, createFeatureSelector } from "@ngrx/store";import * as fromSpinner from './reducers/loading-spinner';export interface State {
  loading: fromSpinner.State
}export const reducers = {
  loading: fromSpinner.reducer
}export const getLoadingState = (state: State) => state.loading;export const isLoadingSpinnerActive = createSelector(
getLoadingState,
fromSpinner.isLoadingSpinnerActive
);
```

# 效果

一旦我们创建了我们的动作和 reducer，我们就可以实现触发`ShowSpinner`和`HideSpinner`动作的效果服务了。

```
import {Injectable} from "@angular/core";
import {Actions, Effect} from "@ngrx/effects";
import {filter, map} from "rxjs/operators";@Injectable()
export class LoadingIndicatorEffects {
    constructor(private actions$: Actions) {}

  @Effect()
  showLoader$ = this.actions$.pipe(
    *filter*((action: any) => action && action.displayLoader ? 
          action : null),
    *map*((action: any) => new ShowSpinner(action))
  );

  @Effect()
  hideLoader$ = this.actions$.pipe(
    *filter*((action: any) => action && action.triggerAction ? 
          action : null),
    *map*((action: any) => new HideSpinner(action))
  );
}
```

`showLoader$`将检查所有动作，如果动作包含`displayLoader`属性，将触发 **ShowSpinner** 。

如果动作包含`triggerAction`属性，`hideLoader$`将触发 **HideSpinner** 。

# 模板

现在，您可以创建一个订阅`isLoadingSpinnerActive`选择器的加载组件，并在用户界面中以可视动画的形式显示它。

```
export class LoadingComponent implements OnInit {
  isLoading: Observable<boolean>;

  constructor(private store: Store<AppState>) {}

  ngOnInit() {
    this.isLoading = this.store.pipe(
        select(isLoadingSpinnerActive)
    );
  }
}
```

下面是加载组件模板的片段:

```
<ng-content *ngIf="!(isLoading | async); else spinner"></ng-content>
<ng-template #spinner>
  <!-- YOUR ANIMATION / SPINNER HERE -->
</ng-template> 
```

# 结论

在本文中，我利用@ngrx/effects 解决了全局显示加载指示器的任务，它对我有效。但这不是唯一的解决方案，[这里](http://gavinschulz.com/posts/2017-03-22-4-techniques-for-loading-states-in-redux.html)描述了另一种可以考虑的技术。

# 演示