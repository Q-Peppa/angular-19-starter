## Angular 组件生命周期详解

Angular 组件的生命周期是指组件从创建到销毁的整个过程。 在这个过程中，Angular 会调用一系列的钩子函数（Lifecycle Hooks），允许开发者在组件的不同阶段执行自定义的操作。 理解 Angular 组件的生命周期对于编写高效、可维护的 Angular 应用至关重要。

## 1. 生命周期钩子函数概览

Angular 提供了以下生命周期钩子函数：

*   **`ngOnChanges`**: 当 Angular（重新）设置数据绑定的输入属性时响应。 该方法接受当前和之前的属性值的 `SimpleChanges` 对象。 在 `ngOnInit` 之前以及每当一个或多个输入属性的值发生变化时都会调用。

*   **`ngOnInit`**: 在 Angular 初始化数据绑定 *之后*，初始化组件/指令。 只在指令/组件实例化后第一次调用。 用于执行组件的初始化逻辑。

*   **`ngDoCheck`**: 检测 Angular 无法或不愿意自己检测的变化。 如果组件使用了 `OnPush` 变更检测策略，或者你需要更细粒度的变更检测，可以使用 `ngDoCheck`。

*   **`ngAfterContentInit`**: 在 Angular 将内容投影进组件 *之后* 响应。 仅当 Angular 做了内容投影之后调用一次。

*   **`ngAfterContentChecked`**: 在 Angular 检查完被投影到组件/指令中的内容 *之后* 响应。 在每次 Angular 做完内容投影的变更检测之后调用。

*   **`ngAfterViewInit`**: 在 Angular 初始化组件的视图 *之后*（及其子视图）。 仅当这个组件的视图及其子视图完全初始化之后调用一次。

*   **`ngAfterViewChecked`**: 在 Angular 检查完组件的视图 *之后*（及其子视图）。 在每次 Angular 做完组件视图及其子视图的变更检测之后调用。

*   **`ngOnDestroy`**: 在 Angular 销毁组件/指令 *之前* 做清理工作。 这是释放组件使用的资源、取消订阅 Observables、解除事件监听器等的好地方，以防止内存泄漏。

## 2. 生命周期钩子函数详解

### 2.1 `ngOnChanges`

*   **作用：** 当组件的输入属性（使用 `@Input` 装饰器声明的属性）的值发生变化时，Angular 会调用 `ngOnChanges` 钩子函数。
*   **参数：** 接收一个 `SimpleChanges` 对象，该对象包含了当前属性值和先前属性值的映射。
*   **使用场景：**
    *   对输入属性的值进行校验。
    *   根据输入属性的值执行一些计算或初始化操作。
    *   监听输入属性的变化，并更新组件的状态。

    ```typescript
    import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';

    @Component({
      selector: 'app-my-component',
      template: `<p>Value: {{ value }}</p>`
    })
    export class MyComponent implements OnChanges {
      @Input() value: number;

      ngOnChanges(changes: SimpleChanges) {
        if (changes['value']) {
          const currentValue = changes['value'].currentValue;
          const previousValue = changes['value'].previousValue;
          console.log(`Value changed from ${previousValue} to ${currentValue}`);

          if (currentValue < 0) {
            console.error('Value cannot be negative');
            // 可以采取一些措施，例如重置 value 的值
          }
        }
      }
    }
    ```

### 2.2 `ngOnInit`

*   **作用：** 在 Angular 初始化数据绑定 *之后*，初始化组件/指令。 只在指令/组件实例化后第一次调用。
*   **使用场景：**
    *   执行组件的初始化逻辑，例如获取数据、设置初始状态等。
    *   组件的数据绑定已经完成，可以安全地访问输入属性的值。
    *   通常用于订阅 Observables，但务必在 `ngOnDestroy` 中取消订阅。

    ```typescript
    import { Component, OnInit, Input } from '@angular/core';
    import { MyService } from './my.service';
    import { Subscription } from 'rxjs';

    @Component({
      selector: 'app-my-component',
      template: `<p>Data: {{ data }}</p>`
    })
    export class MyComponent implements OnInit {
      @Input() id: number;
      data: any;
      private subscription: Subscription;

      constructor(private myService: MyService) { }

      ngOnInit() {
        this.subscription = this.myService.getData(this.id).subscribe(data => {
          this.data = data;
        });
      }
    }
    ```

### 2.3 `ngDoCheck`

*   **作用：** 用于自定义变更检测逻辑。 Angular 会在每次变更检测周期中调用 `ngDoCheck` 钩子函数。
*   **使用场景：**
    *   当需要检测 Angular 无法或不愿意自己检测的变化时，例如检测对象或数组内部的深层变化。
    *   当组件使用了 `OnPush` 变更检测策略，并且需要更精细的变更检测控制时。
