使用 angular 自定义指令封装实现一个分页组件

# 定义组件的外部行为
```html
<zpagination total-items="page.total" items-per-page="page.pageSize" max="page.max" 
  ng-model="page.currentPage" ng-change="getData()">
</zpagination>
```
 以上定义的组件的模型，其中：  
 total-items: 数据的总数条数  
 items-per-page: 每页的数据条数  
 max: 最大同时显示的页码的数量  
 
 现在预期的是 ng-model 绑定的当前页页码，当 ng-model 绑定的模型值变化时，也即页码变化时，触发 ng-change 事件，从而请求新一页的数据。
 
 # 定义组件的内部行为
  根据上面定义的组件外部行为，组件对象的内部状态是当前页的页码值，组件外部的输入触发组件内部状态的更新，当内部状态也即页码值更新时,组件通知 ng-model，引发 model 值的更新，最终触发 ng-change 事件。     
  组件对外暴露的操作可以有前页、后页、首页、尾页、任意页切换，因此组件内部要有对应的事件响应，能够处理这些外部操作，这些事件响应会更新组件内部的状态值(当前页码值)。
 
 # 具体实现代码
 ```javascript
 angular.module('utils', [])
.directive('zpagination', function(){
	    return {
        require: 'ngModel',
        restrict: 'AE',
        scope:{
            totalItems:'=totalItems',//数据总条数
            itemsPerPage:'=itemsPerPage', //每页数据条数
            max: '=max' //同时显示的页码数量上限，超过范围的页码通过前页、后页按钮切换显示
        },
        template: `<ul class="pagination">
                      <li ng-click="firstPage()"><a>首页</a></li>
                     <li ng-click="prevPage()"><a>前页</a></li>
                     <li ng-repeat="page in pageList | filter:{show: true}" ng-click="selectPage(page)" ng-class="{true:'active'}[page.isActive]"><a>{{page.index}}</a></li>
                     <li ng-click="nextPage()"><a>后页</a></li>
                     <li ng-click="lastPage()"><a>末页</a></li>
                   </ul>`,
        link: function(scope, ele, attrs, ctrl) {
            scope.pageList = [];
            scope.range = [1, scope.max || 5];
            createPages(scope.totalItems, scope.itemsPerPage, scope.max);
            scope.$watch('totalItems', function() {
                createPages(scope.totalItems, scope.itemsPerPage, scope.max);
            })

            scope.prevPage = prevPage;
            scope.nextPage = nextPage;
            scope.selectPage = selectPage;
            scope.firstPage = function() {
                scope.pageList[0] && selectPage(scope.pageList[0]);
            }
            scope.lastPage = function() {                    
                scope.pageList.reverse()[0] && selectPage(scope.pageList.reverse()[0]);
            }
            

            ctrl.$render = function() {  // ngModel 绑定的值为当前页，当前页的 model 值改变时，更新分页视图状态
                var currentPage = parseInt(ctrl.$modelValue);
                scope.pageList.map(function(page) {
                    page.isActive = currentPage == page.index;
                });
                updatePageView(currentPage, scope.range);
            };

            function createPages(totalItems, itemsPerPage, max) { // 生成分页页码
                totalItems = totalItems || 0;
                itemsPerPage = itemsPerPage || 8;
                max = max || 5;
                var totalPage = Math.ceil(totalItems / itemsPerPage);
                scope.pageList = [];
                for (var i = 0; i < totalPage; i++) {
                     scope.pageList.push({
                        index: i + 1,
                        isActive: false,
                        show: (i + 1 <= max)
                    });
                }
            }

            function nextPage() { // 前一页
                for (var i = 0, len = scope.pageList.length; i++; i < len) {
                    if (scope.pageList[i].isActive && scope.pageList[i + 1]) {
                        ctrl.$setViewValue(scope.pageList[i + 1].index);  // 更新页码的视图值，触发 model 值的自动更新，model 值的更新进一步触发ng-change事件
                        scope.pageList[i + 1].isActive = true;
                        updatePageView(scope.pageList[i + 1].index, scope.range);
                        break;
                    }
                }
            }

            function prevPage() {// 下一页
                for (var len = scope.pageList.length, i = len - 1; i--; i >= 0) {
                    if (scope.pageList[i].isActive && scope.pageList[i - 1]) {
                        ctrl.$setViewValue(scope.pageList[i - 1].index);
                        scope.pageList[i - 1].isActive = true;
                        updatePageView(scope.pageList[i - 1].index, scope.range);
                        break;
                    }
                }
            }

            function selectPage(page) { //切换页面
                scope.pageList.map(function(p) {
                    p.isActive = p.index == page.index;
                });
                ctrl.$setViewValue(page.index);
                updatePageView(page.index, scope.range);
            }

            function updatePageView(currentPage, range) { //计算分页显示的页码范围
                range[0] = currentPage < range[0] ? currentPage : range[0];
                range[1] = currentPage > range[1] ? currentPage : range[1];
                scope.pageList.map(function(page) {
                    if (page.index <= range[1] && page.index >= range[0]) {
                        page.show = true;
                    } else {
                        page.show = false;
                    }
                });
            }
        }

    }
});
 ```
