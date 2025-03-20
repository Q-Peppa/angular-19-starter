## Angular 服务：构建可重用的代码模块

Angular 服务是构建可重用和模块化代码的关键组成部分。 它们允许你将应用程序的特定逻辑（如数据获取、日志记录或状态管理）封装在可注入的类中，从而提高代码的可维护性、可测试性和可重用性。 本文将深入探讨 Angular 服务的概念、优势以及如何创建和使用它们。

### 什么是 Angular 服务？

简单来说，Angular 服务就是一个普通的 TypeScript 类，它负责处理应用程序的特定任务。 它的特殊之处在于：

*   **可注入：** 可以通过依赖注入（Dependency Injection, DI）注入到组件、指令、其他服务等。
*   **单例模式：**  通常情况下，服务在整个应用程序中只会被实例化一次 (singleton)，确保了数据的一致性和共享。
*   **解耦：**  将业务逻辑从组件中分离出来，使组件更加专注于视图呈现和用户交互。

### 为什么使用 Angular 服务？

使用 Angular 服务带来诸多好处：

*   **代码重用：**  在多个组件中共享相同的逻辑，避免代码冗余。
*   **模块化：**  将应用程序分解为更小、更易于管理的模块。
*   **可测试性：**  更容易对服务进行单元测试，因为它们不依赖于特定的组件或视图。
*   **可维护性：**  修改服务中的逻辑不会影响到使用该服务的组件。
*   **依赖注入：**  利用 DI 容器管理服务的生命周期和依赖关系。

### 创建 Angular 服务

可以使用 Angular CLI 轻松创建一个服务：

```bash
ng generate service my-service
```

这会在 `src/app` 目录下生成 `my-service.service.ts` 文件和一个对应的测试文件 `my-service.service.spec.ts`.

`my-service.service.ts` 文件内容如下：

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root' //  `root` 级别注册，使其成为单例并可在整个应用中使用
})
export class MyServiceService {

  constructor() { }

  getData(): string {
    return "Hello from MyService!";
  }
}
```

**代码解释：**

*   `@Injectable()` 装饰器告诉 Angular 这个类可以被注入到其他类中。
*   `providedIn: 'root'`  指定服务在根模块中注册，使其成为单例，可以在整个应用程序中使用。  也可以将 `providedIn` 设置为特定的模块，使其只在该模块及其子组件中可见。
*   `getData()` 方法是一个简单的示例方法，用于返回数据。

### 使用 Angular 服务

要使用服务，需要将它注入到组件或其他服务中。  在构造函数中使用依赖注入：

**1. 注入服务 (my-component.component.ts):**

```typescript
import { Component, OnInit } from '@angular/core';
import { MyServiceService } from '../my-service.service';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.component.html',
  styleUrls: ['./my-component.component.css']
})
export class MyComponentComponent implements OnInit {

  message: string;

  constructor(private myService: MyServiceService) { } // 注入 MyServiceService

  ngOnInit() {
    this.message = this.myService.getData();
  }

}
```

**代码解释：**

*   在构造函数中，通过类型注解 `private myService: MyServiceService` 将 `MyServiceService` 注入到组件中。 `private` 关键字不仅声明了依赖，还自动创建了一个组件的私有属性来存储该依赖的实例。
*   在 `ngOnInit` 生命周期钩子中，调用 `myService.getData()` 方法获取数据，并将数据赋值给 `message` 属性。

**2. 在模板中使用数据 (my-component.component.html):**

```angular17html
<p>{{ message }}</p>
```

### 不同 `providedIn` 选项的影响

`providedIn` 选项决定了服务的可见性和生命周期。  以下是不同的选项及其影响：

*   **`providedIn: 'root'`:**  服务在根模块中注册，成为应用级的单例。  这意味着整个应用程序只有一个 `MyServiceService` 实例。 建议用于需要在应用程序的任何地方共享的服务。
*   **`providedIn: MyModule`:** 服务在 `MyModule` 模块中注册。  只有 `MyModule` 及其子组件才能访问该服务。  `MyModule` 的每个实例都会创建一个新的 `MyServiceService` 实例。
*   **不使用 `@Injectable()` 或没有 `providedIn`:** 需要手动将服务添加到模块的 `providers` 数组中。 这种方式相对繁琐，推荐使用 `providedIn` 选项。

**示例 (app.module.ts):**

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { MyServiceService } from './my-service.service'; // 导入 MyService

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [
    // MyServiceService //  如果 MyServiceService 没有 @Injectable 装饰器或没有 providedIn 设置，则需要在此处手动注册
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### 高级服务应用

*   **数据获取服务:**  可以使用 Angular 的 `HttpClient` 模块创建一个服务，用于从 API 获取数据。
*   **状态管理服务:**  可以使用服务来管理应用程序的状态，例如使用 RxJS 的 `BehaviorSubject` 或 NgRx 进行状态管理。
*   **认证服务:**  创建一个服务来处理用户认证和授权。

### 示例：数据获取服务

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class DataService {

  private apiUrl = 'https://jsonplaceholder.typicode.com/todos'; // 示例 API

  constructor(private http: HttpClient) { }

  getTodos(): Observable<any[]> {
    return this.http.get<any[]>(this.apiUrl);
  }
}
```

**使用数据获取服务 (my-component.component.ts):**

```typescript
import { Component, OnInit } from '@angular/core';
import { DataService } from '../data.service';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.component.html',
  styleUrls: ['./my-component.component.css']
})
export class MyComponentComponent implements OnInit {

  todos: any[];

  constructor(private dataService: DataService) { }

  ngOnInit() {
    this.dataService.getTodos().subscribe(data => {
      this.todos = data;
    });
  }
}
```

**在模板中显示数据 (my-component.component.html):**

```angular17html
<ul>
  <li *ngFor="let todo of todos">{{ todo.title }}</li>
</ul>
```

### 总结

Angular 服务是构建可重用、模块化和可测试的 Angular 应用的关键。  通过将业务逻辑封装在服务中，你可以提高代码的可维护性和可重用性。  熟练掌握 Angular 服务是成为一名优秀的 Angular 开发者的必备技能。  希望这篇文章能帮助你更好地理解和使用 Angular 服务。  多实践，多尝试，你一定能掌握它!