*   **注意事项：**
    *   `ngDoCheck` 会在每次变更检测周期中调用，因此应该避免在其中执行昂贵的操作，以防止性能问题。
    *   可以使用 `KeyValueDiffers` 和 `IterableDiffers` 来帮助检测对象和数组的变化。

    ```typescript
    import { Component, DoCheck, KeyValueDiffer, KeyValueDiffers } from '@angular/core';

    @Component({
      selector: 'app-my-component',
      template: `
        <p>Name: {{ person.name }}</p>
        <p>Age: {{ person.age }}</p>
      `
    })
    export class MyComponent implements DoCheck {
      person = { name: 'John', age: 30 };
      differ: KeyValueDiffer<string, any>;

      constructor(private differs: KeyValueDiffers) {
        this.differ = this.differs.find(this.person).create();
      }

      ngDoCheck() {
        const changes = this.differ.diff(this.person);
        if (changes) {
          changes.forEachChangedItem(item => {
            console.log('changed', item);
          });
          changes.forEachAddedItem(item => console.log('added', item));
          changes.forEachRemovedItem(item => console.log('removed', item));
        }
      }
    }
    ```

### 2.4 `ngAfterContentInit`

*   **作用：** 在 Angular 将内容投影进组件 *之后* 响应。 只在指令/组件实例化之后调用一次。
*   **使用场景：**
    *   当需要在组件的内容投影完成后执行一些操作时。
    *   可以访问被投影的内容，并对其进行操作。

    ```typescript
    import { Component, AfterContentInit, ContentChildren, QueryList } from '@angular/core';
    import { MyDirective } from './my.directive';

    @Component({
      selector: 'app-my-component',
      template: `
        <ng-content></ng-content>
      `
    })
    export class MyComponent implements AfterContentInit {
      @ContentChildren(MyDirective) myDirectives: QueryList<MyDirective>;

      ngAfterContentInit() {
        console.log('Content children:', this.myDirectives);
        // 可以对 myDirectives 进行操作
      }
    }
    ```

### 2.5 `ngAfterContentChecked`

*   **作用：** 在 Angular 检查完被投影到组件/指令中的内容 *之后* 响应。 在每次 Angular 做完内容投影的变更检测之后调用。
*   **使用场景：**
    *   当需要在每次内容投影的变更检测完成后执行一些操作时。
    *   通常与 `ngAfterContentInit` 配合使用。

    ```typescript
    import { Component, AfterContentChecked, ContentChildren, QueryList } from '@angular/core';
    import { MyDirective } from './my.directive';

    @Component({
      selector: 'app-my-component',
      template: `
        <ng-content></ng-content>
      `
    })
    export class MyComponent implements AfterContentChecked {
      @ContentChildren(MyDirective) myDirectives: QueryList<MyDirective>;

      ngAfterContentChecked() {
        console.log('Content checked:', this.myDirectives);
        // 可以对 myDirectives 进行操作
      }
    }
    ```

### 2.6 `ngAfterViewInit`

