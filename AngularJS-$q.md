AngularJS 之 $q 服务与ES6的 promise

在 Javascript 中，有一种异步处理模式叫做 promise，AngularJS 中的 $q 服务是其一个简单的实现。
# 1 $q 的使用
##  1.1 使用 $q 创建一个 promise 对象
使用 $q.defer():
```javascript
function myPromise() {
	var deferred = $q.defer(); //创建一个 defer 延迟对象
	$timeout(function() {
		if (Math.random() > 0.5) {
			deferred.resolve("resolve")
		} else {
			deferred.reject("reject")
		}
	}, 1000);
	return deferred.promise;  //返回一个 promise 对象
}
myPromise.then(function(res) {  //resolve
	console.log(res); // 'resolve'
}, function(res) { //reject
	console.log(res); // 'reject'
});
```
使用 $q(function(resolve, reject) {}) 构造：
```javascript
var myPromise = $q(function(resolve, reject) {
	$timeout(function() {
		if (Math.random() > 0.5) {
			deferred.resolve("resolve")
		} else {
			deferred.reject("reject")
		}
	}, 1000);
});
myPromise.then(function(res) {  //resolve
	console.log(res); // 'resolve'
}, function(res) { //reject
	console.log(res); // 'reject'
});
```
## 1.2 $q.all([promise...])
$q.all 接收一个一个数组作为参数，数组元素为若干 promise 对象，返回一个 promise，如果参数数组中所有 promise 都成功(resolve)，则其resolve，如果有任何一个 reject 则其结果为 reject。
```javascript
var p1 = $http.get(url1);
var p2 = $http.get(url2);
var p3 = $q.all([p1, p2]);
p3.then(function(res) { // p1 和 p2 都是 resolve 状态
	console.log(res); 
}, function(res) { // p1 或 p2 任意一个是 reject 状态
	
})
```
## 1.3 $q.when()
$q.when() 接收的参数可以是 promise，then-able 的对象或者任意一个值，返回一个 promise
```javascript
var p1 = $http.get(url);
$q.when(p1).then(function() {
	
}, function() {
	
});
```

## 1.4 使用 deferred.notify() 监听状态
```javascript
var deferred = $q.defer();
var promise = deferred.promise();

$http({
	method: 'POST',
	url: url,
	data: formData,
	uploadEventHandlers: {
		progress: function(e) { //上传进度
	    	deferred.notify('progress-->' + e);
		}
	},
}).success(function(res) {
	deferred.resolve(res);
}).error(function(err) {
	deferred.reject(err);
});

promise.then(function(res) {//resolve
	console.log(res);
}, function(err) {//reject
	console.log(err);
}, function(progress) {//notify
	console.log(progress);
});
```

# 2 promise 在 Angular 中的使用案例
## 2.1 将多个异步任务嵌套回调执行的代码利用 promise 改写成链式调用
```javascript
// 嵌套回调方式
$http.get(url1).success(function() {
	$http.get(url2).success(function() {
		$http.get(url3).success(function() {
			....
		})
	})
})

//promise 链式调用方式
$http.get(url1).then(function() {
	...
	return $http.get(url2);
}).then(function() {
	...
	return $http.get(url3);
}).then(function() {
	
})
```
## 2.2 将回调流程的代码改造成 promise 模式
```javascript
// 回调的代码

```

## 2.3 多个异步任务的并行处理
```javascript
$q.all([$http.get(url1), $http.get(url2), $http.get(url3)]).then(function(results) {
    console.log('result 1', results[0]);
    console.log('result 2', results[1]);
    console.log('result 3', results[2]);
});
```

## 2.4 Angular-UI 的 bootstrap 的 modal 模态框的 promise 应用
```javascript
var promise = $modal.open({  //$modal.open().result返回了一个promise对象
	  templateUrl: '/templates/modal.html',
    controller: 'ModalController',
    controllerAs: 'modal',
    resolve: {
    	data: function() { // data是要注入模态框 controller 的数据，这里执行后返回的是一个 promise,模态框会等待$http.get(url)成功返回后才打开
			  return $http.get(url);
    	}
    }
}).result;
promise.then(function() { //关闭时调用(close)
	
}, function() {//取消时调用(dismiss)
	
});
```

