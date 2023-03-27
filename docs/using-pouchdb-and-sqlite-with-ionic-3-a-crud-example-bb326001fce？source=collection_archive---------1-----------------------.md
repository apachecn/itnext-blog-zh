# 在 Ionic 3 中使用 PouchDB 和 SQLite:一个 CRUD 示例

> 原文：<https://itnext.io/using-pouchdb-and-sqlite-with-ionic-3-a-crud-example-bb326001fce?source=collection_archive---------1----------------------->

在本教程中，我们将学习如何使用 PouchDB 和 SQLite 创建一个 CRUD(创建、读取、更新和删除的缩写)移动应用程序。

我们将讨论以下几点:

*   PouchDB 是什么？
*   如何使用 Ionic CLI 创建新的 Ionic 3 项目？
*   如何安装 SQLite 2 Cordova 插件？
*   如何添加 PouchDB 库？
*   如何添加 PouchDB 的 SQLite 适配器？
*   如何创建使用 PouchDB 的数据库服务？
*   如何创建不同的页面来显示、创建、编辑和删除数据库记录？
*   最后，如何使用真实的设备在本地提供应用程序？

本教程假设您已经准备好一台开发机器，并安装了:

*   安装了 NodeJS 平台和 NPM。
*   安装了 Ionic CLI。
*   安装 Java 和 Android SDK 工具，以防您想为 Android 开发应用程序。
*   用于构建 iOS 应用程序的 MAC 和 Xcode。

所以如果你准备好了！让我们开始吧。

# PouchDB 是什么？

PouchDB 在一个基于 CouchDB 的开源 NoSQL(不仅仅是 SQL)浏览器数据库中。它的创建是为了使开发人员能够构建离线的第一网络应用程序，即在没有网络连接时能够离线工作的应用程序，通过将数据存储在浏览器的本地数据库中，如本地存储或 IndexedDB，以及移动应用程序的 SQLite。以及当用户重新在线时与 CouchDB 服务器同步数据。

对于 Ionic 3，您可以使用本地存储、IndexedDB 或 SQLite。前两个选项有存储限制，但速度更快。对于移动设备上的无限存储，无论是 Android 还是 iOS，您都可以使用 SQLite。

PouchDB 有很多特性。让我们简单地看看它们:

*   它是跨浏览器的:它可以在所有主要浏览器中工作，如 Mozilla Firefox、谷歌 Chrome、Opera、苹果 Safari、微软 IE(也许还有 Edge？)和 Node.js 平台。
*   它是轻量级的:当 gzip 压缩时，PouchDB 的大小只有 46KB。您可以在浏览器中用一个简单的`<script>`标签包含它，或者通过 npm 安装在 NodeJS 中。
*   它的学习曲线很短:PouchDB API 易于学习和使用。
*   它具有开箱即用的 CouchDB 服务器同步功能。

# 创建新的 Ionic 3 项目

让我们从使用 Ionic CLI 创建一个新的 Ionic 3 项目开始。因此，继续操作，在 Mac/Linux 上打开终端，或者在 Windows 上打开命令提示符，运行下面的命令来生成一个新项目。

```
$ ionic start ionic3-pouchdb-sqlite blank
```

等待项目设置依赖项，然后在根文件夹中导航:

接下来，您需要添加一些依赖项，以使 PouchDB 和 SQLite 能够工作。

# 正在安装 SQLite Cordova 插件

在 Ionic 2+上，本地 SQLite 数据库是在本地存储数据时最合适的选择，因为它允许无限的存储，不幸的是，这不是本地存储或 IndexedDB 的情况。此外，SQLite 是一个基于文件的数据库，这使得非常便携。

您可以使用许多 Cordova 插件将 SQLite 支持添加到 Ionic 应用程序中，例如:

*   存储:sqlite 的原始 Cordova 插件。
*   cordova-plugin-sqlite-2:前一个插件的分支，具有额外的功能。

让我们安装叉子:

```
$ ionic plugin add cordova-plugin-sqlite-2
```

# 添加 PouchDB

为了能够使用 PouchDB，您需要安装 npm 提供的第三方库:

