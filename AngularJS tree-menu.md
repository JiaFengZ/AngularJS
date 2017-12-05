使用 AngularJS 自定义指令实现树形菜单组件

在项目开发中经常要实现一个侧边栏树状结构的菜单，往往要先根据设计的菜单在 controller 中预先定义其数据结构 model，然后在模版中根据 model 配合 ng-repeat 指令嵌套渲染生成菜单的 DOM 结构，对这个具有通用行为和结构的菜单组件，尝试封装成通用组件时，主要问题是菜单的层级也就是的树的深度是不确定的，受到[Angular实现递归指令 - Tree View-破狼](http://www.cnblogs.com/whitewolf/p/Angular-tree-view.html)的启发，可用递归的方法实现该组件。

## 定义菜单数据结构
```javascript
$scope.menus = [{
	name: '父菜单1',
	expend: false,  // 菜单是否展开
	clickEvent: function() {},  //点击菜单回调函数
	sub: [{
		name: '子菜单',
		expend: false,
		clickEvent: function() {},
		sub: [{
			name: '子子菜单',
			clickEvent: function() {}
		}]
	}]
},{
	name: '父菜单2',
	expend: false,
	clickEvent: function() {}
}]
```

## 定义指令

```javascript
angular.module('app')
.directive('treeMenu',[function(){
 
     return {
          restrict: 'E',
          template: `<ul class="tree-menu">
                      <li ng-repeat="item in treeData" ng-include="'/treeItem.html'" ></li>
                    </ul>`,
          scope: {
              treeData: '='
          },
         controller:['$scope', function($scope){
 
             $scope.isLeaf = function(item){
                return !item.children || !item.children.length;
             };
 
         }]
     };
 }]);
```

 ```html
 <!-- treeItem.html -->
<span ng-click="item.clickEvent()"></span>
<ul ng-if="!isLeaf(item)" ng-show="item.expend">
   <li ng-repeat="item in item.sub" ng-include="'/treeItem.html'">
   </li>
</ul> 
```

使用：
```html
<tree-menu tree-data="menus"></tree-menu>
```

实现树形层级嵌套的关键是利用 ng-include 嵌套加载子节点菜单。
像这种具有树状结构的 UI 组件是常见的，比如目录分类、部门分类等等，对上文的代码稍作改动扩展就能满足需求。

## 其他
有时候需要前端把具有父子关系的数据条目变成树状结构，这里记录一个方法：
```javascript
/*
*@param {array} data 原始数数组
*@param {object} keys 父子数据条目id标识键名
*@param {string} keys.id 数据id标识的键名
*@param {string} keys.pid 数据父条目id标识的键名
*@param {string} topId 根节点的父id标识
*@return {array} 返回的树形结构数据
*/
function generateTreeData(data, keys, topId) {
			    	
	var treeData = {$id: topId, sub: []};			    	
	var getSub = function (parent, data) {
		for(var len = data.length, i = len-1; i >= 0; i--){
			if(parent.$id == data[i][keys['pid']]){		    				
				var item = {};			    				
				for(var j in data[i]){
					item[j] = data[i][j];
				}
				item.$id = data[i][keys['id']];
				item.sub = [];
				parent.sub.unshift(item);
				data.splice(i, 1);
			}			    			
		}
		if(data.length == 0){
			return;
		}
		for(var k = 0, len1 = parent.sub.length; k < len1; k++){
			getSub(parent.sub[k], data);
		}		    		
	};
	getSub(treeData, data);
	return treeData;
}
```
