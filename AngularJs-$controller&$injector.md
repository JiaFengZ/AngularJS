## 使用 $controller 实例化 controller 
$controller 可用于生成一个 controller 实例，其用法如下：  
`$controller(constructor, locals)`    
`constructor`: `function` 或者 `string`，如果 constructor 是一个函数，那么该函数作为 controller 的构造函数，如果 constructor 是一个 string，通过以下步骤找到 controller 的构造函数：   
1、检查 $controllerProvider 注册的所有 controller 中是否存在以该 string 命名的 controller；   
2、若不存在，则检查该 string 所在的当前作用域中是否返回一个构造函数；  
3、若不存在，如果 $controllerProvider 允许全局注册 controller，则检查 window[constructor]是否存在。   
`locals`: controller 注入的依赖
```javascript
app.module('app', []).controller('demoController', ['$scope', function($scope) {
	
}]);
//通过 $controller 手动实例化 demoController
var scope = $rootScope.$new();
var controller = $controller('demoController', {$scope: scope});
```

## 使用 $injector 实例化对象
$injector 是 angular 中实例化对象、注入方法、加载模块的内部服务，$controller 实例化 controller 实例对象时底层调用的就是 $injector 服务。  
### 实例化构造函数
```javascript
function demoController($scope, $http) {
	....
}
$injector.invoke(demoController);
```
以上就是通过 $injector.invoke 实例化一个 controller 对象，需要注意的是上面 $injector 是通过解析源码的方式寻找 demoController 的依赖 `$scope` 和 `$http` ，如果代码被压缩，则寻找依赖会失效，为了避免这种情况，可以给构造函数定义 `$inject` 帮助 `injector` 寻找依赖：
```javascript
function demoController($scope, $http) {
	....
}
demoController.$inject = ['$scope', '$http'];
$injector.invoke(demoController);
```
或者直接内联依赖模块的数组：
```javascript
$injector.invoke(['$scope', '$http'], function demoController($scope, $http) {
	
});
```
还可以调用 $injector.get(name) 方法获取名为 name 的服务实例：
```javascript
app.module('app', []).service('myService', function() {});
var service = $injector.get('myService');//获取自定义服务

var service = $injector.get('$http'); //获取 angular 内置的服务实例
```
