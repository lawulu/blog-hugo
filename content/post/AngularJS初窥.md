+++
title = "AngularJS初窥"
draft = false
date = "2016-11-20"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["js","angular"] 
toc = true

+++

无数人吐槽过前端框架的层出不穷,眼看前端开发要脱离浏览器兼容性的苦海,又要落入框架碎片化,从语言本身到编译到打包到开发框架,是终端用户需求太多还是前端大牛们野心太大? 不管怎样,报G家大腿选择Angular似乎没什么问题,另一个大腿的Reactive本身不是一个完备的框架.

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
- http是异步,要在.function中处理,或者统一在配置里面设置`async : true`
- 怎么Lazy初始化？没有找到办法

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
- 有时候Jquery绑定Function时候,Dom并没有生成,所以需要替换成Angular的`ng-click`事件.
- JS中可以通过元素来找的其绑定的scope
    ```
    window.angular.element("#divId").scope().doSth()
    ```

### Angular的分支循环
Angular提供了Ng-show等逻辑判断,来控制页面的变化.
提供了Switch和For来做分支和循环.
简单来说,基本可以对Html的显示进行编程.