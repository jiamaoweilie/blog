# 使用AngularJS与api集成

如果我们有一个api提供一些json格式的数据，那么如何使用AngularJS来获取这些数据，又如何将处理过的的数据显示在一个具有responsive特性的页面上？本文将一一描述。

在本文的例子中api提供如下格式的json数组：

```json
[{"first_name": "Kelli", "last_name": "Guiney", "time": "01:00:58"},
{"first_name": "Brett", "last_name": "Cooper", "time": "01:05:35"},
{"first_name": "Felicity", "last_name": "White", "time": "01:07:15"},
{"first_name": "Monty", "last_name": "Hamilton", "time": "01:08:05"},
{"first_name": "Sarah", "last_name": "Battwraden", "time": "01:11:03"},
{"first_name": "Bob", "last_name": "Reynolds", "time": "01:11:37"},
{"first_name": "Mike", "last_name": "Baxter", "time": "01:14:14"},
{"first_name": "Terry", "last_name": "Gillespie", "time": "01:14:15"},
{"first_name": "Chris", "last_name": "Ford", "time": "01:22:27"},
{"first_name": "Sunil", "last_name": "Samarasinghe", "time": "01:24:24"},
{"first_name": "Dan", "last_name": "Martin", "time": "01:25:29"}]
```

最终要求是将前十个元素以下图的样式显示的页面上：

<img src="/images/img_for_angularjs_demo/showresult.png" alt="" width="70%">