```
$ npm install pouchdb --save
```

# 为 SQLite 添加 PouchDB Cordova 适配器

接下来，您需要安装另一个第三方适配器来使用 SQLite 和 PouchDB

```
$ npm install pouchdb-adapter-cordova-sqlite --save
```

这就是在 Ionic 应用程序中开始使用 PouchDB 和 SQLite 所需安装的全部内容。现在我们开始编码吧！

我们将建立一个小的应用程序，允许你做 CRUD 操作，即从 PouchDB + SQLIte 数据库创建，读取，更新和删除雇员数据。

该应用程序有许多屏幕:*雇员屏幕:用删除按钮列出雇员*创建雇员屏幕*更新雇员屏幕

# 数据库服务

我们需要创建的第一件事是数据库服务，它连接到 PouchDB 并提供不同的方法来处理 employees 数据库。

因此，继续使用`ionic g provider <name>`创建一个服务提供商

确保您在项目的根文件夹中，然后运行以下命令:

```
ionic g provider employee
```

您应该在`src/providers/employee`文件夹中生成一个`EmployeeProvider`提供程序

然后添加以下代码

```
import { Injectable } from '@angular/core'; import { Http } from '@angular/http'; import 'rxjs/add/operator/map'; import * as PouchDB from 'pouchdb'; import cordovaSqlitePlugin from 'pouchdb-adapter-cordova-sqlite'; @Injectable() export class EmployeeProvider { public pdb; public employees; createPouchDB() { PouchDB.plugin(cordovaSqlitePlugin); this.pdb = new PouchDB('employees.db', { adapter: 'cordova-sqlite' }); } }
```

这段代码将创建一个名为`employees.db`的 SQLite 数据库文件，并通过将适配器设置为`cordova-sqlite`来初始化 PouchDB 数据库，这将指示 PouchDB 使用 SQLite 进行存储，而不是浏览器的存储。

确保将提供者导入到`src/app/app.module.ts`中，如果没有自动添加，则将其添加到提供者数组中。

还要确保导入服务提供者，并在使用它之前将其注入到任何组件的构造函数中。

现在让我们添加 CRUD 方法:

```
create(employee) { return this.pdb.post(employee); }
```

该方法在数据库中创建一个新雇员。

`post()`是一个 PouchDB API，允许您在 PouchDB 数据库中创建新对象。

```
update(employee) { return this.pdb.put(employee); }
```

此方法更新数据库中的现有雇员。请注意，您需要传递一个带有雇员的`id`的雇员对象来进行更新。

```
delete(employee) { return this.pdb.delete(employee); }
```

该方法从数据库中删除一个雇员。

```
read() { function allDocs(){ this.pdb.allDocs({ include_docs: true}) .then(docs => { this.employees = docs.rows.map(row => { row.doc.Date = new Date(row.doc.Date); return row.doc; }); return this.employees; }); } this.pdb.changes({ live: true, since: 'now', include_docs: true}) .on('change', ()=>{ allDocs().then((emps)=>{ this.employees = emps; }); }); return allDocs(); }
```

read 方法只是通过调用`.allDocs()`方法从数据库中获取所有雇员，该方法返回一个承诺，该承诺解析为数据库中所有雇员的数组。`map()`将`docs`数组映射到`docs.rows`数组，后者只包含数据，没有 PouchDB 特定的信息，显然，我们不需要这些信息！

代码还将`row.doc.Date`(存储为 JSON)转换为 JavaScript Date()。

我们还监听 PouchDB 数据库的变化，因此每当有变化时(在开始检索所有雇员后创建、删除或更新),函数`allDocs()`被再次调用，然后被解析以更新`employees`数组。

请注意，当发生任何变化时，这不是更新 employees 数组的最有效方法。您可以更改此代码，只更新或删除受影响的项目，而不是整个数组。

# 构建应用程序 UI 屏幕

现在我们已经创建了服务提供者，它负责连接 PouchDB 和 SQLite，并提供所有 CRUD 方法与 PouchDB 数据库接口。让我们创建不同的页面，允许我们在雇员数据库上列出和执行操作。