## 2.5 Angular 路由模块 ui-view 使用 promise 机制
route resolve 允许 promise 在 controller 的逻辑执行前解决(resolve)，从而为 controller 加载注入数据，例如以下这段代码将在路由跳转之前执行一个 promise ，如果 promise 状态为 resolve 则可跳转路由视图，如果 promise 状态为 reject 则取消路由跳转。
```javascript
//app.js
angular
  .module('app')
  .config(config);

function config ($urlRouteProvider) {
  $urlRouteProvider
      .when('/demo', {
          templateUrl: 'demo.html',
          controller: 'demoCtrl',
          resolve: {
              moviesPrepService: function(movieService) {
                  return movieService.getMovies();
              }
          }
      });
}

// demo.js
angular
  .module('app')
  .controller('demoCtrl', ['moviesPrepService', function(moviesPrepService) {

  }]);
```
## 2.6 加载图片
```javascript
var preloadImage = function (path) {
  return $q(function(resolve, reject) {
    var image = new Image();
    image.onload  = resolve;
    image.onerror = reject;
    image.src = path;
  });
};

preloadImage('my.png').then(function() {
	
}, function() {
	
});
```
# 3 ES6 的 Promise 规范以及使用(内容摘录自[ES6 Promise-阮一峰](http://es6.ruanyifeng.com/#docs/promise))
Promise 是异步编程的一种解决方案，ES6 提供了原生的 promise 对象，可以看作一个保存异步操作结果的容器，Promise 对象具有以下特点： 
1. 对象的状态不受外界影响，Promise 对象代表一个异步操作，有三种状态：pending,fulfilled和rejected; 
2. promise 状态的转变不可逆。要么从pending变为fulfilled，要么从pending变为rejected。 
构造一个 promise 对象：
```javascript
var p = new Promise(function(resolve, reject) {
	if () {
		resolve();
	} else {
		reject();
	}
});
```
resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。
Promise 实例生成后，可以使用 then 方法指定 resolve 和 reject 状态的回调函数:
```javascript
p.then(function() {
	//resolve
}, function() {
	//reject
});
```

## 3.1 Promise 若干方法
### 3.1.1 Promise.prototype.catch()
Promise.prototype.catch方法是用于指定发生错误时的回调函数
```javascript
getJSON('/posts.json').then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});

getJSON('/post/1.json').then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
});
```
### 3.1.2 Promise.all()
Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例.
```javascript
var p = Promise.all([p1, p2, p3]);
```
### 3.1.3 Promise.race()
```javascript
var p = Promise.all([p1, p2, p3]);
```
上面代码中，只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。
### 3.1.4 Promise.resolve()
将现有对象转为Promise对象:
```javascript
var jsPromise = Promise.resolve($.ajax('/whatever.json'));
```
上面代码将jQuery生成的deferred对象，转为一个新的Promise对象。
如果参数是Promise实例，那么Promise.resolve将不做任何修改、原封不动地返回这个实例。 
参数是具有then方法的对象，比如下面:
```javascript
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
  console.log(value);  // 42
});
```
如果参数是一个原始值，或者是一个不具有then方法的对象，则Promise.resolve方法返回一个新的Promise对象，状态为resolved。 
调用时不带参数，直接返回一个resolved状态的Promise对象。
### 3.1.5 Promise.reject()
Promise.reject(reason)方法也会返回一个新的 Promise 实例，该实例的状态为rejected。

## 3.2 利用 Promise 封装 AJAX
```javascript
var get = function(url) {
	var promise = new Promise(function(resolve, reject) {
		var client = new XMLHttpRequest();
		client.open('GET', url);
		client.onreadystatechange = handler;
		client.responseType = 'json';
		client.setRequestHeader('Accept', 'application/json');
		client.send();

		functio handler() {
			if (this.readystate != 4) {
				return;
			}
			if (this.status == 200) {
				resolve(this.response);
			} else {
				reject(new Error(this.statusText));
			}
		}
	});
	return promise;
};

get("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```

# 4 ES7控制异步流程新的解决方案 async/await
前文提及到 promise 的链式调用：
```javascript
const gen = function() {
	p1().then(function() {
		...
		return p2();
	}).then(function() {
		...
		return p3();
	}).then(function() {
		
	})
};
```
如果使用 async/await 则可改写成同步流程代码：
```javascript
const gen = async function () {
  await p1();
  ...
  await p2();
  ...
  p3();
};
```
以上代码仅仅是展示了 async/await 的简单实用，更多细节不作详述。

本文内容参考整合来源：    
[Promise 的前世今生和妙用-破狼](http://www.cnblogs.com/whitewolf/p/promise-best-practice.html)   
[Angular $q 完全指南](http://blog.csdn.net/zhuyucheng123/article/details/50597291)   
[ES6 Promise-阮一峰](http://es6.ruanyifeng.com/#docs/promise)   
[八段代码彻底掌握 promise-掘金](https://juejin.im/post/597724c26fb9a06bb75260e8) 
