AngularJS 的 controller as 语法

# ng-controller
在 AngularJS 中 controller 的作用就是初始化 $scope 对象的状态，给 $scope 对象添加行为和属性。使用 controller 你首先要在 module 注册定义一个 controller 构造函数，比如 `angular.module('myModule', []).controller('myController', createController)`创建了一个名为 myModule 的 module，在这个 module 上定义了一个名为 myController 的controller，注意此时并没有生成 controller 实例，只是声明了一个 controller，在需要时才会调用 createController 方法构造出一个 controller 实例。  

问题就是什么时候才会生成一个 controller 实例？一般而言我们会使用 ng-controller 指令关联 view 和 controller，当声明 ng-controller="myController" 指令属性的元素被创建并添加到 DOM 文档中显示时，就会调用 createController 方法构造出一个 controller 实例，创建出一个 scope 作用域，可以通过 $scope 把当前 controller 实例的作用域注入到 controller 中，这里值得留意的是 view 应该要跟 controller 一一对应，不同的 view 要使用不同的 controller，像以上直接使用 ng-controller="myController" 只会调用一次构造函数生成唯一一个 controller 示例，即时同时创建了多个声明了 ng-controller="myController" 的 view，这些视图都将共享同一个 controller，这些不同的 view 上的都将共享这个 controller 中的数据、行为、状态。通过下面的代码说明此点：  
```html
<div ng-app="app">
	<label>控制器1</label>
	<div ng-controller="myController">
		<span>{{number}}</span>
		<button ng-click="increaseNumber()">+</button>
	</div>
	<label>控制器2</label>
	<div ng-controller="myController">
		<span>{{number}}</span>
		<button ng-click="increaseNumber()">+</button>
	</div>
</div>
```

```javascript
angular.module('app', []).controller('myController', ['$scope', function($scope) {
	$scope.number = 0;
	$scope.increaseNumber = function() {
		$scope.number++; 
	};
}])
```
以上代码在 html 中创建了两个使用相同 controller 的 div，都能显示当前 $scope 对象中的 number 属性值，并且可通过 increaseNumber 方法更改 $scope 对象的 number 属性值，如果去做一下试验会发现点击任意一个 div 中的 button 按钮，都能增加 number 的数字值，并且两个 div 显示的 number 是一样的，这就说明两个 div 是共享同一个 controller 实例的数据行为和状态的。

# 如何生成多个 controller 实例？
Angular 的最佳实践是保持 controller 的专一，不同的 view 要使用不同 controller。有时候有显示多个相同的 view 的需求，这些 view 相同的意思就是它们的交互逻辑和行为都是一样的，只是数据状态不同，比如说有一个商品列表，点击某个商品就会在某个区域添加一个标签页显示这个商品的详情，如果点击选取了多个商品，页面上就会有多个标签页显示这些不同商品的详情，易见这些标签页的行为交互和展示数据属性都是相同的，完全可视为相同的 view，只要用一个 controller 就可以定义并控制这些 view 的数据和行为，问题就来了，如果每个标签页都使用 ng-controller="myController" 去声明使用同一个 controller，这几个标签页都将共享同一个 controller 实例，但它们要显示的数据属性的值是不一样，使用 ng-controller="myController" 显然不能满足要求，这时候可以使用 controller as 语法，controller 实例将会是 $scope 对象上的一个属性，因此能创建多个不同的 controller 实例。可看以下示例：
```html
<html ng-app="app">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <script src="jquery-2.1.3.min.js"></script>
    <script src="angular.min.js"></script>
</head>
<body>
<div ng-controller="parentController">
    <ul>
        <li ng-repeat="good in goodList" ng-click="addGood(good)">{{good.name}}</li>
    </ul>
    <div good-tabs></div>
</div>
</body>
<script>
    angular.module('app', []).directive('goodTabs', ['$compile', function($compile) {
    return {
        restrict: 'A',
        scope: true,
        link: function(scope, ele, attrs) {
            scope.$watchCollection('goods', function(n) {
                if (scope.goods.curGood) {
                    if (!scope.goods[scope.goods.curGood.id]) {
                        $('#' + scope.goods.curGood.id).remove();
                    } else {
                        var div = document.createElement('div');
                        div.setAttribute('ng-controller', 'childController as '+ scope.goods.curGood.ctrName);
                        div.setAttribute('id', scope.goods.curGood.id);
                        var HTML = '<div>' +
                                       '<button ng-click="scope.close()">关闭</button>' +
                                    '</div>' +
                                    '<div>'+
                                        '<label>名称：</label><span>{{scope.good.name}}</span>' +
                                        '<label>价格：</label><span>{{scope.good.price}}</span>' +
                                    '</div>'
                        HTML = HTML.replace(/scope/g, scope.goods.curGood.ctrName);
                        div.innerHTML = HTML;
                        $compile(div)(scope);
                        ele.append(div);
                    }
                }
                                
            });
        }
    }
}]).controller('parentController', ['$scope', function($scope) {
    $scope.goodList = [{
        id: 1,
        name: '商品1',
        price: '12元',
        ctrName: 'good1'
    },{
        id: 2,
        name: '商品2',
        price: '13元',
        ctrName: 'good2'
    },{
        id: 3,
        name: '商品3',
        price: '11元',
        ctrName: 'good3'
    }];

    $scope.goods = {};

    $scope.addGood = function(good) {
        $scope.goods[good.id] = good;
        $scope.goods.curGood = good;
    };

    $scope.removeTab = function() {

    };
}]).controller('childController', ['$scope', function($scope) {
    var ctrl = this;
    ctrl.good = angular.copy($scope.goods.curGood);
    ctrl.close = function() {
        delete $scope.goods[ctrl.good.id];
        $scope.goods.curGood = ctrl.good;
    };
}]);
</script>
</html>
```

# 在路由中使用 controller as 语法
```javascipt
angular
  .module('app')
  .config(function($routeProvider) {
  	$routeProvider.when('/avengers', {
          templateUrl: 'goods.html',
          controller: 'goodsCtrl',
          controllerAs: 'good'
      });
}).controller('goodsCtrl', function() {
	var ctrl = this;
	this.name = 'apple';
});
```
```html
<!-- good.html -->
<div>{{good.name}}</div>
```

# 在 directive 中使用 controller as 语法([改编自angular代码规范](http://www.reqianduan.com/1722.html))

```html
<div my-example max="77"></div
```

```javascript
angular
  .module('app')
  .directive('myExample', function() {
  var directive = {
      restrict: 'EA',
      templateUrl: 'example.html',
      scope: {
          max: '='
      },
      link: function(scope, el, attr, ctrl) {
      	console.log('LINK: scope.vm.min = %s', scope.vm.min); // '3'
      	console.log('LINK: scope.vm.max = %s', scope.vm.max);// '17'
  	  },
      controller : exampleController,
      controllerAs: 'vm',
      bindToController: true // vm 绑定到父作用域，因此scope.vm.max值继承 `max="17"` 的值
  };

  return directive;
}).controller('exampleController', ['$scope', function($scope) {
	var vm = this;
  	vm.min = 3;
}]);
```

```html
<!-- example.html -->
<div>hello world</div>
<div>max={{vm.max}}<input ng-model="vm.max"/></div>
<div>min={{vm.min}}<input ng-model="vm.min"/></div>
```