我们已经有一个生活在`src/app/pages/home`为我们生成的主页。打开`home.html`，然后使用`<ion-list>`更新代码以显示员工列表。

```
<ion-header> <ion-navbar> <ion-title>The Employees Database</ion-title> <ion-buttons end> <button ion-button (click)="addEmployee()"> <ion-icon name="add"></ion-icon> </button> </ion-buttons> </ion-navbar> </ion-header> <ion-content padding> <ion-list> <ion-item *ngFor="let emp of employees" (click)="showDetails(employee)"> <div> </div> </ion-item> </ion-list> </ion-content>
```

接下来，我们需要添加代码以获取`src/pages/home/home.ts`中的雇员，因此打开文件，然后更新它以匹配以下代码:

```
import { Component } from '@angular/core'; import { NavController , ModalController } from 'ionic-angular'; import { EmployeePage } from './../employee/employee.ts'; @Component({ selector: 'page-home', templateUrl: 'home.html' }) export class HomePage { public employees : [] = []; constructor(public navCtrl: NavController,public modalCtrl: ModalController, public employeeProvider : EmployeeProvider) { } ionViewDidEnter() { this.employeeProvider.createPouchDB(); this.employeeProvider.read() .then(employees => { this.employees = employees; }) .catch((err)=>{}); } showDetails(employee) { let modal = this.modalCtrl.create(EmployeePage, { employee: employee }); modal.present(); } }
```

接下来，我们需要为员工的详细信息生成一个新页面，因此继续在您的终端中运行以下命令:

这将在`app/pages/employee`中生成一个包含三个文件的`employee`文件夹。打开`employee.html`，然后更新它以匹配以下代码:

```
<ion-header> <ion-navbar> <ion-title> Employee Details</ion-title> <ion-buttons end *ngIf="canDelete"> <button ion-button (click)="delete()"> <ion-icon name="trash"></ion-icon> </button> </ion-buttons> </ion-navbar> </ion-header> <ion-content> <ion-list> <ion-item> <ion-label>First Name</ion-label> <ion-input text-right type="text" [(ngModel)]="employee.firstName"></ion-input> </ion-item> <ion-item> <ion-label>Last Name</ion-label> <ion-input text-right type="text" [(ngModel)]="employee.lastName"></ion-input> </ion-item> </ion-list> <button ion-button block (click)="addOrUpdate()">Add/Update Employee</button> </ion-content>
```

下一次更新`src/pages/employee/employee.ts`以添加所需的逻辑代码

```
import { Component } from '@angular/core'; import { NavController, NavParams ,ViewController } from 'ionic-angular'; @Component({ selector: 'employee', templateUrl: 'employee.html' }) export class EmployeePage { employee: any = {}; canDelete : false; canUpdate : false; constructor(public navCtrl: NavController, navParams: NavParams, private employeeProvider: EmployeeProvider) { } ionViewDidEnter(){ var employee = this.navParams.get('employee'); if(employee){ this.employee = employee; this.canDelete = true; this.canUpdate = true; } } addOrUpdate() { if (this.canUpdate) { this.employeeProvider.update(this.employee) .catch(()=>{}); } else { this.employeeProvider.create(this.employee) .catch(()=>{}); } this.viewCtrl.dismiss(this.employee); } delete() { this.employeeProvider.delete(this.employee) .catch(()=>{}); this.viewCtrl.dismiss(this.employee); } }
```

# 为应用服务

我们已经用 PouchDB 和 SQLite 完成了简单的 CRUD 示例。现在你可以在本地测试你的应用了。

因此，继续运行以下命令:

你也可以插入你的 Android 或 iOS 设备，在真实的设备上运行应用程序，而不是在浏览器上。

```
$ ionic run ios $ ionic run android
```

# 结论

在本教程中，我们一步一步地看到了如何使用 Ionic CLI 从头开始创建 Ionic 3 移动应用程序，然后添加了基本的 CRUD 方法，用于使用 PouchDB 创建、读取、更新和删除 SQLIte 数据库中的项目。

*原载于*[*www.techiediaries.com*](https://www.techiediaries.com/ionic-sqlite-pouchdb/)*。*