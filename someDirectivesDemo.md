若干增强页面易用性的 angular 指令

## 1、表单导航
填写表单时，为了减少用户鼠标点击的次数，有更好的输入体验，编写了一个指令：enter 键焦点自动跳到下一个输入框，↑ 键焦点移动到上一个输入框， ↓ 键焦点移动到下一个输入框。
```javascript
module.directive("autoFocus", [function() { 
    return {
        restrict: 'A',
        controller: ['$scope', function($scope) {
        	$scope.$inputs = [];
        }],
        link: function(scope, ele, attrs) {
        	if (!ele.context) {
        		ele = $(ele);
        	}
           scope.$inputs = ele.find("input[type='text']:enabled,input[type='number']:enabled,textarea:enabled");
           focus(scope.$inputs);
           scope.$inputs[0] && scope.$inputs[0].focus();    
                
           scope.$on('autoFocusInputs', function() {
        	   focus(scope.$inputs);
               scope.$inputs[0] && scope.$inputs[0].focus();    
           });
           
           function focus(inputs) {
          	 inputs.each(function(index, input) {
          		function autoFocus(e) {
	         		   e = window.event || e;
	                    var k = e.keyCode || e.which;
	                    if (k == 13 || k == 40) {
	                 	   inputs[index+1] && inputs[index+1].focus &&  inputs[index+1].focus();
	                    }
	                    if (k == 38) {
	                 	   inputs[index-1] && inputs[index-1].focus &&  inputs[index-1].focus();
	                    }
	         	   }
          		 $(input).unbind('keyup', autoFocus);
          		 $(input).keyup(autoFocus);
        	   })
          }
          
        }
    
    }    
}]);
```
示例：
```html
<form auto-focus>
	<input type="text">
	<input type="text">
	<textarea></textarea>
</form>
```

## 2、搜索栏查询条件改变时自动触发查询动作
该指令实现的功能是：input 输入完毕后按 enter 键触发查询动作，select 下拉狂更改选项自动触发查询动作，避免用户手动点击查询按钮的动作。
```javascript
module.directive("autoSearch", [function() {
	return {
		restrict: 'A',
		controller: ['$scope', '$attrs', '$parse', function($scope, $attrs, $parse) {
        	$scope.$inputs = [];
        	$scope.$selects = [];
        	$scope.$search = function() {
        		$parse($attrs.search)($scope.$parent); //search是被触发的查询函数
        	}
        }],
		link: function(scope, ele, attr) {
			scope.$inputs = ele.find("input:enabled");
			scope.$selects = ele.find("select");
			
			auto(scope.$inputs, scope.$selects);
		    scope.$on('autoSearchInputs', function() {
		    	auto(scope.$inputs, scope.$selects);
            });
			
			function auto(inputs, selects) {
				selects.each(function(index, select) {
					function enterSearch(e) {
						scope.$apply(scope.$search);                 
	         	    }					
					$(select).unbind('change', enterSearch);
		      		$(select).change(enterSearch);
				});
				
				inputs.each(function(index, input) {
		      		function autoSearch(e) {
	         		    e = window.event || e;
	                    var k = e.keyCode || e.which;
	                    if (k == 13) {
	                    	scope.$apply(scope.$search);
	                    	input.blur();
	                    }                    
	         	   	}
	      		   	$(input).unbind('keypress', autoSearch);
	      		   	$(input).keypress(autoSearch);
		    	})
			}			
		}
	}
}]);
```
示例：
```html
<div class="search-bar" auto-search search="search()">
	<input type="text">
	<select></select>
	<button ng-click="search()">查询</button>
</div>
```

## 3、让元素可拖动
对于 angular 的组件库 ui-bootstrap 中的 $uibModal 模态框，是不可移动的，为了让之可移动，编写下面的指令。
```javascript
module.directive("enablemove", [function() { 
    return {
        restrict: 'A',
        link: function(scope, ele, attrs) {
            var move = {
                oldX: "",
                oldY: "",
                left: "",
                top: "",
                enablemove: false,
                ele: ele
            };

            function mousemoveHandler(evt) {
                evt.preventDefault(); 
                var dx = parseInt(evt.clientX - move.oldX);
                var dy = parseInt(evt.clientY - move.oldY);
                that.style.left = (move.left + dx) + 'px';
                that.style.top = (move.top + dy) + 'px';
            };

            function mouseupHandler(evt) {
                document.body.removeEventListener('mousemove', mousemoveHandler);
                document.body.removeEventListener('mouseup', mouseupHandler);
            };
            var that = move.ele;
            that.onmousedown = function(evt) {
                move.oldX = evt.clientX;
                move.oldY = evt.clientY;
                move.enablemove = true;
                if (that.currentStyle) {
                    move.left = angular.element(that).position().left;
                    move.top = angular.element(that).position().top;
                } else {
                    var divStyle = document.defaultView.getComputedStyle(that, null);
                    move.left = parseInt(divStyle.left);
                    move.top = parseInt(divStyle.top);
                }  
                var target = evt.srcElement || evt.target; //兼容火狐浏览器和其他浏览器
                if (target.dataset.move) {  //在指定区域才可触发元素拖动的动作
                    document.body.addEventListener('mousemove', mousemoveHandler);
                    document.body.addEventListener('mouseup', mouseupHandler);
                }

            };
        }
    }
}]);
```
示例：
```html
<div enablemove>
	<div class="title" data-move="true"></div>
	<div class="content"></div>
</div>
```

## 4、滚动到特定元素位置触发事件
```javascript
module.directive('scrollTrigger', function() {
    return {
        restrict: "A",
        link: function ($scope, $element, $attrs) {
          	function debounce(fn, delay) {
	            var timer = null;
	            return function () {//用于包装函数，避免频繁触发
		            var context = this;
		            var args = arguments;
		            clearTimeout(timer);
		            timer = setTimeout(function () {
		                fn.apply(context, args);
		            }, delay);
	            }
          	}
          	var scrollEle = $('#' + $attrs.scrollTrigger);
          	var preScrollTop = 0;
          	var  trigger = debounce(function() {
	            if ($element.is(":hidden")) {
	              	return;
	            }

	            if (($(window).scrollTop() + $(window).height()) > ($element.offset().top + $element.height() + 5)) {
	            	if (preScrollTop > scrollEle.scrollTop()) {//上滚
	            		$scope.$eval($attrs.zScrollTriggerUp); //向上滚动到该元素触发事件
	            	} else if (preScrollTop < scrollEle.scrollTop()) {//下滚
	            		$scope.$eval($attrs.zScrollTriggerDown);//向下滚动到该元素触发事件
	            	}
	            	$scope.$eval($attrs.zScrollTrigger);//向上或向下滚动到该元素触发事件
	            }
	            preScrollTop = scrollEle.scrollTop();
          	}, 100);
          	scrollEle.bind('scroll', trigger);
          	$scope.$on("destroy", function() {
        	  scrollEle.unbind("scroll", trigger);
          })
        }
      }
    });
```
示例：
```html
<div id="ele-scroll"><!-- 滚动的元素 -->
	<div></div>
	<div scroll-trigger="ele-scroll" z-scroll-trigger="scroll()" z-scroll-trigger-up="scrollUp()" z-scroll-trigger-down="scrollDown()"></div>
	<div></div>
</div>
```
```javascript
$scope.scroll = function() { console.log('滚动到当前元素'); };
$scope.scrollUp = function() { console.log('向上滚动到当前元素'); };
$scope.scrollDown = function() { console.log('向下滚动到当前元素'); };
```
