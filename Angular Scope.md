本文翻译自[angular.js scope.ngdoc](https://github.com/angular/angular.js/blob/master/docs/content/guide/scope.ngdoc)
# scope 是什么？
Scope 是持有应用模型(model)引用的一个对象，是一个可执行的上下文环境，Scopes 有一个类似 DOM 可实现上下级继承嵌套树形的结构，可以监听和传播事件。

## Scope 的特点

- Scopes 提供 APIs($watch) 来监听 model 的改变。
- Scopes 提供 APIs($apply) 把在 Angular 执行环境之外引起的 model的改变更新到视图(view)。
- Scopes 通过层级嵌套的方式实现 model 属性的访问和继承共享。嵌套的 scopes 要么是子 scope，要么是独立隔离的 scope，普通的子 scope 继承它的父 scope 的属性，独立隔离的 scope 不会继承父域的属性。
- Scopes 为 angular 表达式提供执行的上下文环境。例如 `{{username}}` 脱离作用域是没有意义的，只有在一个特定的 scope 中定义了 `username` 属性，这个表达式才会有相应的值。

## 作为数据模型的 Scope 
Scope 是 controller 和 view 的中间管道，在模版(template)链接(linking)阶段，指令(directive)会建立对 scope 中的表达式的监听($watch)，$watch 允许指令监听到任何的属性的变化，及时更新 DOM。 

controllers 和 directives 不会产生直接关联，但是两者都可以和 scope 建立关联，这样就把 controller 跟 directive 以及 DOM 隔离开来，这可以提升应用的可测试性。

```html
<example module="scopeExample" name="scope-data-model">
  <file name="script.js">
    angular.module('scopeExample', [])
      .controller('MyController', ['$scope', function($scope) {
        $scope.username = 'World';

        $scope.sayHello = function() {
          $scope.greeting = 'Hello ' + $scope.username + '!';
        };
      }]);
  </file>
  <file name="index.html">
    <div ng-controller="MyController">
      Your name:
        <input type="text" ng-model="username">
        <button ng-click='sayHello()'>greet</button>
      <hr>
      {{greeting}}
    </div>
  </file>
</example>
```
在上面的例子中可以看到 `MyController` 在 scope 中给 `username` 对象分配了 `World` 属性，scope 监听到 `input` 输入框的模型值，然后把已经定义好的 username 渲染显示在 html 文档中，这就是 controller 向 scope 注入数据的过程。

类似的，controller 可以给 scope 定义行为，比如上面的 `sayHello` 方法，当用户点击 `greet` 按钮时就会触发这个方法。`sayHello` 方法访问 `username` 属性并且创建了一个 `greeting` 属性，这说明 scope 中的绑定到 html 视图的模型值会自动更新。

`{{greeting}}`的渲染涉及下面两方面：  
  * 收集所有定义了`{{greeting}}`表达式的DOM节点相关的 scope，这个例子里面注入到 `MyController`的 scope 就是该文本节点的 scope（稍后将介绍 scope 的继承）  
  * 计算所有收集到的 `greeting` 表达式在 scope 的值，把结果显示在 DOM 文本元素中。

scope 和它的属性是用来驱动渲染视图(view)的，scope 就是视图(view)的唯一信任源。

controller 和 view 的分离保证了 view 的可测试性，这样允许我们在忽略渲染细节的情况下就能测试程序的行为。

```javascript
it('should say hello', function() {
    var scopeMock = {};
    var cntl = new MyController(scopeMock);

    // Assert that username is pre-filled
    expect(scopeMock.username).toEqual('World');

    // Assert that we read new username and greet
    scopeMock.username = 'angular';
    scopeMock.sayHello();
    expect(scopeMock.greeting).toEqual('Hello angular!');
  });
```

## Scope 继承

AngularJS 应用只有一个根作用域(root scope)，但是可以有很多个子作用域(child scope)。

应用拥有多个 scopes 是因为指令(directive)能创建新的子作用域(child scope)。新创建的 scope 会被添加到 scope 树中，作为父作用域的子作用域，这就创建出一个和 DOM 结构平行的树形作用域。

AngularJS 计算`{{name}}`表达式的值时，首先去当前所在的作用域(scope)寻找 `name` 属性，如果没找到，就继续在父作用域(parent scope)中寻找直到根作用域(root scope)，这其实就是基于原型链的继承，子作用域的原型就是他的父作用域。

下面的例子说明了 scopes 的原型链继承关系：
```html
<example module="scopeExample" name="scope-hierarchies">
  <file name="index.html">
  <div class="show-scope-demo">
    <div ng-controller="GreetController">
      Hello {{name}}!
    </div>
    <div ng-controller="ListController">
      <ol>
        <li ng-repeat="name in names">{{name}} from {{department}}</li>
      </ol>
    </div>
  </div>
  </file>
  <file name="script.js">
    angular.module('scopeExample', [])
      .controller('GreetController', ['$scope', '$rootScope', function($scope, $rootScope) {
        $scope.name = 'World';
        $rootScope.department = 'AngularJS';
      }])
      .controller('ListController', ['$scope', function($scope) {
        $scope.names = ['Igor', 'Misko', 'Vojta'];
      }]);
  </file>
  <file name="style.css">
    .show-scope-demo.ng-scope,
    .show-scope-demo .ng-scope  {
      border: 1px solid red;
      margin: 3px;
    }
  </file>
</example>

<img class="center" src="img/guide/concepts-scope.png">
```

AngularJS 会自动在 scopes 依附的元素上添加 `ng-scope` 类名，本例中`<style>`把新作用域(new scope)所在的元素高亮显示了。为了计算`{{name}}`表达式的值，子作用域是必须的，在不同的子作用域中`{{name}}`会有不同的值，类似的`{{department}}`属性值是继承自根作用域的，因为只有跟作用域定义了 `department`属性。

## 收集 DOM 中的 scopes

Scopes 作为的`$scope`的数据属性依附到 DOM 中，`ng-app`指令定义了跟作用域的作用的元素节点，一般`ng-app`放在`<html>`节点，但是也可以放在其他元素节点上，只允许 AngularJS 控制一部分 DOM 文本片段。

可以这样调试检查作用域： 
1.右键检查元素，打开浏览器调试器，使用元素探测器选中节点元素高亮显示该节点元素。
2. 调试器允许你使用 `$0`变量获取当前被选中的元素。
3. 用 `angular.element($0).scope()` 获取当前元素的作用域对象(scope).

## Scope 事件传播

Scopes 能够以类似 DOM 事件的方式传播事件，事件可以通过 `$broadcast` 广播到子作用域中或者通过 `$emit` 通知父作用域。

```html
<example module="eventExample" name="scope-events-propagation">
  <file name="script.js">
    angular.module('eventExample', [])
      .controller('EventController', ['$scope', function($scope) {
        $scope.count = 0;
        $scope.$on('MyEvent', function() {
          $scope.count++;
        });
      }]);
  </file>
  <file name="index.html">
    <div ng-controller="EventController">
      Root scope <tt>MyEvent</tt> count: {{count}}
      <ul>
        <li ng-repeat="i in [1]" ng-controller="EventController">
          <button ng-click="$emit('MyEvent')">$emit('MyEvent')</button>
          <button ng-click="$broadcast('MyEvent')">$broadcast('MyEvent')</button>
          <br>
          Middle scope <tt>MyEvent</tt> count: {{count}}
          <ul>
            <li ng-repeat="item in [1, 2]" ng-controller="EventController">
              Leaf scope <tt>MyEvent</tt> count: {{count}}
            </li>
          </ul>
        </li>
      </ul>
    </div>
  </file>
</example>
```

## Scope 生命周期

待续。。。
