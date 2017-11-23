
# angular 的 scope 事件机制

angular 的作用域结构是类似 DOM 树的结构，拥有父子嵌套的层级关系，angular 实现了类似 DOM 事件的事件模型，controller 之间事件通讯通过 $on、$emit、$broadcast来实现。
$on 在当前作用域注册监听事件，$emit 向父级作用域冒泡传递事件，$broadcast 向子作用域广播事件
```html
<div ng-controller="parentController">
	<button ng-click="broadcastToChild()"></button>
	<div ng-controller="childController1">
		<button ng-click="emitParent()"></button>
	</div>
	<div ng-controller="childController2">
		<button ng-click="emitParent()"></button>
	</div>
</div>
```
```javascript
app.module('app', []).controller('parentController', ['$scope', function($scope) {
	
	$scope.$on('parentEvent', function(event, data) { //当前父作用域绑定事件，可在子作用域中$emit触发
		console.log('form' + data.source);
	});

	$scope.broadcastToChild = function() {//向子作用域广播事件
		$scope.$broadcast('childEvent', {source: 'parentEvent'});
	};
	
}]).controller('childController1', ['$scope', function($scope) {
	$scope.emitParent = function() { //触发父作用域事件
		$scope.$emit('parentEvent', {source: 'childController1'});
	};

	$scope.$on('childEvent', function(event, data) {//当前子作用域绑定事件，接收到父作用域广播事件可触发
		console.log('form' + data.source);
	});
}]).controller('childController2', ['$scope', function($scope) {
	$scope.emitParent = function() {
		$scope.$emit('parentEvent', {source: 'childController2'}); //触发父作用域事件
	};

	$scope.$on('childEvent', function(event, data) { //当前子作用域绑定事件，接收到父作用域广播事件可触发
		console.log('form' + data.source);
	});

}])
```
值得注意的是$on 绑定事件的 event 参数具有以下几个属性：  
		event.targetScope：传播事件的原始作用域  
		event.currentScope: 事件所在的当前作用域  
		event.name: 事件名称  
		event.stopPropagation(): 阻止事件继续传播，只对$emit有效，不能停止$broadcast事件广播

# 实现一个事件订阅发布模型
参考阅读： 
[js中的一对多 - 订阅发布模式](https://segmentfault.com/a/1190000004315216)  
[奇舞特训营 JavaScript 常用设计模式和组件开发实战](https://ppt.baomitu.com/d/f4075f50#/15)
```javascript
function Scope(){
  this.subscribers = {};
}

Scope.prototype = {
  emit: function(name, data){ // 发布事件
    var subscribers = this.subscribers[name];
    subscribers.forEach(function(subscriber){
      subscriber(data);
    });
  },

  on: function(name, target, fn){ //订阅事件
    this.subscribers[name] = this.subscribers[name] || [];
    this.subscribers[name].push(fn.bind(target));
  },

  remove: function(name, fn) {  //取消订阅事件
    var subscribers = this.subscribers[name];		
    if (!subscribers) {
        return false;
    }

    if (!fn) {
        subscribers && (subscribers.length = 0);
    }
    else {
        for (var i = subscribers.length - 1; i >= 0; i--) {
            var _fn = subscribers[i];
            if (_fn === fn) {
                // 删除订阅回调函数
                this.subscribers[type].splice(i, 1);
            }
        }
    }
  }
};

var scope = new Scope();
var jeny = {
	word: 'goodbye',
	
};

function say() {
		console.log(this.word);
}

scope.on('sayGoodbye', jeny, say);//向 scope 订阅了 sayGoodbye 事件

scope.emit('sayGoodbye'); //发布 sayGoodbye 事件

scope.remove('sayGoodbye', say); //取消订阅 sayGoodbye 事件
```