并且这个页面具有[responsive](http://en.wikipedia.org/wiki/Responsive_web_design)特性，即当使用其他屏幕尺寸不同的设备时，其样式会有相应的变化。在手机上显示的结果如下图所示：

<img src="/images/img_for_angularjs_demo/phone.png" alt="" width="30%" />

## 设计一个responsive的表格

[Bootstrap](http://getbootstrap.com/)是Twitter推出的一个开源的用于前端开发的工具包。它是一个简洁、直观、强悍的前端开发框架，使用它可以很容易的设计出一个具有responsive特性的页面。在使用Bootstrap设计页面之前，需要在hmtl中引入它的`bootstrap.css`、`jquery.js`和`bootstrap.js`文件（Bootstrap是基于JQuery开发的，所以必须引入）：

``` html
<link href="bootstrap.min.css" rel="stylesheet">
<script src="jquery.min.js"></script>
<script src="bootstrap.min.js"></script>
```

我们的例子使用Bootstrap来设计一个表格，以展示后台提供的json数组：

``` html
<div class="table-responsive" ng-controller="LeaderBoardController">
    <table class="table table-box">
        <thead>
        <tr class="table-header ">
            <th width="50px" class="text-center">RANK</th>
            <th>NAME</th>
            <th width="120px" class="text-center text-nowrap">TIME<span style="color:#98ACE1">(HH:MM:SS)</span></th>
        </tr>
        </thead>
        <tbody id="items">
        <tr ng-repeat="item in items | limitTo:10 ">
            <td class="text-center">{ {$index + 1} }</td>
            <td class="text-left text-ellipsis">{ {item.first_name} } { {item.last_name} }</td>
            <td class="text-center text-nowrap" style="min-width:100px">{ {item.time} }</td>
        </tr>
        </tbody>
    </table>
</div>
```

从代码看来这个`table`似乎也没有什么神奇的地方，怎么就能自适应各种页面显示呢，答案其实就在`class="table-responsive"`。引入Bootstrap之后，可以通过使用它预设的一些class来实现我们想要的效果。比如本例中的`table-responsive`说明这个`div`里面的`table`具有responsive的特性。更多关于table的设置，请参见[官方文档](http://getbootstrap.com/css/#tables)。

## 显示api提供的数据

读上面一段代码时，我们会发现里面有一些类似于`ng-controller="LeaderBoardController"`，`{ {$index + 1} }`的奇怪东西。它们就是AngularJS中的一些标示符，用来帮助我们和页面交互。[AngularJS](https://angularjs.org/)诞生于Google,是一款优秀的前端JS框架，已经被用于Google的多款产品当中。它有着诸多特性，最为核心的是：MVC、模块化、自动化双向数据绑定、语义化标签、依赖注入，等等。使用AngularJS时必须首先引入`angular.js`:

```html
<script src="angular.min.js"></script>
```
我们的例子中使用AngularJS作为前端框架，来和后端的api交互。上面`html`代码中的`ng-controller="LeaderBoardController"`定义了一个Controller，用以响应与api交互的事件。`ng-repeat="item in items | limitTo:10 ">`表示repeat它所在的DOM并用元素`items`里面的值来填充DOM需要的值，并限制repeat次数为10。`{ {$index + 1} }`表示输出当前编号（`$index`是其索引值，从0开始），`{ {item.first_name} }`表示输出`item`的`fist_name`属性，其他与前面类似。

Controller的代码如下：

``` js
var app = angular.module("LeaderBoardApp", []);

app.controller('LeaderBoardController', ['$scope', 'LeaderBoardService'],
    function ($scope, LeaderBoardService) {
        LeaderBoardService.getLeaderBoardData().then(
            function (result) {
                $scope.items = result;
            });
    });
```

代码首先定义了一个AngularJS的模块，模块名称为`LeaderBoardApp`，然后定义了一个名为`LeaderBoardController`的Controller。很容易看出，这个Controller的作用是调用`LeaderBoardService`的`getLeaderBoardData()`函数，并将获取的数据传给`$scope.items`。此处的[`$scope`](https://docs.angularjs.org/guide/scope)是AngularJS的内置服务,当服务被找到并加载之后，执行匿名函数将其传入。比如此处，当AngularJS解析页面时，会动态的创建一个scope, 并将它的属性items设为`getLeaderBoardData()`函数返回的值，这样页面就可以获取到Controller得到的数据。

`LeaderBoardService`是AngularJS中的Service，用于与后台交互，它的定义与Controller相似：

``` js
var app = angular.module("LeaderBoardApp", []);

app.factory('LeaderBoardService', ['$http', '$q'],
    function ($http, $q) {
        return {
            getLeaderBoardData: function () {
                var deferred = $q.defer();

                $http.get("http://test/api").success(function (result) {
                    deferred.resolve(result.data);
                }).error(function (result) {
                    deferred.reject(result);
                });

                return deferred.promise;
            }
        };
    });
```

factory是AngularJS支持的Service定义的一种。从代码可以看出Service名字为`LeaderBoardService`，它依赖于[`$http`](https://docs.angularjs.org/api/ng/service/$http)服务和[`$q`](https://docs.angularjs.org/api/ng/service/$q)服务。`$http`服务用于向后台发送请求，并获取相应，例如例子中的代码向一个URL发送了一个get请求。$q可以用来创建以defer对象，defer有两个状态：resolve和reject。这里的$http不直接返回异步操作的结果，而是返回一个对defer对象的引用。这样就使用上例中`LeaderBoardController`里面的方法一样处理这个Service返回的结果了：

``` js
LeaderBoardService.getLeaderBoardData().then(
    function (result) {
        $scope.items = result;
    });
```

至此，我们已经可以从将后台api提供的数据显示在页面上了。如果此时我们有一个新的需求：因为后台的api提供的数据是变化的，所以我们需要每隔30秒刷新一次数据，即每隔30秒调用一次`LeaderBoardService.getLeaderBoardData()`。这时候就需要用到AngularJS的另一个服务[`$interval`](https://docs.angularjs.org/api/ng/service/$interval)。
代码如下：

``` js
var app = angular.module("LeaderBoardApp", []);
app.controller('LeaderBoardController', ['$scope', 'LeaderBoardService', '$interval'],
    function ($scope, LeaderBoardService, $interval) {
        refresh();
        $interval(function () {
            refresh();
        }, 30000);

        function refresh() {
            LeaderBoardService.getLeaderBoardData().then(
                function (result) {
                    $scope.items = result;
                });
        }
    });
```

首先需要像使用`$scope`那样，将`$interval`作为参数传给Controller，然后设置间隔时间为30000毫秒，无限重复，这样就会每隔30秒执行一遍refresh函数，刷新一遍页面。自此我们的需求也就初步实现。

更多关于AngularJS的用法，请参见[官方文档](https://docs.angularjs.org/api)。

