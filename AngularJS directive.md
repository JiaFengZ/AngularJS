angular 自定义指令 directive 的配置

一个典型的 directive 定义如下：
```javascript
angula.module('app.directive', []).directive('directiveName', function() {
	var config = {
		// 配置项
		//restrict: ...,
		//scope: ...,
		//...
	};
	return config;
});
```

## restrict
指定指令的的声明类型，可选值如下：  
  1、'E': 元素  <directiveName></directiveName>  
  2、'A'：属性  <div directiveName="expression"></div>  
  3、'C': class 类  <div class="directiveName"></div>  
  4、'M': 注释  <!--directive:directiveName expression--> 

# scope
 1、false: 默认值，继承父级作用域，不创建自己的作用域  
 2、true: 继承父作用域，并且创建自己的子作用域   
 3、{}: 创建一个独立隔离的子作用域，可使用三种方式引用外部作用域的属性：    
     '@attr' 单向绑定引用父作用域的属性值，属性值的改变可以改变隔离作用域绑定的值  
     '=attr' 双向绑定引用父作用域的属性值，属性值的改变可以改变隔离作用域绑定的值，同时隔离作用域的值也可以改变父作用域值    
     '&attr' 引用执行父作用域的函数

```html
<div ng-controller="demoCtrl">
  <span>{{name}}</span>
  <div demo-directive></div>
  <div demo-directive1></div>
  <div demo-directive2></div>
</div>
```
```javascript
angular.module('demo', []).controller('demoCtrl', ['$scope', function($scope) {
  $scope.name = 'A';
}]).directive('demoDirective', [function() {
  return {
    restrict: 'EA',
    scope: false,
    link: function(scope, ele, attr) {
      console.log(scope.name); // name继承自父scope，因此值为'A'
      scope.name = 'B'; // 更改name属性值，由于指令没有创建子作用域，因此更改的是父scope，即 demoCtrl 中的name值
    }
  }
}]).directive('demoDirective1', [function() {
  return {
    restrict: 'EA',
    scope: true,
    link: function(scope, ele, attr) {
      console.log(scope.name); //  name继承自父scope，因此值为'A'
      scope.name = 'B'; // 由于指令创建了子作用域，因此会在指令子作用域scope中创建一个name属性，值为'B'
    }
  }
}]).directive('demoDirective2', [function() {
  return {
    restrict: 'EA',
    scope: {
      name: 'C'
    },
    link: function(scope, ele, attr) {
      console.log(scope.name); //  指令创建隔离的作用域，值为'C'
    }
  }
}]);
```

## priority
指令的优先级，值越大，优先级越高，当某个元素拥有多个指令时，会根据这些指令的 pirority 优先级从高到底执行编译，例如 ng-repeat 的优先级为 1000，是一个优先级较高的指令，一般而言，需要创造渲染 DOM 元素的指令优先级会比较高。

# terminal
这是一个跟 priority 息息相关的选项，terminal 为 true 时，优先级低于此指令的其他指令不会执行，意为终结。

# template
模版，可以是一段 html 字符，也可是一个返回一段模版字符串的函数：  
```html
<div ng-controller="demoCtrl">
  <div demo-directive name="{{name}}"></div>
</div>
```
```javascript
angular.module('demo', []).controller('demoCtrl', ['$scope', function($scope) {
  $scope.name = 'A';
}]).directive('demoDirective', [function() {
  return {
    restrict: 'EA',
    template: '<span>{{name}}</span>',
    scope: {
      name: '@'
    },
    link: function(scope, ele, attr) {
      
    }
  }
}]);
```

上面指令编译后的html如下所示：
```html
<div ng-controller="demoCtrl">
  <div demo-directive name="A"><span>A</span></div>
</div>
```

## templateUrl
指令嵌入模版的url地址，异步进行加载
```html
<!-- directive.html -->
<span>{{name}}</span>
```
```javascript
angular.module('demo', []).controller('demoCtrl', ['$scope', function($scope) {
  $scope.name = 'A';
}]).directive('demoDirective', [function() {
  return {
    restrict: 'EA',
    templateUrl: 'directive.html',
    scope: {
      name: '@'
    },
    link: function(scope, ele, attr) {
      
    }
  }
}]);
```
在以下html中调用指令：
```html
<div ng-controller="demoCtrl">
  <div demo-directive name="{{name}}"></div>
</div>
```
编译后的html如下，和使用template效果一样
```html
<div ng-controller="demoCtrl">
  <div demo-directive name="A"><span>A</span></div>
</div>
```

## replace
  true: 模版内容替换该自定义指令的节点标签  
  false: 不替换，保留声明自定义指令的节点标签
以上例说明，如果声明指令时如下：
```javascript
module.directive('demoDirective', [function() {
  return {
    restrict: 'EA',
    templateUrl: 'directive.html',
    replace: true,
    scope: {
      name: '@'
    },
    link: function(scope, ele, attr) {
      
    }
  }
}]);
```
则编译后html如下，可见声明指令的 div 被指令模版元素替换了：
```html
<div ng-controller="demoCtrl">
  <span>A</span>
</div>
```

