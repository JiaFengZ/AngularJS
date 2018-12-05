# 利用  `component` 进行组件化编程改造
在 `component` 特性出现之前，angularjs 应用呈现的是 `scope` 元素作用域树的结构，DOM 树和 scope 树是一体化的。
scope 主要有以下几个特点：
* 具有父子继承的关系，子 scope 可以访问父 scope 中的数据和方法
* 充当数据中心和事件中心的角色，连接 model 和 view 视图层，数据直接双向绑定在 scope 中，页面事件也直接绑定在 scope 中
存在以下问题：
* 可见 scope 职责过重，虽然渲染数据和事件交互很方便，但是随着应用的扩展，scope 难免变得越来越臃肿，而且事件和数据完全耦合在一起，
也不便于维护。
* scope 默认的作用域继承，导致应用过于依赖继承树，局部界限不够明确，数据和事件不够独立，模块性和移植性容易被破坏

当然如果应用规划得当，这些问题也是可以避免的，而且 angularjs 也提供了一些额外的手段：
* 自定义指令，实际上就是修饰型指令和元素结构型指令，修饰型指令主要用于扩展元素功能，例如 `enableMove` 使元素可拖动 `autoSearch` 查找子元素的输去变化自动触发父元素的搜索方法
* 重点是结构型指令，能够渲染生成元素，编译生成一段html模版，同样具有事件和数据，这些事件和数据存在于指令创建的 scope 中。
指令的 scope 可以是隔离的，也就是不继承外部父元素的 scope 数据和方法，同时父元素通过 attributes  给指令元素模版传递属性数据和事件。

起始结构型指令已经可见 `component` 组件化的雏形，通过一定手段强制应用的各个部分相对独立隔离，明确界限，不同元素界限内数据和方法相对
独立，使用传递属性进行父子元素界限的数据通信。因此在 angularjs 的后续版本中出现了 `component` 这个概念，其实就是结构型指令转变而来的组件
化概念，`module.component()` 便可在 angular 模块上注册组件，`component` 声明定义 `template` 模版、`controller` 控制器（双向绑定的数据，事件方法）、`bindings` 接收父组件的数据和方法
```javascript
myMod.component('myComp', {
  templateUrl: 'views/my-comp.html',
  controller: function($http) {
      var self = this;
      self.age = '12';
      self.commit = function() {

      };
  },
  bindings: {
      name: '@', //绑定不变的string
      hero: '<', //单向绑定
      sex: '=', //双向绑定
      commitCallback: '&'
    }
});
```

使用 `component` 来架构应用之后，对整个项目的代码编写和组织结构都有了很大的改变，从以 controller 控制为中心的结构变成了以 component 为中心的结构，
一定程度上实现了组件化。整个angular应用变成了以 component 组合而成的组件树。
```
-- components
    -- moduleA
        -- moduleA.component.js
        -- moduleA.html
        -- moduleA.css
    -- moduelB
        -- moduleB.component.js
        -- moduleB.html
        -- moduleB.css
```
component具有以下特点：
* 每个 component 之间是相对独立的，职责和界限比较明确，而且数据和事件不再是直接定义在 socpe 上，而是定义在 scope 上的一个 ctrl 实例对象上，可以实现组件的复用。
* 暴露了生命周期事件，甚至可以实现面向组件生命周期编程：
     `$onInit` （组件创建初始化，可获取绑定输入的属性数据） 
     `$onChanges` （绑定的输入值变化后，执行回调）
     `$doCheck` （组件内部事件执行完毕触发脏值检测循环后执行的回调）
     `$onDestroy` （组件销毁时执行的回调，执行一些清理工作，比如清除定时器）
     `$postLink` （模版元素链接到dom文档后执行的回调，可以设置DOM事件，操作DOM元素）

# angularjs 的模块化系统
angularjs 使用 `angular.module(xxxx, [])` 定义一个模块，在模块上可注册 服务（factory-工厂函数，service-构造函数）、控制器（controller）、过滤器（filter）、组件（component）、指令（directive）
angularjs 有一个很重要的概念就是依赖注入，在模块按需注入其他依赖模块，在控制器中按需注入服务依赖。
