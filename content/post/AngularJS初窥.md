+++
title = "AngularJS初窥"
draft = false
date = "2016-11-20"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["js","angular"] 
toc = true

+++

无数人吐槽过前端框架的层出不穷,眼看前端开发要脱离浏览器兼容性的苦海,又要落入框架碎片的深坑.从语言本身到编译到打包到开发框架各种框架层出不穷...是终端用户需求太多还是前端大牛们野心太大? 不管怎样,报G家大腿选择Angular似乎没什么问题,另一个大腿的Reactive本身不是一个完备的框架.

## Angular简介
- 在Web开发中需要JS做什么？
    - 与服务器交互
    - 操作DOM
    - 简单的逻辑控制
    - 输入验证
    - 路由

- Angular做了什么改变？
    - 引入了指令,ng开头,来扩展HTML
    - 数据绑定,通过对HTML元素增加指令ng-model
    - 使用directive来自定义指令
    - Scope,提供了HTML视图和JS控制器之间的纽带
    - Controller,可以视为一个$Scope的构造函数
- 数据绑定 即将Dom的变化绑定到数据的变化上面去,因为用的1,所以这个绑定是双向的,有时候想只改变页面,会影响到数据

## 使用一周的感受

### 一些坑:
- 架构很恢弘，学习曲线很陡峭
- http是异步,要在.function中处理,为避免因为异步未完成引起的页面问题，课可以统一在配置里面设置`async : true`
- 怎么Lazy初始化？没有找到办法，可能需要自定义Directive
- 一些自定义Directive封装的不彻底，各种问题。

### Controller之间的交互
两种办法

- http://stackoverflow.com/questions/29467339/how-to-call-a-function-from-another-controller-in-angularjs
本质是绑定到RootScope里面,通过事件来响应
- http://blog.csdn.net/findsafety/article/details/50403242
原生dom与js的处理办法,可能会版本之间不兼容,本质是通过dom找到Ng-controller的方法.

```
    var appElement = document.querySelector('[ng-controller="EndCardEditCtrl as ctrl"]');
    var editCtrl = angular.element(appElement);
    
```

### 与Jquery/JS之间的交互
- 有时候Jquery绑定Function时候,Dom并没有生成,所以需要替换成Angular的`ng-click`事件,或者需要绑定到顶级上面,冒泡绑定.
- JS中可以通过元素来找的其绑定的scope
    ```
    window.angular.element("#divId").scope().doSth()
    ```

### Angular的分支循环
Angular提供了Ng-show等逻辑判断,来控制页面的变化.
提供了Switch和For来做分支和循环.
简单来说,基本可以对Html的显示进行编程.

### $scope.$apply()
点击按钮之后,为什么会绑定成功?奥秘就在于Angular把
```
Events => ng-click
Timeouts => $timeout
jQuery.ajax() => $http
```
都Wrapper了,只要这些事件被触发,自动调用`$scope.$apply()`

问题是很多时候(在$scope之外的操作)需要自己手动调用, 但是Angular又保证此方法不能调用两次.
所以官方推荐使用Angular自己那套($Resource,$Route,service之类...).恩,有种上了贼船的感觉.

### $Resource
一个对HttpRest的封装,简单的CRUD确实很方便,但是遇到复杂的需求就要抓瞎,很多时候需要调整后端..所以有很多人说,Angular适合做简单的CRUD应用.

### Ng-change
ng-change requires ng-model, without it will not work

```
Error: [$compile:ctreq] http://errors.angularjs.org/1.4.5/$compile/ctreq?p0=ngModel&p1=ngChange

```
另外 ng-change对File不生效

https://github.com/angular/angular.js/issues/1375

### 单元测试
Angular轮了一套Service,Factory之流,自然也少不了单元测试..不知道有多少人会为页面写这玩意.

## Angular自定义Directive和Ng-repeat的问题
### 现象
同事自定义了一个`directive`，模仿[这里](http://www.cnblogs.com/e50000/p/5663806.html)做的一个日期控件。这个`directive`需要input域里面含有一个id。另外一个同事想在 `np-repeat`里面使用这个控件，为了保证id唯一性，写成了这样，
```
 <input type="text" ng-model="statDatas[$index].endDate" class="overviews_date" placeholder="结束时间" "id="{{'startDateInput'+$index}}"  ng-change="statDateChanged($index)" readonly  def-laydate>
```
结果一直报错

```
TypeError: Cannot read property 'tagName' of undefined
    at Object.Dates.run (http://localhost:8081/resources/js/laydate.js:171:42)
    at win.laydate (http://localhost:8081/resources/js/laydate.js:34:11)
    at link (http://localhost:8081/resources/platformJs/ad_angular.js:592:20)
    at http://localhost:8081/resources/angular/angular.min.js:72:493
    at $ (http://localhost:8081/resources/angular/angular.min.js:73:46)
    at L (http://localhost:8081/resources/angular/angular.min.js:61:495)
    at g (http://localhost:8081/resources/angular/angular.min.js:54:326)
    at g (http://localhost:8081/resources/angular/angular.min.js:54:349)
    at g (http://localhost:8081/resources/angular/angular.min.js:54:349)
    at g (http://localhost:8081/resources/angular/angular.min.js:54:349) <input type="text" ng-model="statDatas[$index].startDate" class="overviews_date ng-pristine ng-untouched ng-valid ng-isolate-scope" placeholder="开始时间" id="{{'startDateInput'+$index}}" ng-change="statDateChanged($index,ad.id)" readonly="" def-laydate="">
```

### 分析
一开始就怀疑，是因为执行这个`directive`时候`np-repeat`还没有执行完，即这个dom还没有渲染完毕。那怎么解决呢？
1. 监听`np-repeat`,等结束之后，再去动态的执行自定义`directive`。例如这篇文章里面：[利用angular指令监听ng-repeat渲染完成后执行脚本](http://www.tuicool.com/articles/Fb2um2e)
2. 貌似有个Priority的属性，然而，搜了一下`np-repeat`的Priority非常大，为1000，而自定义的priority为0.
3. 仔细看报错日志，感觉像是laydate里面拿到的document是angular渲染之前的，所以找不到对应的Input域。这时候搜索已经没有意义了，应该仔细弄清楚`directive`各行代码的意义。
4. scope的问题？
### 解决办法
用原始的值传递和`$scope.$apply`来交互数据，弃掉自定义控件