## transclude
默认值是 false，可选值为 true 或者 'element'， true 则自定义指令元素的内部元素被嵌入指令模版 ng-transclude 声明的位置，'element'则整个自定义元素嵌入到指令模版 ng-transclude 声明的位置。  
首先看看 transclude 为 true 时：
```javascript
angular.module('demo', []).controller('demoCtrl', ['$scope', function($scope) {
  $scope.name = 'A';
}]).directive('demoDirective', [function() {
  return {
    restrict: 'EA',
    template: `<div>
                <div ng-transclude></div>
                <span>{{name}}</span>
              </div>`,
    transclude: true,
    scope: {
      name: '@'
    },
    link: function(scope, ele, attr) {
      
    }
  }
}]);
```
以下使用指令的html
```html
<div ng-controller="demoCtrl">
  <div demo-directive name="{{name}}"><span>element to be transcluded</span></div>
</div>
```
经过编译后：
```html
<div ng-controller="demoCtrl">
  <div demo-directive name="A">
    <div>
      <div ng-transclude><span>element to be transcluded</span></div><!-- 指令的子元素被嵌入到指令模版的这个位置 -->
      <span>A</span>
    </div>
  </div>
</div>
```

如果需要更加灵活的控制元素嵌入到指定位置，可以使用指令 complie 选项中的 transclude 参数,transclude 的值，就是 directive 所在的原始节点编译之后得到的 link 函数,装载入 scope 执行它就得到了一个节点
```html
<div demo-directive>
  <div id="transcluded1"><span>element1 to be transcluded</span></div>
  <div id="transcluded2"><span>element1 to be transcluded</span></div>
</div>
```
```javascript
angular.module('demo', []).controller('demoCtrl', ['$scope', function($scope) {
  $scope.name = 'A';
}]).directive('demoDirective', [function() {
  return {
    restrict: 'EA',
    template: `<div>
                <div id="transclude1"></div>
                <span>{{name}}</span>
                <div id="transclude2"></div>
              </div>`,
    transclude: true,
    scope: {
      name: '@'
    },
    compile: function(ele, attr, transclude) {
      return function(scope, element, attr) {
        transclude(scope, function(clones) { // link 函数
          angular.forEach(clones, function(clone) {
            if (clone.id == 'transclude1') {
              ele.find('#transclude1').append(clone);
            } else if (clone.id == 'transcluded2') {
              ele.find('#transclude2').append(clone);
            }
          })
          
        });
      };
    }
  }
}]);
```
上述指令编译后的html如下：
```html
<div demo-directive>
  <div>
    <div id="transclude1"><div id="transcluded1"><span>element1 to be transcluded</span></div></div>
    <span>A</span>
    <div id="transclude2"><div id="transcluded2"><span>element1 to be transcluded</span></div></div>
  </div>
</div>
```

需要注意的是，上面的指令是 ng 自动初始化编译执行的，如果是手动调用 $compile 方法编译，那么传入指令 compile 的 transclude 参数是手动调用$compile传入的函数(transclude)：
```javascript
angular.module('demo', []).controller('demoCtrl', ['$scope', '$compile', '$element', function($scope, $compile, $element) {
  $scope.name = 'A';
  var htm = `<div demo-directive>
              <div id="transcluded1"><span>element1 to be transcluded</span></div>
              <div id="transcluded2"><span>element1 to be transcluded</span></div>
            </div>`;
  var link = $compile(htm, function(scope, cloneAttachFn) {  //这个函数就是传入指令 compile 选项的 tranclude 参数(不建议传入这个函数)
    
  });
  var node = link($scope); //返回link执行后的节点元素
  $element.append(node);
}])
```
如果给link函数传入第二个参数 cloneFn,则可获得编译链接后的节点元素的拷贝：
```javascript
  var link = $compile(angular.element(htm));
  var clonedNode = link($scope, function(clonedNode, scope) { //link执行后返回克隆的节点元素
      $element.append(clonedElement); //添加到DOM中
  });
```

也可以在controller中注入 $transclude 服务来添加元素到 DOM 中：
```javascript
angular.module('demo', []).controller('demoCtrl', ['$scope', function($scope) {
  $scope.name = 'A';
}]).directive('demoDirective', ['$element', '$transclude', function($element, $transclude) {
  return {
    restrict: 'EA',
    template: `<div>
                <div id="transclude1"></div>
                <span>{{name}}</span>
                <div id="transclude2"></div>
              </div>`,
    transclude: true,
    scope: {
      name: '@'
    },
    controller: ['$scope', '$element', '$transclude', function ($scope, $element, $transclude) {
      var transcludedContent, transclusionScope;
      $transclude(function(clones, scope) {
        angular.forEach(clones, function(clone) {
          if (clone.id == 'transcluded1') {
            $element.find('#transclude1').append(clone);
          } else if (clone.id == 'transcluded2') {
            $element.find('#transclude2').append(clone);
          }
        })
        transcludedContent = clone;
        transclusionScope = scope;
      });

      $scope.$on('$deatroy', function() { //手动调用 $transclude 向 DOM 添加元素,当scope销毁时需要手动消除元素拷贝和销毁transclude产生的作用域
        transcludedContent.remove();
        transclusionScope.$destroy();
      })
    }]
  }
}]);
```


## controller
指令控制器，可以定义指令内部的行为，也可以定义指令之间复用的行为，提供指令间的交互。

## controllerAs
通过一个别名生成当前指令的 controller 实例，属性和方法挂载在 controller 构造的this对象上，this对象是 scope 的一个属性，属性和方法并不直接定义在scope对象上。

## require
依赖第三方指令controller，可以引用第三方指令 controller 中定义的属性和方法，实现指令间的复用。


[参考资料-AngularJS 文档](https://docs.segmentfault.com/angularjs~1.5/api/ng/service/$compile)
