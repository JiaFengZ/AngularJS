AngularJs 之 $parse 小记

# $parse 的 api 用法
`$parse(experssion) `
  arguments: 
  	`experssion`:需要被编译的 AngularJS 表达式   
  return:   
	`function(context, locals)`   
	 	`context[object]`:编译解析 AngularJS 表达式的上下文对象，比如 $scope;   
	 	`locals[object]`:本地变量，覆盖 context 上下文对象中的同名变量;   
  返回的函数还可以具有以下三个属性：   
  	`literal[boolean]`:被编译的 Angular 表达式的顶节点是否是一个 javascript 字面量;   
  	`constant[boolean]`:被编译的 Angular 表达式是否全部由常量字面量组成;   
  	`assign[func(context, local)]`:如果返回的函数assignable,则具有该属性，可在给定的上下文中再次修改表达式的值 

# 手动解析一个 AngularJS 表达式
```javascript
angular.module("demo",[])
.controller("demoCtrl", function($scope, $parse){
    $scope.name = 'tony';
    var exp = "'Hello ' + name"; //待解析的表达式
    var parseFunc = $parse(exp); //调用 $parse 解析表达式，返回一个函数
    console.log(parseFunc($scope));// 指定编译的上下文对象为$scope, 因此表达中 name 变量被替换为 tony,结果输出为：'Hello tony'

    var myScope = {
	name: 'image'
    };
    console.log(parseFunc(myScope)); // 指定编译的上下文对象为myScope,name 变量被替换为 image,结果输出为：'Hello image'
    console.log(parseFunc($scope, myScope)); // 指定编译的上下文对象为$scope,但是$scope中 name 变量值被myScope覆盖了,结果输出
                                             //为：'Hello image'
}); 
```

# $parse 在 directive 中的使用
```html
<div my-attr="obj.name" parse-attr>testing</div>
```
```javascript
app.directive('parseAttr',function($log,$parse){
    return function(scope, ele, attrs){

        var model = $parse(attrs.myAttr);
        //model现在是一个函数，可以调用它来获取表达式的值
        //下面这行代码将会输出作用域中obj.name的值 
        $log.log(model(scope));

        ele.bind('click', function(){
        	model.assign(scope,'New name');
        	scope.$apply();
        })
    }
});
```
