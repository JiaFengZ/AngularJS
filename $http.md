AngularJS之$http服务

# 1 $http简介

$http 是AngularJS的内置核心服务，通过浏览器的XMLHttpRequest对象向服务器请求数据，其使用如下：

```javascript
$http({
	method: '',//请求方法，常用POST/GET/DELETE
	url: '',//请求地址
	params: {} || '', //对象或者字符串，作为get请求的参数，拼接在url上
	data: {},//对象，一般作为post请求的数据参数
	headers: {  //设置请求头
		Content-Type: ''
	},
	cache: true,//是否开启缓存，get请求有效。值为true或者$cacheFactory创建的实例
	responseType: '',//字符串，响应数据类型，值为text/arraybuffer/blob/json/document
	timeout: 2000//毫秒数值，设置请求超时
});
```

$http()执行返回一个promise，用于请求返回后的异步操作，常用有以下两种书写方式：

```javascript
//scuccess是请求成功后的执行逻辑，error是请求失败后的执行逻辑
$http().success(function(data) {

}).error(function() {

});

$http().then(function(res) { //请求成功

}, function(res) {  //	请求失败

});
```

其中第一种方式 success 和 error 方法在 AngularJS 的1.6新版本中已废弃，其回调函数的参数data就是请求返回的数据。第二种方式，$http()执行返回的是一个promise，具有then方法，then方法里面的第一个方法请求成功时触发，第二个方法请求失败时调用，其中回调方法的参数res是一个对象，包含以下几个属性：
	data: 返回的数据
	status: 数值，响应的http状态码
	headers: 函数，相应头的getter函数
	config: 对象，请求的配置对象
	statusText：字符串，响应的http状态文本


$http常用的简化方法有$http.get(),$http.post(),$http.delete(),$http.put()等

# 2 $http使用示例

## 2.1 文件上传

angular 1.5.5版本后$http新增了eventHandler和uploadEventHandlers方法，eventHandler获取文件下载进度信息，uploadEventHandlers获取文件上传进度信息（参考自[路易斯ajax知识体系大梳理](https://juejin.im/post/58c883ecb123db005311861a)）

```javascript
var file = document.getElementById('file');
var formData = new FormData();
formData.append('file',files[0]);

$http({
	method: 'POST',
	url: url,
	data: formData,
	uploadEventHandlers: {
		progress: function(e) { //上传进度
	    	console.log('UploadProgress -> ' + e);
		}
	},
}).success(function(res) {
	console.log(res);
}).error(function(err, status) {
	console.log(err);
});
```

## 2.2 读取图片二进制流加载图片

```javascript
var img = document.createElement("img");
$http({
	method: 'GET',
	url: url,
	responseType: 'arraybuffer'
}).success(function(data) {
	var blob = new Blob([data], {type: 'image/png'});
	var reader = new FileReader();
	reader.readAsDataURL(blob);  //FileReader将返回base64编码的data-uri对象
	reader.onload = function(){
        img.src = this.result;
    }
}).error(function() {
	        		
});
```

## 2.3 $http.get()使用缓存机制

```javascript
angular.module("demo", []).factory("demoCache", ['$cacheFactory', function($cacheFactory){  //生成$cacheFactory实例
	return $cacheFactory("demoCache", {capacity: 20});
}]).controller("demoCtrl" ,['$scope', '$http', 'demoCache', function($scope, $http, demoCache) {
	$http.get(url, {cache: demoCache}).success(function(response){  //使用$cacheFactory实例缓存数据，再次请求相同url时会读取缓存数据
				
	}).error(function(){
		
	});
}]);
```

## 2.4 $http()执行返回作为promise的用法

```javascript
var p1 = $http.get(url1);
var p2 = $http.get(url2);

var task = $q.all([p1, p2]);
task.then(function(res) {   //p1和p2都执行完后，才会执行task.then()里面的方法
	console.log(res[0]);
	console.log(res[1]);
});
```
