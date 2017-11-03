AngularJS 之 $compile与指令编译过程

# 1. $compile 编译过程
AngularJS 内置了很多指令扩展 html 的语法，增强了我们对元素其表现和行为的控制能力，比如说 ng-show/ng-hide 可以根据表达式的值控制元素的可见性，但是仅靠内置的指令并不能完全满足我们的需求，这时候就需要编写自定义指令，这种扩展 html 语法的的做法需要通过编译(compile)来实现，在 AngularJS 中 compile 是一个遍历收集元素属性的过程，可分为两个阶段：
 1. compile(编译)：遍历 DOM 元素，收集所有指令(directives),执行指令的 complie 方法，返回一个链接函数(linkng function)
 2. link(链接): 把 DOM 元素链接到 scope 树，为 DOM 元素添加事件监听，监听 scope 模型属性值的变化,更新 DOM

 HTML 编译过程详解(翻译自 [angular.js 文档](https://github.com/angular/angular.js/blob/master/docs/content/guide/compiler.ngdoc))：
 1. $compile 遍历 DOM 匹配指令。把匹配到的每一个指令添加到该 DOM 元素的指令列表，一个元素可以匹配多个指令。
 2. 一旦所有指令匹配完毕后，compiler 会根据指令的优先级(pirority)进行排序。每个指令的 compile 方法将会依次执行，compile 方法可以修改 DOM ，每个 compile 方法执行后都返回一个 link 方法，所有这些 link 方法会组合到一个大的 link 方法中，这个组合的 link 方法依次调用每个指令返回的链接函数。
 3. $compile 调用这个组合的链接函数，为元素注册事件监听，并且监听 scope 模型。结果就是在 scope 和 DOM 之间建立了一个动态绑定，任何在编译后的 scope 的模型值的变化都将反映在 DOM 中。

 以下代码展示了 $compile 的使用过程：

 ```javascript
 var htm = '<div>{{expression}}</div>'
 //包装成 DOM 元素
 var template = angular.element(htm);
 //使用 $compile 编译元素,返回一个链接函数
 var linkFn = $compile(template);
 //把 DOM 元素链接到 scope
 var ele = linkFn(scope);
 //添加到html文档
 document.body.appendChild(ele);
 ```

# 2. $compile 用法
$compile(ele, transcludeFn, maxPriority):
ele: 要编译的 DOM 元素;
transcludeFn: 是一个函数 function(cloneEle, scope) {},此文不作详解;
maxPriority: 最大优先级，优先级大于此值的指令将不会编译;


# 3. 使用示例
## 3.1. 动态编译显示html文档片段

```javascript
angular.module('compileExample', []).directive('compile', function($compile) {
      return function(scope, element, attrs) {
        scope.$watch(
          function(scope) {
            return scope.$eval(attrs.compile);
          },
          function(value) {
            $compile(element.contents())(scope);
          }
        );
      };
    }).controller('GreeterController', ['$scope', function($scope) {
	    $scope.testFun = function() {
			  console.log("hello");
		  };
		  $scope.test = "hello";
	    $scope.htm = "<div ng-click='testFun()'>{{test}}</div>";
  	}]);
```

```html
<div ng-controller="GreeterController">
	<div compile="htm"></div>
</div>
```

## 3.2. Transcluded 指令的应用(参考来源 [angular.js 文档](https://github.com/angular/angular.js/blob/master/docs/content/guide/compiler.ngdoc))
考虑如下的一个指令：

```html
<div ng-controller="demoCtrl">
  <button ng-click="show=true">show</button>

  <dialog title="Hello {{username}}"
          visible="show"
          on-cancel="show = false"
          on-ok="show = false; doSomething()">
     Body goes here: {{username}} is {{title}}.
  </dialog>
</div>
```

```html
//dialog.html
<div ng-show="visible">
  <h3>{{title}}</h3>
  <div class="body" ng-transclude></div>
  <div class="footer">
    <button ng-click="onOk()">Save changes</button>
    <button ng-click="onCancel()">Close</button>
  </div>
</div>
```

```javascript
angular.module('demo', []).directive('dialog', [function() {
	var dialog = {
		restrict: 'AE',
		transclude: true,//设置可包含子元素
		replace: true,
		templateUrl: 'dialog.html',
		scope: {
		    title: '@',             // the title uses the data-binding from the parent scope
		    onOk: '&',              // create a delegate onOk function
		    onCancel: '&',          // create a delegate onCancel function
		    visible: '='            // set up visible to accept data-binding
  		}，
  		link: function() {

  		}
	};
}]).controller('demoCtrl', ['$scope', function($scope) {
	$scope.show = true;
	$scope.username = 'demo';
	$scope.doSomething = function() {
	  console.log('hello!you has clicked OK!');
	}
}]);
```

经过指令的编译处理后的html将会是这样：

```html
<div ng-controller="demoCtrl">
    <button ng-click="show=true">show</button>
    <div ng-show="visible">
	  <h3>Hello demo</h3>
	  <div class="body" ng-transclude>Body goes here: demo is Hello demo.</div><!-- dialog指令元素的子元素被包含在这里 -->
	  <div class="footer">
	    <button ng-click="onOk()">Save changes</button>
	    <button ng-click="onCancel()">Close</button>
	  </div>
	</div>
</div>
```

## 3.3. 避免重复编译
看如下指令，这个指令的作用是：当鼠标移入元素时显示 'My Hint'，鼠标移出时不显示，为了添加 mouseenter 和 mouseleave 事件，使用了 $compile 对元素进行了二次编译，存在的问题就是，如果该元素有子元素时，会对所有子元素进行重新编译，可能会导致指令工作异常，出现性能和内存泄漏等问题

```javascript
angular.module('app').directive('addMouseover', function($compile) {
  return {
    link: function(scope, element, attrs) {
      var newEl = angular.element('<span ng-show="showHint"> My Hint</span>');
      element.on('mouseenter mouseleave', function() {
        scope.$apply('showHint = !showHint');
      });

      attrs.$set('addMouseover', null); // 阻止编译循环编译
      element.append(newEl);
      $compile(element)(scope); // 进行二次编译
    }
  }
})
```

因此，需要做一下改进，把 `$compile(element)(scope)` 改成 ` $compile(newEl)(scope)`，只编译新增的元素。