*   **作用：** 在 Angular 初始化组件的视图 *之后*（及其子视图）响应。 只在指令/组件实例化之后调用一次。
*   **使用场景：**
    *   当需要在组件的视图及其子视图完全初始化完成后执行一些操作时。
    *   可以访问组件的 DOM 元素和子组件。
    *   通常用于与第三方库集成，或者执行一些需要在 DOM 准备好之后才能进行的操作。

    ```typescript
    import { Component, AfterViewInit, ViewChild, ElementRef } from '@angular/core';

    @Component({
      selector: 'app-my-component',
      template: `<div #myDiv>Hello</div>`
    })
    export class MyComponent implements AfterViewInit {
      @ViewChild('myDiv') myDiv: ElementRef;

      ngAfterViewInit() {
        console.log('My div:', this.myDiv.nativeElement);
        // 可以对 myDiv.nativeElement 进行操作
      }
    }
    ```

### 2.7 `ngAfterViewChecked`

*   **作用：** 在 Angular 检查完组件的视图 *之后*（及其子视图）响应。 在每次 Angular 做完组件视图及其子视图的变更检测之后调用。
*   **使用场景：**
    *   当需要在每次视图的变更检测完成后执行一些操作时。
    *   通常与 `ngAfterViewInit` 配合使用。
    *   避免在 `ngAfterViewChecked` 中修改组件的状态，这可能会导致无限循环。

    ```typescript
    import { Component, AfterViewChecked, ViewChild, ElementRef } from '@angular/core';

    @Component({
      selector: 'app-my-component',
      template: `<div #myDiv>Hello</div>`
    })
    export class MyComponent implements AfterViewChecked {
      @ViewChild('myDiv') myDiv: ElementRef;

      ngAfterViewChecked() {
        console.log('View checked:', this.myDiv.nativeElement);
        // 可以在这里进行一些只读的操作
      }
    }
    ```

### 2.8 `ngOnDestroy`

*   **作用：** 在 Angular 销毁组件/指令 *之前* 做清理工作。
*   **使用场景：**
    *   释放组件使用的资源，例如取消订阅 Observables、解除事件监听器等。
    *   防止内存泄漏。
    *   清理临时数据。

    ```typescript
    import { Component, OnInit, OnDestroy, Input } from '@angular/core';
    import { MyService } from './my.service';
    import { Subscription } from 'rxjs';

    @Component({
      selector: 'app-my-component',
      template: `<p>Data: {{ data }}</p>`
    })
    export class MyComponent implements OnInit, OnDestroy {
      @Input() id: number;
      data: any;
      private subscription: Subscription;

      constructor(private myService: MyService) { }

      ngOnInit() {
        this.subscription = this.myService.getData(this.id).subscribe(data => {
          this.data = data;
        });
      }

      ngOnDestroy() {
        this.subscription.unsubscribe(); // 取消订阅，防止内存泄漏
      }
    }
    ```

## 3. 生命周期钩子函数的调用顺序

Angular 组件生命周期钩子函数的调用顺序如下：

1.  `ngOnChanges` (仅当输入属性发生变化时)
2.  `ngOnInit`
3.  `ngDoCheck`
4.  `ngAfterContentInit`
5.  `ngAfterContentChecked`
6.  `ngAfterViewInit`
7.  `ngAfterViewChecked`
8.  `ngOnDestroy`

## Angular 组件生命周期钩子比较

以下表格比较了 Angular 组件的生命周期钩子，包括它们的触发时机、作用以及注意事项：

| 钩子函数            | 触发时机                               | 作用                                                                                                                                                              | 注意事项                                                                                                                                                              |
|--------------------|---------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **ngOnChanges**    | 输入属性（用 `@Input()` 装饰的属性）的值发生变化时 | 响应输入属性的变化。 接收 `SimpleChanges` 对象，其中包含当前和之前的属性值。                                                                                                                               | - 在组件初始化时（即第一次执行 `ngOnInit` 之前）也会执行一次。  - 必须显式绑定输入属性才能触发。                                                              |
| **ngOnInit**       | 在 Angular 完成输入属性的初始化之后被调用                 | 执行组件的初始化逻辑。 这是执行初始化任务的最佳位置。 例如，获取数据、设置订阅等。                                                                                                                              | - 只执行一次。 - 避免在这里执行复杂的操作，尽量将复杂逻辑移到其他方法中。                                                                                                |
| **ngDoCheck**      | 在 Angular 检测到组件的输入属性可能发生了变化时             | 实现自定义的变更检测逻辑。  允许你手动检查组件中任何值的变化。                                                                                                                                        | - 每次变更检测周期都会执行，因此要谨慎使用，避免性能问题。 - 尽量避免直接修改视图，这可能会导致 infinite loop。                                                                      |
| **ngAfterContentInit** | 在 Angular 将外部内容投影到组件的视图中后被调用           | 对投影内容进行操作。  例如，查询或操作投影内容。                                                                                                                                                   | - 只执行一次。 - 只有当组件使用了内容投影（`<ng-content>`）时才有用。                                                                                               |
| **ngAfterContentChecked**| 在 Angular 检查完投影到组件的内容之后被调用            | 在每次 Angular 检查完投影内容后执行操作。                                                                                                                                                          | - 每次变更检测周期都会执行，因此要谨慎使用，避免性能问题。                                                                                                 |
| **ngAfterViewInit**  | 在 Angular 完全初始化组件的视图及其子视图之后被调用       | 可以访问模板中定义的子组件和 DOM 元素。  通常用于执行与视图相关的初始化任务，例如操作 DOM 元素或订阅事件。                                                                                                              | - 只执行一次。 - 在组件及其子组件的视图完全初始化之后才执行。                                                                                               |
| **ngAfterViewChecked** | 在 Angular 检查完组件的视图及其子视图之后被调用        | 在每次 Angular 检查完视图后执行操作。                                                                                                                                                              | - 每次变更检测周期都会执行，因此要谨慎使用，避免性能问题。                                                                                                 |
| **ngOnDestroy**     | 在 Angular 销毁组件之前被调用                 | 执行组件的清理逻辑。 这是取消订阅、释放资源等的最佳位置。                                                                                                                                                   | - 这是防止内存泄漏的关键。  - 在组件从 DOM 中移除之前执行。                                                                                                   |


## 4. 总结

理解 Angular 组件的生命周期对于编写高质量的 Angular 应用至关重要。 通过合理地使用生命周期钩子函数，你可以控制组件的行为，管理组件的资源，并优化组件的性能。 记住在`ngOnDestroy` 取消订阅，这是防止 Angular 应用程序中出现内容泄漏的关键。

